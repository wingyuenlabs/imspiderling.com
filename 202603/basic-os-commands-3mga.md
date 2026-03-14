---
Title: basic OS commands
Description: 
Author: methew smith 
Date: 2026-03-14T22:12:18.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Focus:</strong> Terminal-first, step-by-step command explanations for practical exam prep</p>
</blockquote>




<h2>
  
  
  Table of Contents
</h2>

<ol>
<li>Linux Basics &amp; File System</li>
<li>Linux Commands &amp; Navigation</li>
<li>File Permissions &amp; Ownership</li>
<li>Users &amp; Groups Management</li>
<li>SSH (Secure Shell)</li>
<li>Apache Web Server</li>
<li>Nginx Web Server</li>
<li>Port Debugging &amp; Troubleshooting</li>
<li>Bash Scripting</li>
<li>Docker Containers</li>
<li>Lab Exercises Summary</li>
<li>Quick Reference Cheat Sheet</li>
</ol>




<h2>
  
  
  1. Linux Basics &amp; File System
</h2>

<h3>
  
  
  What is Linux?
</h3>

<p>Linux is an open-source operating system kernel. A <strong>Linux Distribution</strong> (distro) is a collection of software built around the Linux kernel, all licensed under open-source or GPL. <strong>90% of all cloud infrastructure is powered by Linux</strong>, making it the foundation of DevOps.</p>

<h3>
  
  
  Linux Distributions
</h3>

<ul>
<li>
<strong>Debian-based</strong>: Ubuntu, Debian — use <code>.deb</code> packages and <code>apt</code> package manager</li>
<li>
<strong>Red Hat-based</strong>: CentOS, Fedora, RHEL — use <code>.rpm</code> packages and <code>yum/dnf</code> package manager</li>
<li>Technical differences are mainly about package management, software versions, and file locations</li>
<li>Once you grasp those differences, switching between distros becomes painless</li>
</ul>

<h3>
  
  
  Choosing a Distribution — Key Questions
</h3>

<ul>
<li>What is the main function? (Server or Desktop?)</li>
<li>What packages are important? (web server, database, etc.)</li>
<li>How much disk space is available?</li>
<li>How often are packages updated?</li>
<li>How long is the support cycle? (LTS = Long Term Support)</li>
<li>Do you need long-term stability or cutting-edge software?</li>
</ul>

<h3>
  
  
  Package Management (apt — Debian/Ubuntu)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>apt update                    <span class="c"># Update the package index (local DB of available repos)</span>
apt upgrade                   <span class="c"># Upgrade installed packages to latest versions</span>
apt <span class="nb">install</span> &lt;pkg_name&gt;        <span class="c"># Install a package</span>
apt remove &lt;pkg_name&gt;         <span class="c"># Remove a package</span>
apt search &lt;pkg_name&gt;         <span class="c"># Search for a package</span>
apt show &lt;pkg_name&gt;           <span class="c"># Show package details</span>
apt list <span class="nt">--installed</span>          <span class="c"># List all installed packages</span>
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> Always run <code>apt update</code> BEFORE <code>apt install</code>. Update refreshes the package list, upgrade actually updates the software.</p>
</blockquote>

<h3>
  
  
  Linux File System Hierarchy
</h3>

<p>Linux organizes everything in a tree structure starting from <code>/</code> (root):</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Directory</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/</code></td>
<td>Root — top of the entire filesystem tree</td>
</tr>
<tr>
<td><code>/home</code></td>
<td>User home directories (e.g., <code>/home/daniyal</code>)</td>
</tr>
<tr>
<td><code>/etc</code></td>
<td>System configuration files (ssh config, apache config, etc.)</td>
</tr>
<tr>
<td><code>/var</code></td>
<td>Variable data — logs (<code>/var/log</code>), web files (<code>/var/www</code>)</td>
</tr>
<tr>
<td><code>/usr</code></td>
<td>User programs and utilities (<code>/usr/bin</code>, <code>/usr/lib</code>)</td>
</tr>
<tr>
<td><code>/bin</code></td>
<td>Essential command binaries (<code>ls</code>, <code>cp</code>, <code>mv</code>, <code>cat</code>)</td>
</tr>
<tr>
<td><code>/sbin</code></td>
<td>System binaries (<code>systemctl</code>, <code>fdisk</code>) — need root</td>
</tr>
<tr>
<td><code>/tmp</code></td>
<td>Temporary files — cleared on reboot</td>
</tr>
<tr>
<td><code>/opt</code></td>
<td>Optional/third-party software</td>
</tr>
<tr>
<td><code>/root</code></td>
<td>Home directory of the root user</td>
</tr>
<tr>
<td><code>/dev</code></td>
<td>Device files (hardware represented as files)</td>
</tr>
<tr>
<td><code>/proc</code></td>
<td>Virtual filesystem — running process info</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>🔥 EXAM TIP:</strong> Know these paths by heart: <code>/etc/ssh/sshd_config</code>, <code>/var/www/html</code>, <code>/var/log/apache2/</code>, <code>/etc/apache2/sites-available/</code></p>
</blockquote>

<h3>
  
  
  OS and Kernel Information
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">uname</span> <span class="nt">-a</span>              <span class="c"># Full system info (kernel name, version, architecture)</span>
lsb_release <span class="nt">-a</span>        <span class="c"># Distribution-specific info</span>
<span class="nb">cat</span> /etc/os-release   <span class="c"># OS release details</span>
</code></pre>

</div>






<h2>
  
  
  2. Linux Commands &amp; Navigation
</h2>

<h3>
  
  
  Command Structure
</h3>

<p>A Linux command has three parts: <code>command [options] [arguments]</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">wc</span> <span class="nt">-l</span> devops_course_outline.txt
<span class="c">#  ^    ^    ^</span>
<span class="c">#  |    |    +-- argument (what to operate on)</span>
<span class="c">#  |    +------- option (modifies behavior, starts with - or --)</span>
<span class="c">#  +------------ command (program to execute)</span>
</code></pre>

</div>



<h3>
  
  
  The sudo Command
</h3>

<p><code>sudo</code> (superuser do) lets you run commands with root (administrator) privileges:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo ls</span> <span class="nt">-la</span> /root      <span class="c"># List root's home directory with elevated privileges</span>
<span class="nb">sudo </span>apt <span class="nb">install </span>nginx  <span class="c"># Install software (needs root)</span>
</code></pre>

</div>



<h3>
  
  
  Directory Navigation
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What It Does</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>pwd</code></td>
<td>Print current working directory</td>
<td>
<code>pwd</code> → <code>/home/daniyal</code>
</td>
</tr>
<tr>
<td><code>cd</code></td>
<td>Change directory</td>
<td><code>cd /var/www/html</code></td>
</tr>
<tr>
<td>
<code>cd ~</code> or <code>cd</code>
</td>
<td>Go to home directory</td>
<td><code>cd ~</code></td>
</tr>
<tr>
<td><code>cd ..</code></td>
<td>Go up one level (parent dir)</td>
<td><code>cd ..</code></td>
</tr>
<tr>
<td><code>cd -</code></td>
<td>Go to previous directory</td>
<td><code>cd -</code></td>
</tr>
<tr>
<td><code>ls</code></td>
<td>List directory contents</td>
<td><code>ls /etc</code></td>
</tr>
<tr>
<td><code>ls -la</code></td>
<td>List ALL files (hidden + details)</td>
<td><code>ls -la</code></td>
</tr>
<tr>
<td><code>tree</code></td>
<td>Show directory tree structure</td>
<td><code>tree /var/www</code></td>
</tr>
<tr>
<td><code>mkdir</code></td>
<td>Create a directory</td>
<td><code>mkdir myproject</code></td>
</tr>
<tr>
<td><code>mkdir -p</code></td>
<td>Create nested directories</td>
<td><code>mkdir -p a/b/c</code></td>
</tr>
<tr>
<td><code>rmdir</code></td>
<td>Remove empty directory</td>
<td><code>rmdir myproject</code></td>
</tr>
<tr>
<td><code>rm -r</code></td>
<td>Remove directory and contents</td>
<td><code>rm -r myproject</code></td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Absolute vs Relative Paths
</h3>

<ul>
<li>
<strong>Absolute path</strong>: Starts from root <code>/</code>. Example: <code>/usr/bin/python3</code>
</li>
<li>
<strong>Relative path</strong>: Starts from current directory. Example: <code>../usr/bin</code>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd</span> /usr/bin     <span class="c"># Absolute path</span>
<span class="nb">cd</span> ../usr/bin   <span class="c"># Relative path</span>
</code></pre>

</div>



<h3>
  
  
  Viewing Files
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>cat file.txt</code></td>
<td>Display entire file at once (not ideal for large files)</td>
</tr>
<tr>
<td><code>cat -n file.txt</code></td>
<td>Display with line numbers</td>
</tr>
<tr>
<td><code>tac file.txt</code></td>
<td>Display file in reverse (last line first)</td>
</tr>
<tr>
<td><code>less file.txt</code></td>
<td>Page through large files (scroll up/down, search with <code>/</code>)</td>
</tr>
<tr>
<td><code>head file.txt</code></td>
<td>Show first 10 lines (use <code>-n 20</code> for first 20)</td>
</tr>
<tr>
<td><code>tail file.txt</code></td>
<td>Show last 10 lines (use <code>-n 20</code> for last 20)</td>
</tr>
<tr>
<td><code>tail -f file.txt</code></td>
<td><strong>Follow file in real-time (great for logs!)</strong></td>
</tr>
<tr>
<td><code>wc file.txt</code></td>
<td>Count lines, words, characters</td>
</tr>
<tr>
<td><code>wc -l file.txt</code></td>
<td>Count only lines</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Creating &amp; Manipulating Files
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">touch </span>newfile.txt                    <span class="c"># Create empty file (or update timestamp)</span>
<span class="nb">touch</span> <span class="nt">-t</span> 12091600 file.txt           <span class="c"># Set specific timestamp (Dec 9, 4PM)</span>
<span class="nb">echo</span> <span class="s2">"hello"</span> <span class="o">&gt;</span> file.txt              <span class="c"># Write to file (OVERWRITES!)</span>
<span class="nb">echo</span> <span class="s2">"world"</span> <span class="o">&gt;&gt;</span> file.txt             <span class="c"># Append to file</span>
<span class="nb">cp </span>source.txt dest.txt               <span class="c"># Copy file</span>
<span class="nb">mv </span>old.txt new.txt                   <span class="c"># Rename/move file</span>
<span class="nb">rm </span>file.txt                          <span class="c"># Delete file</span>
<span class="nb">rm</span> <span class="nt">-rf</span> directory/                    <span class="c"># Force delete directory and contents</span>
</code></pre>

</div>



<h3>
  
  
  Searching for Files
</h3>

<h4>
  
  
  The <code>find</code> Command
</h4>

<p>The <code>find</code> command searches the filesystem in real-time. Extremely useful in DevOps for automation, cleanup, backups, and CI/CD scripts.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>find /var/log <span class="nt">-name</span> <span class="s2">"*.log"</span>           <span class="c"># Find all .log files in /var/log</span>
find /usr <span class="nt">-name</span> gcc                   <span class="c"># Find files/dirs named gcc</span>
find /usr <span class="nt">-type</span> d <span class="nt">-name</span> gcc           <span class="c"># Find only DIRECTORIES named gcc</span>
find /usr <span class="nt">-type</span> f <span class="nt">-name</span> gcc           <span class="c"># Find only regular FILES named gcc</span>
find / <span class="nt">-size</span> +10M                     <span class="c"># Find files larger than 10MB</span>
find / <span class="nt">-size</span> 0                        <span class="c"># Find empty files</span>
find / <span class="nt">-mtime</span> 3                       <span class="c"># Files modified 3 days ago</span>
find / <span class="nt">-user</span> root                     <span class="c"># Files owned by root</span>
find <span class="nt">-name</span> <span class="s2">"*.jpg"</span> <span class="nt">-exec</span> <span class="nb">rm</span> <span class="o">{}</span> <span class="s1">';'</span>   <span class="c"># Find and DELETE all .jpg files</span>
find / <span class="nt">-size</span> +10M <span class="nt">-exec</span> <span class="nb">command</span> <span class="o">{}</span> <span class="s1">';'</span>  <span class="c"># Find large files and run command</span>
</code></pre>

</div>



<p><strong>Key <code>find</code> Options:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Option</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>-name</code></td>
<td>Match filename (case-sensitive)</td>
</tr>
<tr>
<td><code>-iname</code></td>
<td>Match filename (case-insensitive)</td>
</tr>
<tr>
<td><code>-type f</code></td>
<td>Regular files only</td>
</tr>
<tr>
<td><code>-type d</code></td>
<td>Directories only</td>
</tr>
<tr>
<td><code>-size</code></td>
<td>File size (<code>+100M</code> = over 100MB, <code>-10k</code> = under 10KB)</td>
</tr>
<tr>
<td><code>-mtime</code></td>
<td>Modified time in days</td>
</tr>
<tr>
<td><code>-exec</code></td>
<td>Execute a command on each result</td>
</tr>
<tr>
<td><code>-user</code></td>
<td>Files owned by specific user</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>📝 NOTE:</strong> The <code>{}</code> is a placeholder filled with each file found. The <code>';'</code> or <code>\;</code> terminates the <code>-exec</code> command.</p>
</blockquote>

<h4>
  
  
  The <code>locate</code> Command
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>locate zip         <span class="c"># Fast search using pre-built database</span>
<span class="c"># Database is updated by 'updatedb' (runs daily automatically)</span>
</code></pre>

</div>



<h4>
  
  
  <code>which</code> &amp; <code>whereis</code>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>which python3      <span class="c"># Shows path of executable: /usr/bin/python3</span>
whereis python3    <span class="c"># Shows binary, source, and man page locations</span>
</code></pre>

</div>



<h3>
  
  
  I/O Redirection &amp; Pipes
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo</span> <span class="s2">"hello"</span> <span class="o">&gt;</span> file.txt       <span class="c"># Redirect stdout to file (overwrite)</span>
<span class="nb">echo</span> <span class="s2">"world"</span> <span class="o">&gt;&gt;</span> file.txt      <span class="c"># Redirect stdout to file (append)</span>
<span class="nb">command</span> &lt; input.txt           <span class="c"># Feed file as stdin to command</span>
<span class="nb">command </span>2&gt; errors.txt         <span class="c"># Redirect stderr to file</span>
<span class="nb">command</span> <span class="o">&gt;</span> out.txt 2&gt;&amp;1        <span class="c"># Redirect BOTH stdout AND stderr to file</span>
<span class="nb">cat</span> /etc/passwd | <span class="nb">wc</span> <span class="nt">-l</span>       <span class="c"># Pipe: send output of cat as input to wc</span>
<span class="nb">cat</span> /etc/passwd | <span class="nb">cut</span> <span class="nt">-d</span>: <span class="nt">-f1</span> | <span class="nb">sort</span> | <span class="nb">uniq</span>  <span class="c"># Chain multiple pipes</span>
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> <code>&gt;</code> overwrites, <code>&gt;&gt;</code> appends. <code>2&gt;&amp;1</code> redirects errors to same place as stdout. The pipe <code>|</code> chains commands together.</p>
</blockquote>

<h3>
  
  
  Process Management
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">kill</span> <span class="nt">-SIGKILL</span> &lt;PID&gt;       <span class="c"># Force kill a process (same as kill -9)</span>
<span class="nb">kill</span> <span class="nt">-9</span> &lt;PID&gt;             <span class="c"># Force kill by PID</span>
node server.js &amp;          <span class="c"># Run process in background (&amp; suffix)</span>
lsof <span class="nt">-i</span> :80              <span class="c"># Find what process is using port 80</span>
</code></pre>

</div>



<blockquote>
<p><strong>📝 NOTE:</strong> You can only kill your own processes (unless you are root).</p>
</blockquote>

<h3>
  
  
  Getting Help
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>man apache2       <span class="c"># Full manual page for a command</span>
man <span class="nt">-k</span> compress   <span class="c"># Search man pages by keyword</span>
info make         <span class="c"># GNU Info documentation (alternative to man)</span>
<span class="nb">command</span> <span class="nt">--help</span>    <span class="c"># Quick help/usage summary (fastest option)</span>
</code></pre>

</div>






<h2>
  
  
  3. File Permissions &amp; Ownership
</h2>

<h3>
  
  
  Three Types of Owners
</h3>

<ul>
<li>
<strong>User (u)</strong>: The owner of the file. When you create a file, you become the owner.</li>
<li>
<strong>Group (g)</strong>: A collection of users. Useful in multi-user environments.</li>
<li>
<strong>Other (o)</strong>: Everyone else on the system.</li>
</ul>

<h3>
  
  
  Three Types of Permissions
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Permission</th>
<th>Symbol</th>
<th>Number</th>
<th>On Files</th>
<th>On Directories</th>
</tr>
</thead>
<tbody>
<tr>
<td>Read</td>
<td><code>r</code></td>
<td>4</td>
<td>View/copy file contents</td>
<td>List directory contents (<code>ls</code>)</td>
</tr>
<tr>
<td>Write</td>
<td><code>w</code></td>
<td>2</td>
<td>Modify file contents</td>
<td>Create/delete files inside</td>
</tr>
<tr>
<td>Execute</td>
<td><code>x</code></td>
<td>1</td>
<td>Run the file as program</td>
<td>Enter the directory (<code>cd</code>)</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Octal Permission Table (MEMORIZE THIS!)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Number</th>
<th>0</th>
<th>1</th>
<th>2</th>
<th>3</th>
<th>4</th>
<th>5</th>
<th>6</th>
<th>7</th>
</tr>
</thead>
<tbody>
<tr>
<td>Permission</td>
<td><code>---</code></td>
<td><code>--x</code></td>
<td><code>-w-</code></td>
<td><code>-wx</code></td>
<td><code>r--</code></td>
<td><code>r-x</code></td>
<td><code>rw-</code></td>
<td><code>rwx</code></td>
</tr>
<tr>
<td>Meaning</td>
<td>None</td>
<td>Exec only</td>
<td>Write only</td>
<td>Write+Exec</td>
<td>Read only</td>
<td>Read+Exec</td>
<td>Read+Write</td>
<td>ALL</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Reading Permission Strings
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>-rw-r--r-- 1 sajid devops_students 13 2022-05-11 08:29 lecture.txt
 ^^^^^^^^^   ^     ^
 |           |     +-- group owner
 |           +-------- file owner (user)
 +-------------------- permission string

First char: - = file, d = directory, l = symlink
Next 9 chars: rwx for user | rwx for group | rwx for others

Example: -rw-r--r-- means:
  User:  rw-  (read + write)      = 4+2+0 = 6
  Group: r--  (read only)          = 4+0+0 = 4
  Other: r--  (read only)          = 4+0+0 = 4
  Octal: 644
</code></pre>

</div>



<h3>
  
  
  <code>chmod</code> — Change Permissions
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">chmod </span>644 file.txt         <span class="c"># rw-r--r-- (standard file permission)</span>
<span class="nb">chmod </span>755 directory/       <span class="c"># rwxr-xr-x (standard directory permission)</span>
<span class="nb">chmod </span>700 secret.txt       <span class="c"># rwx------ (owner only, full access)</span>
<span class="nb">chmod </span>666 file.txt         <span class="c"># rw-rw-rw- (everyone can read/write)</span>
<span class="nb">chmod</span> +x script.sh         <span class="c"># Add execute permission for everyone</span>
<span class="nb">chmod </span>u+x script.sh        <span class="c"># Add execute for user only</span>
<span class="nb">chmod </span>g-w file.txt         <span class="c"># Remove write from group</span>
<span class="nb">chmod </span>o-rwx file.txt       <span class="c"># Remove all permissions from others</span>
</code></pre>

</div>



<h3>
  
  
  <code>chown</code> — Change Ownership
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo chown </span>root:root file.txt           <span class="c"># Change owner AND group to root</span>
<span class="nb">sudo chown </span>daniyal file.txt             <span class="c"># Change owner only</span>
<span class="nb">sudo chown </span>daniyal:www-data file.txt    <span class="c"># Change owner to daniyal, group to www-data</span>
<span class="nb">sudo chown</span> <span class="nt">-R</span> www-data:www-data /var/www/html/  <span class="c"># Recursive (all files inside)</span>
</code></pre>

</div>



<h3>
  
  
  ⭐ DevOps Golden Rules for Permissions
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Rule</th>
<th>Permission</th>
<th>Why</th>
</tr>
</thead>
<tbody>
<tr>
<td>Directories</td>
<td>
<code>755</code> (rwxr-xr-x)</td>
<td>Owner full access, others can read/enter</td>
</tr>
<tr>
<td>Regular files</td>
<td>
<code>644</code> (rw-r--r--)</td>
<td>Owner read/write, others read only</td>
</tr>
<tr>
<td>Scripts (.sh)</td>
<td>
<code>755</code> (rwxr-xr-x)</td>
<td>Must be executable</td>
</tr>
<tr>
<td>SSH private key</td>
<td>
<code>600</code> (rw-------)</td>
<td>ONLY owner can read (SSH refuses otherwise)</td>
</tr>
<tr>
<td>~/.ssh/ directory</td>
<td>
<code>700</code> (rwx------)</td>
<td>ONLY owner can access</td>
</tr>
<tr>
<td>authorized_keys</td>
<td>
<code>600</code> (rw-------)</td>
<td>ONLY owner can read/write</td>
</tr>
<tr>
<td>Web files owner</td>
<td><code>www-data:www-data</code></td>
<td>Web server user must be able to read</td>
</tr>
<tr>
<td><strong>NEVER use</strong></td>
<td><strong><code>777</code></strong></td>
<td><strong>SECURITY RISK — everyone has full access!</strong></td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong>🔥 EXAM TIP:</strong> If a web page shows '403 Forbidden', first check permissions and ownership! Owner should be <code>www-data</code>, dirs <code>755</code>, files <code>644</code>.</p>
</blockquote>




<h2>
  
  
  4. Users &amp; Groups Management
</h2>

<h3>
  
  
  User Management Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cat</span> /etc/passwd                    <span class="c"># List ALL users on the system</span>
<span class="nb">sudo </span>adduser newuser               <span class="c"># Create new user (interactive - sets password)</span>
<span class="nb">sudo </span>useradd newuser               <span class="c"># Create user (non-interactive - no password set)</span>
<span class="nb">sudo </span>usermod <span class="nt">-aG</span> <span class="nb">sudo </span>newuser      <span class="c"># Add user to sudo group (give admin rights)</span>
<span class="nb">sudo </span>userdel newuser               <span class="c"># Delete a user</span>
<span class="nb">sudo </span>userdel <span class="nt">-r</span> newuser            <span class="c"># Delete user AND their home directory</span>
su - newuser                       <span class="c"># Switch to another user</span>
<span class="nb">whoami</span>                             <span class="c"># Show current username</span>
<span class="nb">id</span>                                 <span class="c"># Show user ID, group ID, and groups</span>
</code></pre>

</div>



<h3>
  
  
  Group Management Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cat</span> /etc/group                     <span class="c"># List all groups</span>
<span class="nb">sudo </span>groupadd devopsgroup          <span class="c"># Create a new group</span>
<span class="nb">sudo </span>usermod <span class="nt">-aG</span> devopsgroup user1 <span class="c"># Add user1 to devopsgroup</span>
<span class="nb">sudo </span>groupdel devopsgroup          <span class="c"># Delete a group</span>
<span class="nb">groups </span>username                    <span class="c"># Show which groups a user belongs to</span>
</code></pre>

</div>



<blockquote>
<p><strong>📝 NOTE:</strong> The <code>-aG</code> flag is critical! Without <code>-a</code>, the user gets REMOVED from all other groups. Always use <code>-aG</code> together.</p>
</blockquote>

<h3>
  
  
  Important Files
</h3>

<ul>
<li>
<code>/etc/passwd</code> — Contains user account info (<code>username:x:UID:GID:info:home:shell</code>)</li>
<li>
<code>/etc/shadow</code> — Contains encrypted passwords (only root can read)</li>
<li>
<code>/etc/group</code> — Contains group definitions</li>
</ul>




<h2>
  
  
  5. SSH (Secure Shell)
</h2>

<h3>
  
  
  What is SSH?
</h3>

<p>SSH (Secure Shell) allows you to connect and log into remote systems securely. It encrypts ALL communication between client and server. SSH runs on <strong>port 22</strong> by default. The server must have <strong>sshd</strong> (SSH daemon) installed and running.</p>

<h3>
  
  
  Basic SSH Connection
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ssh username@remote_server_ip       <span class="c"># Connect to remote machine</span>
ssh daniyal@192.168.1.100           <span class="c"># Example with IP</span>
<span class="nb">whoami</span>                               <span class="c"># Verify you are logged in</span>
<span class="nb">hostname</span>                             <span class="c"># Verify which machine you are on</span>
<span class="nb">exit</span>                                 <span class="c"># Disconnect from remote</span>
</code></pre>

</div>



<h3>
  
  
  SSH Service Control
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>systemctl status ssh          <span class="c"># Check if SSH is running</span>
<span class="nb">sudo </span>systemctl start ssh           <span class="c"># Start SSH service</span>
<span class="nb">sudo </span>systemctl stop ssh            <span class="c"># Stop SSH service</span>
<span class="nb">sudo </span>systemctl restart ssh         <span class="c"># Restart (apply config changes)</span>
<span class="nb">sudo </span>systemctl <span class="nb">enable </span>ssh          <span class="c"># Start automatically on boot</span>
<span class="nb">sudo </span>systemctl disable ssh         <span class="c"># Don't start on boot</span>
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> <code>systemctl enable</code> makes service start on boot. <code>systemctl start</code> starts it NOW. They are different! You often need BOTH.</p>
</blockquote>

<h3>
  
  
  SSH Configuration File
</h3>

<p>The main SSH server config file is: <code>/etc/ssh/sshd_config</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nano /etc/ssh/sshd_config     <span class="c"># Edit SSH server configuration</span>

<span class="c"># Key settings to know:</span>
Port 22                            <span class="c"># Default SSH port</span>
PermitRootLogin <span class="nb">yes</span>/no             <span class="c"># Allow/block root login via SSH</span>
PasswordAuthentication <span class="nb">yes</span>/no      <span class="c"># Allow/block password-based login</span>
PubkeyAuthentication <span class="nb">yes</span>           <span class="c"># Allow key-based authentication</span>

<span class="c"># IMPORTANT: After ANY change, restart SSH!</span>
<span class="nb">sudo </span>systemctl restart ssh
</code></pre>

</div>



<h3>
  
  
  SSH Key-Based Authentication
</h3>

<p>Key-based auth is more secure than passwords. It uses a pair of keys:</p>

<ul>
<li>
<strong>Private key</strong> (<code>~/.ssh/id_rsa</code>): Stays on YOUR machine. NEVER share this!</li>
<li>
<strong>Public key</strong> (<code>~/.ssh/id_rsa.pub</code>): Goes to the REMOTE server.</li>
</ul>

<p><strong>Step-by-step setup:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># STEP 1: Generate key pair on YOUR machine (client)</span>
ssh-keygen <span class="nt">-t</span> rsa
<span class="c"># Press Enter for default location (~/.ssh/id_rsa)</span>
<span class="c"># Press Enter for no passphrase (or set one for extra security)</span>
<span class="c"># Creates: ~/.ssh/id_rsa (private) and ~/.ssh/id_rsa.pub (public)</span>

<span class="c"># STEP 2: Copy public key to the REMOTE server</span>
ssh-copy-id username@remote_server
<span class="c"># OR manually:</span>
<span class="nb">cat</span> ~/.ssh/id_rsa.pub <span class="o">&gt;&gt;</span> ~/.ssh/authorized_keys  <span class="c"># On the remote server</span>

<span class="c"># STEP 3: Set correct permissions (CRITICAL!)</span>
<span class="nb">chmod </span>700 ~/.ssh                   <span class="c"># Directory: owner only</span>
<span class="nb">chmod </span>600 ~/.ssh/id_rsa            <span class="c"># Private key: owner only</span>
<span class="nb">chmod </span>600 ~/.ssh/authorized_keys   <span class="c"># Auth keys: owner only</span>

<span class="c"># STEP 4: Now you can login without password!</span>
ssh username@remote_server
</code></pre>

</div>



<blockquote>
<p><strong>📝 NOTE:</strong> The remote server stores public keys in <code>~/.ssh/authorized_keys</code> (one key per line). If permissions on this file are wrong, SSH will REFUSE the key and fall back to password auth.</p>
</blockquote>

<h3>
  
  
  SSH Config File (Client Side)
</h3>

<p>Create <code>~/.ssh/config</code> to avoid typing username and IP every time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># File: ~/.ssh/config</span>
Host myserver
    HostName 192.168.1.100
    User daniyal
    IdentityFile ~/.ssh/id_rsa

<span class="c"># Now you can just type:</span>
ssh myserver
<span class="c"># Instead of: ssh daniyal@192.168.1.100</span>
</code></pre>

</div>



<h3>
  
  
  Non-Interactive SSH Commands
</h3>

<p>Run commands on a remote server without opening a full session:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ssh user@server <span class="s2">"whoami"</span>                    <span class="c"># Run whoami on remote</span>
ssh user@server <span class="s2">"mkdir ~/test_dir"</span>          <span class="c"># Create dir on remote</span>
ssh user@server <span class="s2">"cat /etc/hostname"</span>         <span class="c"># Read remote file</span>
ssh user@server <span class="s2">"systemctl restart apache2"</span> <span class="c"># Restart service remotely</span>
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> Non-interactive SSH is key for scripting — you can loop over servers: <code>for server in list; do ssh server "command"; done</code></p>
</blockquote>

<h3>
  
  
  Disabling Root Login &amp; Password Auth
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Edit /etc/ssh/sshd_config:</span>
PermitRootLogin no              <span class="c"># Block root from logging in via SSH</span>
PasswordAuthentication no       <span class="c"># Force key-based auth only</span>

<span class="c"># Restart to apply:</span>
<span class="nb">sudo </span>systemctl restart ssh

<span class="c"># Test: trying ssh root@server should now be DENIED</span>
</code></pre>

</div>






<h2>
  
  
  6. Apache Web Server
</h2>

<h3>
  
  
  Installation &amp; Service Management
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt update
<span class="nb">sudo </span>apt <span class="nb">install </span>apache2           <span class="c"># Install Apache</span>

<span class="nb">sudo </span>systemctl start apache2       <span class="c"># Start the service</span>
<span class="nb">sudo </span>systemctl stop apache2        <span class="c"># Stop the service</span>
<span class="nb">sudo </span>systemctl restart apache2     <span class="c"># Restart (apply changes)</span>
<span class="nb">sudo </span>systemctl reload apache2      <span class="c"># Reload config without downtime</span>
<span class="nb">sudo </span>systemctl status apache2      <span class="c"># Check if running</span>
<span class="nb">sudo </span>systemctl <span class="nb">enable </span>apache2      <span class="c"># Start on boot</span>

curl localhost                     <span class="c"># Test: should show Apache default page</span>
</code></pre>

</div>



<h3>
  
  
  Apache runs on Port 80 (HTTP) or Port 443 (HTTPS)
</h3>

<h3>
  
  
  Apache Default Paths
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Path</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/var/www/html/</code></td>
<td>Default web root (put your website files here)</td>
</tr>
<tr>
<td><code>/etc/apache2/</code></td>
<td>Main configuration directory</td>
</tr>
<tr>
<td><code>/etc/apache2/sites-available/</code></td>
<td>Virtual host config files (available)</td>
</tr>
<tr>
<td><code>/etc/apache2/sites-enabled/</code></td>
<td>Symlinks to active virtual hosts</td>
</tr>
<tr>
<td><code>/etc/apache2/apache2.conf</code></td>
<td>Main Apache configuration file</td>
</tr>
<tr>
<td><code>/etc/apache2/ports.conf</code></td>
<td>Which ports Apache listens on</td>
</tr>
<tr>
<td><code>/var/log/apache2/error.log</code></td>
<td>Error log file</td>
</tr>
<tr>
<td><code>/var/log/apache2/access.log</code></td>
<td>Access log file</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Deploying a Static Website
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step 1: Clone or copy your website files</span>
<span class="nb">sudo </span>git clone https://github.com/user/repo.git /var/www/html/mysite

<span class="c"># Step 2: Set correct ownership</span>
<span class="nb">sudo chown</span> <span class="nt">-R</span> www-data:www-data /var/www/html/mysite

<span class="c"># Step 3: Set correct permissions</span>
<span class="nb">sudo chmod</span> <span class="nt">-R</span> 755 /var/www/html/mysite/                        <span class="c"># Directories</span>
<span class="nb">sudo </span>find /var/www/html/mysite <span class="nt">-type</span> f <span class="nt">-exec</span> <span class="nb">chmod </span>644 <span class="o">{}</span> <span class="se">\;</span>   <span class="c"># Files</span>

<span class="c"># Step 4: Verify</span>
curl localhost
</code></pre>

</div>



<h3>
  
  
  Virtual Hosts (Multiple Websites)
</h3>

<p>Virtual hosts let you run MULTIPLE websites on one server, each on a different port or domain.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step 1: Create config file in sites-available</span>
<span class="nb">sudo </span>nano /etc/apache2/sites-available/mysite.conf
</code></pre>

</div>



<p>Content of <code>mysite.conf</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight apache"><code><span class="p">&lt;</span><span class="nl">VirtualHost</span><span class="sr"> *:8080</span><span class="p">&gt;
</span>    <span class="nc">ServerName</span> mysite.local
    <span class="nc">DocumentRoot</span> /var/www/mysite
    <span class="nc">ErrorLog</span> /var/log/apache2/mysite-error.log
    <span class="nc">CustomLog</span> /var/log/apache2/mysite-access.log combined
<span class="p">&lt;/</span><span class="nl">VirtualHost</span><span class="p">&gt;
</span></code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step 2: Add Listen directive for new port</span>
<span class="c"># Edit /etc/apache2/ports.conf and add: Listen 8080</span>

<span class="c"># Step 3: Enable the site</span>
<span class="nb">sudo </span>a2ensite mysite.conf

<span class="c"># Step 4: Test config and reload</span>
<span class="nb">sudo </span>apache2ctl configtest         <span class="c"># Check for syntax errors</span>
<span class="nb">sudo </span>systemctl reload apache2

<span class="c"># Step 5: Verify</span>
curl localhost:8080
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> <code>sites-available</code> = config files stored here. <code>sites-enabled</code> = symlinks to active configs. Use <code>a2ensite</code>/<code>a2dissite</code> to enable/disable.</p>
</blockquote>




<h2>
  
  
  7. Nginx Web Server
</h2>

<h3>
  
  
  Installation &amp; Service Management
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt update
<span class="nb">sudo </span>apt <span class="nb">install </span>nginx

<span class="nb">sudo </span>systemctl start nginx
<span class="nb">sudo </span>systemctl stop nginx
<span class="nb">sudo </span>systemctl restart nginx
<span class="nb">sudo </span>systemctl status nginx
<span class="nb">sudo </span>systemctl <span class="nb">enable </span>nginx

curl localhost                     <span class="c"># Test</span>
</code></pre>

</div>



<h3>
  
  
  Nginx Key Paths
</h3>

<ul>
<li>Default web root: <code>/var/www/html/</code> (same as Apache)</li>
<li>Config files: <code>/etc/nginx/sites-available/</code> and <code>/etc/nginx/sites-enabled/</code>
</li>
<li>Main config: <code>/etc/nginx/nginx.conf</code>
</li>
<li>Default site config: <code>/etc/nginx/sites-available/default</code>
</li>
</ul>

<h3>
  
  
  Nginx Virtual Host (Server Block)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="c1"># File: /etc/nginx/sites-available/mysite</span>
<span class="k">server</span> <span class="p">{</span>
    <span class="kn">listen</span> <span class="mi">80</span><span class="p">;</span>
    <span class="kn">server_name</span> <span class="s">mysite.local</span><span class="p">;</span>
    <span class="kn">root</span> <span class="n">/var/www/mysite</span><span class="p">;</span>
    <span class="kn">index</span> <span class="s">index.html</span><span class="p">;</span>

    <span class="kn">location</span> <span class="n">/</span> <span class="p">{</span>
        <span class="kn">try_files</span> <span class="nv">$uri</span> <span class="nv">$uri</span><span class="n">/</span> <span class="p">=</span><span class="mi">404</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Enable the site by creating a symlink:</span>
<span class="nb">sudo ln</span> <span class="nt">-s</span> /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

<span class="c"># Test config:</span>
<span class="nb">sudo </span>nginx <span class="nt">-t</span>

<span class="c"># Reload:</span>
<span class="nb">sudo </span>systemctl reload nginx
</code></pre>

</div>



<blockquote>
<p><strong>📝 NOTE:</strong> Both Apache and Nginx use port 80 by default. They CANNOT run on the same port at the same time!</p>
</blockquote>




<h2>
  
  
  8. Port Debugging &amp; Troubleshooting
</h2>

<h3>
  
  
  Common Scenario: Port Conflict
</h3>

<p>If Apache fails to start with <strong>"Address already in use"</strong> error, another process (like Nginx) is already using port 80.</p>

<h3>
  
  
  Debugging Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check what's using a specific port</span>
<span class="nb">sudo </span>lsof <span class="nt">-i</span> :80                   <span class="c"># Show process using port 80</span>

<span class="c"># Production-friendly command (all open network connections)</span>
<span class="nb">sudo </span>lsof <span class="nt">-i</span> <span class="nt">-P</span> <span class="nt">-n</span>
<span class="c"># -i = open network connections</span>
<span class="c"># -P = show port numbers (not service names)</span>
<span class="c"># -n = don't resolve hostnames</span>

<span class="c"># Check service logs for errors</span>
<span class="nb">sudo </span>journalctl <span class="nt">-xe</span>                <span class="c"># Show recent system logs with details</span>
<span class="nb">sudo </span>journalctl <span class="nt">-u</span> apache2         <span class="c"># Show only Apache logs</span>

<span class="c"># Monitor log files in real-time</span>
<span class="nb">sudo tail</span> <span class="nt">-f</span> /var/log/apache2/error.log
<span class="nb">sudo tail</span> <span class="nt">-f</span> /var/log/nginx/error.log
</code></pre>

</div>



<h3>
  
  
  Fixing Port Conflicts Step by Step
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step 1: Check which service is using port 80</span>
<span class="nb">sudo </span>lsof <span class="nt">-i</span> :80
<span class="c"># Output shows: nginx (PID 1234)</span>

<span class="c"># Step 2: Stop the conflicting service</span>
<span class="nb">sudo </span>systemctl stop nginx

<span class="c"># Step 3: Start the service you want</span>
<span class="nb">sudo </span>systemctl start apache2

<span class="c"># Step 4: Verify</span>
<span class="nb">sudo </span>systemctl status apache2
curl localhost
</code></pre>

</div>



<h3>
  
  
  Common Web Server Errors &amp; Fixes
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Error</th>
<th>Likely Cause</th>
<th>Fix</th>
</tr>
</thead>
<tbody>
<tr>
<td>403 Forbidden</td>
<td>Wrong permissions/ownership</td>
<td>
<code>chown www-data:www-data</code>, <code>chmod 755</code>/<code>644</code>
</td>
</tr>
<tr>
<td>404 Not Found</td>
<td>Wrong DocumentRoot path</td>
<td>Check config file, verify path exists</td>
</tr>
<tr>
<td>Address in use</td>
<td>Port conflict</td>
<td>
<code>lsof -i :80</code>, stop conflicting service</td>
</tr>
<tr>
<td>Service won't start</td>
<td>Config syntax error</td>
<td>
<code>apache2ctl configtest</code> or <code>nginx -t</code>
</td>
</tr>
<tr>
<td>Connection refused</td>
<td>Service not running</td>
<td><code>systemctl start apache2/nginx</code></td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  9. Bash Scripting
</h2>

<h3>
  
  
  Script Basics
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># The first line is the SHEBANG — tells the system to use bash</span>
<span class="c"># Every script MUST start with #!/bin/bash</span>

<span class="c"># Make script executable:</span>
<span class="nb">chmod</span> +x script.sh

<span class="c"># Run script:</span>
./script.sh
<span class="c"># OR</span>
bash script.sh
</code></pre>

</div>



<h3>
  
  
  Variables
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nv">NAME</span><span class="o">=</span><span class="s2">"Daniyal"</span>                     <span class="c"># Assign (NO spaces around =)</span>
<span class="nb">echo</span> <span class="s2">"Hello </span><span class="nv">$NAME</span><span class="s2">"</span>                 <span class="c"># Use variable with $</span>
<span class="nb">echo</span> <span class="s2">"Hello </span><span class="k">${</span><span class="nv">NAME</span><span class="k">}</span><span class="s2">"</span>               <span class="c"># Curly braces version (safer)</span>

<span class="c"># Read user input</span>
<span class="nb">echo</span> <span class="s2">"Enter your name:"</span>
<span class="nb">read </span>USERNAME                      <span class="c"># Stores input in USERNAME</span>
<span class="nb">echo</span> <span class="s2">"Welcome </span><span class="nv">$USERNAME</span><span class="s2">"</span>

<span class="c"># Command substitution — store command output in variable</span>
<span class="nv">TODAY</span><span class="o">=</span><span class="si">$(</span><span class="nb">date</span><span class="si">)</span>                      <span class="c"># Run date command, store result</span>
<span class="nb">echo</span> <span class="s2">"Today is: </span><span class="nv">$TODAY</span><span class="s2">"</span>

<span class="nv">HOSTNAME</span><span class="o">=</span><span class="si">$(</span><span class="nb">hostname</span><span class="si">)</span>
<span class="nb">echo</span> <span class="s2">"Machine: </span><span class="nv">$HOSTNAME</span><span class="s2">"</span>
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> NO SPACES around <code>=</code> when assigning variables! <code>NAME='value'</code> is correct. <code>NAME = 'value'</code> is WRONG and will cause an error.</p>
</blockquote>

<h3>
  
  
  Command Line Arguments
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Variable</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>$0</code></td>
<td>Script name itself</td>
</tr>
<tr>
<td>
<code>$1</code>, <code>$2</code>, <code>$3</code>...</td>
<td>First, second, third argument...</td>
</tr>
<tr>
<td><code>$#</code></td>
<td>Total number of arguments passed</td>
</tr>
<tr>
<td><code>$@</code></td>
<td>All arguments as separate words</td>
</tr>
<tr>
<td><code>$?</code></td>
<td>Exit code of the LAST command (0=success, non-0=failure)</td>
</tr>
</tbody>
</table></div>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Example: ./myscript.sh hello world</span>
<span class="nb">echo</span> <span class="s2">"Script name: </span><span class="nv">$0</span><span class="s2">"</span>            <span class="c"># ./myscript.sh</span>
<span class="nb">echo</span> <span class="s2">"First arg: </span><span class="nv">$1</span><span class="s2">"</span>              <span class="c"># hello</span>
<span class="nb">echo</span> <span class="s2">"Second arg: </span><span class="nv">$2</span><span class="s2">"</span>             <span class="c"># world</span>
<span class="nb">echo</span> <span class="s2">"Total args: $#"</span>             <span class="c"># 2</span>
<span class="nb">echo</span> <span class="s2">"All args: </span><span class="nv">$@</span><span class="s2">"</span>               <span class="c"># hello world</span>
</code></pre>

</div>



<h3>
  
  
  Exit Codes
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Every command returns an exit code</span>
<span class="c"># 0 = SUCCESS, anything else = FAILURE</span>

<span class="nb">ls</span> /tmp
<span class="nb">echo</span> <span class="nv">$?</span>    <span class="c"># 0 (success — /tmp exists)</span>

<span class="nb">ls</span> /nonexistent
<span class="nb">echo</span> <span class="nv">$?</span>    <span class="c"># 2 (failure — directory doesn't exist)</span>

<span class="c"># Set your own exit code</span>
<span class="nb">exit </span>0     <span class="c"># Exit script with success</span>
<span class="nb">exit </span>1     <span class="c"># Exit script with failure</span>
</code></pre>

</div>



<h3>
  
  
  Conditionals (if/else)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Basic if/else</span>
<span class="k">if</span> <span class="o">[</span> <span class="nt">-d</span> <span class="s2">"/var/www/html"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"Directory exists"</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"Directory does NOT exist"</span>
<span class="k">fi</span>

<span class="c"># Check if file exists</span>
<span class="k">if</span> <span class="o">[</span> <span class="nt">-f</span> <span class="s2">"/etc/ssh/sshd_config"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"SSH config found"</span>
<span class="k">fi</span>

<span class="c"># Compare strings</span>
<span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="o">==</span> <span class="s2">"install"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"Installing..."</span>
<span class="k">elif</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="o">==</span> <span class="s2">"remove"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"Removing..."</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"Usage: </span><span class="nv">$0</span><span class="s2"> install|remove"</span>
    <span class="nb">exit </span>1
<span class="k">fi</span>

<span class="c"># Check if command succeeded</span>
apt <span class="nb">install </span>nginx <span class="nt">-y</span>
<span class="k">if</span> <span class="o">[</span> <span class="nv">$?</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"Installation successful"</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"Installation FAILED"</span>
    <span class="nb">exit </span>1
<span class="k">fi</span>
</code></pre>

</div>



<p><strong>Common test operators:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Operator</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>-f file</code></td>
<td>File exists and is a regular file</td>
</tr>
<tr>
<td><code>-d dir</code></td>
<td>Directory exists</td>
</tr>
<tr>
<td><code>-e path</code></td>
<td>Path exists (file or directory)</td>
</tr>
<tr>
<td><code>-z string</code></td>
<td>String is empty (zero length)</td>
</tr>
<tr>
<td><code>-n string</code></td>
<td>String is NOT empty</td>
</tr>
<tr>
<td><code>str1 == str2</code></td>
<td>Strings are equal</td>
</tr>
<tr>
<td><code>str1 != str2</code></td>
<td>Strings are NOT equal</td>
</tr>
<tr>
<td><code>num1 -eq num2</code></td>
<td>Numbers are equal</td>
</tr>
<tr>
<td><code>num1 -ne num2</code></td>
<td>Numbers are NOT equal</td>
</tr>
<tr>
<td><code>num1 -gt num2</code></td>
<td>Greater than</td>
</tr>
<tr>
<td><code>num1 -lt num2</code></td>
<td>Less than</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Case Statement (for Flags)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Great for handling command-line flags like -i, -r, -h</span>
<span class="k">case</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="k">in</span>
    <span class="nt">-i</span><span class="p">|</span><span class="nt">--install</span><span class="p">)</span>
        <span class="nb">echo</span> <span class="s2">"Installing..."</span>
        <span class="p">;;</span>
    <span class="nt">-r</span><span class="p">|</span><span class="nt">--remove</span><span class="p">)</span>
        <span class="nb">echo</span> <span class="s2">"Removing..."</span>
        <span class="p">;;</span>
    <span class="nt">-h</span><span class="p">|</span><span class="nt">--help</span><span class="p">)</span>
        <span class="nb">echo</span> <span class="s2">"Usage: </span><span class="nv">$0</span><span class="s2"> [-i install] [-r remove] [-h help]"</span>
        <span class="p">;;</span>
    <span class="k">*</span><span class="p">)</span>
        <span class="nb">echo</span> <span class="s2">"Invalid option: </span><span class="nv">$1</span><span class="s2">"</span>
        <span class="nb">echo</span> <span class="s2">"Use -h for help"</span>
        <span class="nb">exit </span>1
        <span class="p">;;</span>
<span class="k">esac</span>
</code></pre>

</div>



<h3>
  
  
  For Loops
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Loop through a list</span>
<span class="k">for </span>server <span class="k">in </span>server1 server2 server3<span class="p">;</span> <span class="k">do
    </span><span class="nb">echo</span> <span class="s2">"Restarting </span><span class="nv">$server</span><span class="s2">..."</span>
    ssh <span class="nv">$server</span> <span class="s2">"systemctl restart apache2"</span>
<span class="k">done</span>

<span class="c"># Loop through files</span>
<span class="k">for </span>file <span class="k">in</span> <span class="k">*</span>.sh<span class="p">;</span> <span class="k">do
    </span><span class="nb">echo</span> <span class="s2">"Found script: </span><span class="nv">$file</span><span class="s2">"</span>
<span class="k">done</span>

<span class="c"># Loop with numbers</span>
<span class="k">for </span>i <span class="k">in </span>1 2 3 4 5<span class="p">;</span> <span class="k">do
    </span><span class="nb">echo</span> <span class="s2">"Number: </span><span class="nv">$i</span><span class="s2">"</span>
<span class="k">done</span>

<span class="c"># C-style loop</span>
<span class="k">for</span> <span class="o">((</span><span class="nv">i</span><span class="o">=</span>1<span class="p">;</span> i&lt;<span class="o">=</span>5<span class="p">;</span> i++<span class="o">))</span><span class="p">;</span> <span class="k">do
    </span><span class="nb">echo</span> <span class="s2">"Count: </span><span class="nv">$i</span><span class="s2">"</span>
<span class="k">done</span>
</code></pre>

</div>



<h3>
  
  
  The <code>tar</code> Command (Archives/Backups)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create a compressed archive</span>
<span class="nb">tar</span> <span class="nt">-czf</span> backup.tar.gz /path/to/directory
<span class="c"># -c = create</span>
<span class="c"># -z = compress with gzip</span>
<span class="c"># -f = filename follows</span>

<span class="c"># Create with timestamp in name</span>
<span class="nb">tar</span> <span class="nt">-czf</span> backup_<span class="si">$(</span><span class="nb">date</span> +%Y%m%d_%H%M%S<span class="si">)</span>.tar.gz /var/www/html/

<span class="c"># Extract an archive</span>
<span class="nb">tar</span> <span class="nt">-xzf</span> backup.tar.gz
<span class="c"># -x = extract</span>

<span class="c"># List contents without extracting</span>
<span class="nb">tar</span> <span class="nt">-tzf</span> backup.tar.gz
</code></pre>

</div>



<h3>
  
  
  The <code>awk</code> Command
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># awk processes text line by line, splitting into fields</span>
<span class="c"># Default field separator is whitespace</span>

<span class="c"># Print second column of output</span>
<span class="nb">df</span> <span class="nt">-h</span> | <span class="nb">awk</span> <span class="s1">'{print $2}'</span>

<span class="c"># Print specific fields with custom format</span>
free <span class="nt">-m</span> | <span class="nb">awk</span> <span class="s1">'/Mem:/ {print "Total: "$2" MB, Used: "$3" MB"}'</span>

<span class="c"># Use custom field separator</span>
<span class="nb">cat</span> /etc/passwd | <span class="nb">awk</span> <span class="nt">-F</span>: <span class="s1">'{print $1}'</span>   <span class="c"># Print usernames (field 1, : separator)</span>

<span class="c"># Filter lines matching a pattern</span>
<span class="nb">df</span> <span class="nt">-h</span> | <span class="nb">awk</span> <span class="s1">'!/tmpfs/ {print $0}'</span>        <span class="c"># Exclude lines containing tmpfs</span>
</code></pre>

</div>



<h3>
  
  
  Heredoc (Writing Multi-line Content)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Heredoc writes multi-line content to a file</span>
<span class="nb">cat</span> <span class="o">&lt;&lt;</span> <span class="no">EOF</span><span class="sh"> &gt; /var/www/html/index.html
&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;&lt;title&gt;System Report&lt;/title&gt;&lt;/head&gt;
&lt;body&gt;
    &lt;h1&gt;Server: </span><span class="si">$(</span><span class="nb">hostname</span><span class="si">)</span><span class="sh">&lt;/h1&gt;
    &lt;p&gt;Date: </span><span class="si">$(</span><span class="nb">date</span><span class="si">)</span><span class="sh">&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;
</span><span class="no">EOF
</span><span class="c"># Variables inside heredoc get expanded!</span>
<span class="c"># Use &lt;&lt; 'EOF' (with quotes) to PREVENT variable expansion</span>
</code></pre>

</div>



<h3>
  
  
  Redirection in Scripts
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># Redirect command output to log file</span>
apt <span class="nb">install </span>nginx <span class="nt">-y</span> <span class="o">&gt;</span> nginx.log 2&gt;&amp;1
<span class="c"># &gt; nginx.log    = redirect stdout to file</span>
<span class="c"># 2&gt;&amp;1           = redirect stderr (2) to same place as stdout (1)</span>
<span class="c"># Combined: ALL output (normal + errors) goes to nginx.log</span>

<span class="c"># Append to log</span>
<span class="nb">echo</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">date</span><span class="si">)</span><span class="s2"> - Installation complete"</span> <span class="o">&gt;&gt;</span> nginx.log
</code></pre>

</div>






<h3>
  
  
  Lab Script Examples
</h3>

<h4>
  
  
  Script 1: <code>backup.sh</code> — Automated Directory Backup
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">echo</span> <span class="s2">"Enter directory path to backup:"</span>
<span class="nb">read </span>DIR_PATH

<span class="k">if</span> <span class="o">[</span> <span class="o">!</span> <span class="nt">-d</span> <span class="s2">"</span><span class="nv">$DIR_PATH</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"ERROR: Directory </span><span class="nv">$DIR_PATH</span><span class="s2"> does not exist!"</span>
    <span class="nb">exit </span>1
<span class="k">fi

</span><span class="nv">TIMESTAMP</span><span class="o">=</span><span class="si">$(</span><span class="nb">date</span> +%Y%m%d_%H%M%S<span class="si">)</span>
<span class="nv">BACKUP_FILE</span><span class="o">=</span><span class="s2">"backup_</span><span class="k">${</span><span class="nv">TIMESTAMP</span><span class="k">}</span><span class="s2">.tar.gz"</span>

<span class="nb">tar</span> <span class="nt">-czf</span> <span class="s2">"</span><span class="nv">$BACKUP_FILE</span><span class="s2">"</span> <span class="s2">"</span><span class="nv">$DIR_PATH</span><span class="s2">"</span>

<span class="k">if</span> <span class="o">[</span> <span class="nv">$?</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"SUCCESS: Backup created: </span><span class="nv">$BACKUP_FILE</span><span class="s2">"</span>
    <span class="nb">echo</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">date</span><span class="si">)</span><span class="s2"> - SUCCESS - </span><span class="nv">$BACKUP_FILE</span><span class="s2">"</span> <span class="o">&gt;&gt;</span> backup.log
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"FAILED: Backup failed!"</span>
    <span class="nb">echo</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">date</span><span class="si">)</span><span class="s2"> - FAILED"</span> <span class="o">&gt;&gt;</span> backup.log
    <span class="nb">exit </span>1
<span class="k">fi</span>
</code></pre>

</div>



<h4>
  
  
  Script 2: Virtual Host Automator
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">echo</span> <span class="s2">"Enter domain/directory name:"</span>
<span class="nb">read </span>DOMAIN

<span class="nb">sudo mkdir</span> <span class="nt">-p</span> /var/www/<span class="nv">$DOMAIN</span>

<span class="nb">sudo cat</span> <span class="o">&lt;&lt;</span> <span class="no">EOF</span><span class="sh"> &gt; /var/www/</span><span class="nv">$DOMAIN</span><span class="sh">/index.html
&lt;html&gt;&lt;body&gt;&lt;h1&gt;Welcome to </span><span class="nv">$DOMAIN</span><span class="sh">&lt;/h1&gt;&lt;/body&gt;&lt;/html&gt;
</span><span class="no">EOF

</span><span class="nb">sudo cat</span> <span class="o">&lt;&lt;</span> <span class="no">EOF</span><span class="sh"> &gt; /etc/apache2/sites-available/</span><span class="nv">$DOMAIN</span><span class="sh">.conf
&lt;VirtualHost *:80&gt;
    ServerName </span><span class="nv">$DOMAIN</span><span class="sh">
    DocumentRoot /var/www/</span><span class="nv">$DOMAIN</span><span class="sh">
    ErrorLog /var/log/apache2/</span><span class="k">${</span><span class="nv">DOMAIN</span><span class="k">}</span><span class="sh">-error.log
    CustomLog /var/log/apache2/</span><span class="k">${</span><span class="nv">DOMAIN</span><span class="k">}</span><span class="sh">-access.log combined
&lt;/VirtualHost&gt;
</span><span class="no">EOF

</span><span class="nb">sudo chown</span> <span class="nt">-R</span> www-data:www-data /var/www/<span class="nv">$DOMAIN</span>
<span class="nb">sudo chmod</span> <span class="nt">-R</span> 755 /var/www/<span class="nv">$DOMAIN</span>
<span class="nb">sudo </span>a2ensite <span class="nv">$DOMAIN</span>.conf
<span class="nb">sudo </span>systemctl reload apache2
<span class="nb">echo</span> <span class="s2">"Website </span><span class="nv">$DOMAIN</span><span class="s2"> deployed successfully!"</span>
</code></pre>

</div>



<h4>
  
  
  Script 3: <code>wserver.sh</code> — Nginx Install/Remove with Flags
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="k">case</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="k">in</span>
    <span class="nt">-i</span><span class="p">)</span>
        <span class="k">if </span>dpkg <span class="nt">-l</span> | <span class="nb">grep</span> <span class="nt">-q</span> nginx<span class="p">;</span> <span class="k">then
            </span><span class="nb">echo</span> <span class="s2">"Nginx is already installed."</span>
            <span class="nb">exit </span>0
        <span class="k">fi
        </span><span class="nb">sudo </span>apt <span class="nb">install </span>nginx <span class="nt">-y</span> <span class="o">&gt;</span> nginx.log 2&gt;&amp;1
        <span class="k">if</span> <span class="o">[</span> <span class="nv">$?</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
            </span><span class="nb">echo</span> <span class="s2">"Nginx installed successfully."</span>
        <span class="k">else
            </span><span class="nb">echo</span> <span class="s2">"ERROR: Installation failed. Check nginx.log"</span>
            <span class="nb">exit </span>1
        <span class="k">fi</span>
        <span class="p">;;</span>
    <span class="nt">-r</span><span class="p">)</span>
        <span class="k">if</span> <span class="o">!</span> dpkg <span class="nt">-l</span> | <span class="nb">grep</span> <span class="nt">-q</span> nginx<span class="p">;</span> <span class="k">then
            </span><span class="nb">echo</span> <span class="s2">"Nginx is not installed."</span>
            <span class="nb">exit </span>0
        <span class="k">fi
        </span><span class="nb">sudo </span>apt remove nginx <span class="nt">-y</span> <span class="o">&gt;</span> nginx.log 2&gt;&amp;1
        <span class="k">if</span> <span class="o">[</span> <span class="nv">$?</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
            </span><span class="nb">echo</span> <span class="s2">"Nginx removed successfully."</span>
        <span class="k">else
            </span><span class="nb">echo</span> <span class="s2">"ERROR: Removal failed. Check nginx.log"</span>
            <span class="nb">exit </span>1
        <span class="k">fi</span>
        <span class="p">;;</span>
    <span class="k">*</span><span class="p">)</span>
        <span class="nb">echo</span> <span class="s2">"Usage: </span><span class="nv">$0</span><span class="s2"> [-i install] [-r remove]"</span>
        <span class="nb">exit </span>1
        <span class="p">;;</span>
<span class="k">esac</span>
</code></pre>

</div>



<h4>
  
  
  Script 4: <code>system_health.sh</code> — HTML Report
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nv">TOTAL_MEM</span><span class="o">=</span><span class="si">$(</span>free <span class="nt">-m</span> | <span class="nb">awk</span> <span class="s1">'/Mem:/ {print $2}'</span><span class="si">)</span>
<span class="nv">USED_MEM</span><span class="o">=</span><span class="si">$(</span>free <span class="nt">-m</span> | <span class="nb">awk</span> <span class="s1">'/Mem:/ {print $3}'</span><span class="si">)</span>
<span class="nv">AVAIL_MEM</span><span class="o">=</span><span class="si">$(</span>free <span class="nt">-m</span> | <span class="nb">awk</span> <span class="s1">'/Mem:/ {print $7}'</span><span class="si">)</span>
<span class="nv">TOTAL_SWAP</span><span class="o">=</span><span class="si">$(</span>free <span class="nt">-m</span> | <span class="nb">awk</span> <span class="s1">'/Swap:/ {print $2}'</span><span class="si">)</span>
<span class="nv">USED_SWAP</span><span class="o">=</span><span class="si">$(</span>free <span class="nt">-m</span> | <span class="nb">awk</span> <span class="s1">'/Swap:/ {print $3}'</span><span class="si">)</span>

<span class="nv">DISK_TOTAL</span><span class="o">=</span><span class="si">$(</span><span class="nb">df</span> <span class="nt">-h</span> <span class="nt">--total</span> <span class="nt">--exclude-type</span><span class="o">=</span>tmpfs | <span class="nb">awk</span> <span class="s1">'/total/ {print $2}'</span><span class="si">)</span>
<span class="nv">DISK_USED</span><span class="o">=</span><span class="si">$(</span><span class="nb">df</span> <span class="nt">-h</span> <span class="nt">--total</span> <span class="nt">--exclude-type</span><span class="o">=</span>tmpfs | <span class="nb">awk</span> <span class="s1">'/total/ {print $3}'</span><span class="si">)</span>
<span class="nv">DISK_AVAIL</span><span class="o">=</span><span class="si">$(</span><span class="nb">df</span> <span class="nt">-h</span> <span class="nt">--total</span> <span class="nt">--exclude-type</span><span class="o">=</span>tmpfs | <span class="nb">awk</span> <span class="s1">'/total/ {print $4}'</span><span class="si">)</span>

<span class="nb">cat</span> <span class="o">&lt;&lt;</span> <span class="no">EOF</span><span class="sh"> &gt; system_health.html
&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;&lt;title&gt;System Health Report&lt;/title&gt;&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;System Health Report&lt;/h1&gt;
&lt;p&gt;Date: </span><span class="si">$(</span><span class="nb">date</span><span class="si">)</span><span class="sh">&lt;/p&gt;
&lt;h2&gt;Memory Usage&lt;/h2&gt;
&lt;p&gt;Total: </span><span class="k">${</span><span class="nv">TOTAL_MEM</span><span class="k">}</span><span class="sh">MB | Used: </span><span class="k">${</span><span class="nv">USED_MEM</span><span class="k">}</span><span class="sh">MB | Available: </span><span class="k">${</span><span class="nv">AVAIL_MEM</span><span class="k">}</span><span class="sh">MB&lt;/p&gt;
&lt;h2&gt;Disk Usage&lt;/h2&gt;
&lt;p&gt;Total: </span><span class="nv">$DISK_TOTAL</span><span class="sh"> | Used: </span><span class="nv">$DISK_USED</span><span class="sh"> | Available: </span><span class="nv">$DISK_AVAIL</span><span class="sh">&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;
</span><span class="no">EOF

</span><span class="nb">sudo cp </span>system_health.html /var/www/html/index.html
<span class="k">if</span> <span class="o">[</span> <span class="nv">$?</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"SUCCESS: Report hosted at http://localhost"</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"ERROR: Failed to deploy report"</span>
    <span class="nb">exit </span>1
<span class="k">fi</span>
</code></pre>

</div>



<h4>
  
  
  Script 5: <code>systeminfo.sh</code>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">echo</span> <span class="s2">"Kernel Name: </span><span class="si">$(</span><span class="nb">uname</span> <span class="nt">-s</span><span class="si">)</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Kernel Release: </span><span class="si">$(</span><span class="nb">uname</span> <span class="nt">-r</span><span class="si">)</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Processor: </span><span class="si">$(</span><span class="nb">uname</span> <span class="nt">-p</span><span class="si">)</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"OS: </span><span class="si">$(</span><span class="nb">cat</span> /etc/os-release | <span class="nb">grep </span>PRETTY_NAME | <span class="nb">cut</span> <span class="nt">-d</span><span class="o">=</span> <span class="nt">-f2</span><span class="si">)</span><span class="s2">"</span>

<span class="nv">EDITOR</span><span class="o">=</span><span class="s2">"nano"</span>
<span class="nb">echo</span> <span class="s2">"Favorite Editor: </span><span class="nv">$EDITOR</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Editor Location: </span><span class="si">$(</span>which <span class="nv">$EDITOR</span><span class="si">)</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="s2">"Documentation: </span><span class="si">$(</span>whereis <span class="nv">$EDITOR</span><span class="si">)</span><span class="s2">"</span>
</code></pre>

</div>






<h2>
  
  
  10. Docker Containers
</h2>

<h3>
  
  
  What is Docker?
</h3>

<p>Docker is a platform that lets you package, ship, and run applications in isolated environments called <strong>containers</strong>. A container is like a lightweight virtual machine that shares the host OS kernel. Think of it as: your app + all its dependencies, packaged together so it runs the same everywhere.</p>

<h3>
  
  
  Key Concepts
</h3>

<ul>
<li>
<strong>Image</strong>: A blueprint/template for a container (like a class in OOP). Downloaded from Docker Hub.</li>
<li>
<strong>Container</strong>: A running instance of an image (like an object). You can run multiple from one image.</li>
<li>
<strong>Docker Hub</strong>: Online registry of Docker images (like GitHub for images).</li>
<li>
<strong>Tag</strong>: Version of an image. Example: <code>ubuntu:22.04</code> vs <code>ubuntu:latest</code>
</li>
</ul>

<h3>
  
  
  Verify Docker Installation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker <span class="nt">--version</span>           <span class="c"># Check Docker version</span>
docker version             <span class="c"># Detailed client + server version</span>
docker run hello-world     <span class="c"># Test Docker works</span>
</code></pre>

</div>



<h3>
  
  
  Docker Container Lifecycle
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># PULL an image from Docker Hub</span>
docker pull ubuntu                 <span class="c"># Download ubuntu image (latest)</span>
docker pull ubuntu:22.04           <span class="c"># Download specific version</span>
docker pull nginx                  <span class="c"># Download nginx image</span>

<span class="c"># RUN a container from an image</span>
docker run ubuntu                  <span class="c"># Runs and exits immediately (no process)</span>
docker run ubuntu <span class="nb">sleep </span>5          <span class="c"># Runs, sleeps 5 seconds, then exits</span>

<span class="c"># RUN in interactive mode (get a shell inside container)</span>
docker run <span class="nt">-it</span> ubuntu bash         <span class="c"># -i=interactive, -t=terminal</span>
<span class="c"># You are now INSIDE the container!</span>
<span class="nb">echo</span> <span class="s2">"Hello Docker"</span>
<span class="nb">ls
exit</span>                               <span class="c"># Exit container (container stops)</span>

<span class="c"># RUN with a custom name</span>
docker run <span class="nt">--name</span> myubuntu ubuntu <span class="nb">sleep </span>30

<span class="c"># RUN in detached mode (background)</span>
docker run <span class="nt">-d</span> ubuntu <span class="nb">sleep </span>300     <span class="c"># -d = detached (runs in background)</span>
docker run <span class="nt">-dit</span> <span class="nt">--name</span> <span class="nb">test </span>ubuntu bash  <span class="c"># Detached + interactive</span>
</code></pre>

</div>



<h3>
  
  
  Managing Containers
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># LIST containers</span>
docker ps                          <span class="c"># Show RUNNING containers only</span>
docker ps <span class="nt">-a</span>                       <span class="c"># Show ALL containers (running + stopped)</span>

<span class="c"># STOP a container</span>
docker stop &lt;container_id&gt;         <span class="c"># Graceful stop</span>
docker stop myubuntu               <span class="c"># Stop by name</span>

<span class="c"># REMOVE a container</span>
docker <span class="nb">rm</span> &lt;container_id&gt;           <span class="c"># Remove stopped container</span>
docker <span class="nb">rm </span>myubuntu                 <span class="c"># Remove by name</span>

<span class="c"># STOP ALL containers at once</span>
docker stop <span class="si">$(</span>docker ps <span class="nt">-q</span><span class="si">)</span>        <span class="c"># -q = quiet (only IDs)</span>

<span class="c"># REMOVE ALL containers</span>
docker <span class="nb">rm</span> <span class="si">$(</span>docker ps <span class="nt">-aq</span><span class="si">)</span>         <span class="c"># Remove all (running + stopped)</span>
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> <code>docker run ubuntu</code> exits immediately because there is no long-running process. Use <code>sleep</code>, <code>-it bash</code>, or <code>-d</code> to keep it alive.</p>
</blockquote>

<h3>
  
  
  Managing Images
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># LIST downloaded images</span>
docker images

<span class="c"># REMOVE an image</span>
docker rmi ubuntu                  <span class="c"># Remove ubuntu image</span>
docker rmi &lt;image_id&gt;              <span class="c"># Remove by ID</span>
<span class="c"># Note: Must remove all containers using that image first!</span>
</code></pre>

</div>



<h3>
  
  
  Attach vs Exec (IMPORTANT DIFFERENCE!)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># ATTACH to a running container (connects to main process)</span>
docker attach <span class="nb">test</span>
<span class="c"># CTRL+P then CTRL+Q = detach WITHOUT stopping container</span>
<span class="c"># If you type 'exit', container STOPS</span>

<span class="c"># EXEC runs a NEW process inside a running container</span>
docker <span class="nb">exec</span> <span class="nt">-it</span> <span class="nb">test </span>bash
<span class="c"># If you type 'exit', only the exec process ends</span>
<span class="c"># The container keeps running!</span>
</code></pre>

</div>



<blockquote>
<p><strong>📝 NOTE:</strong> Use <code>exec</code> when you want to inspect a running container without affecting it. Use <code>attach</code> to reconnect to the main process.</p>
</blockquote>

<h3>
  
  
  Container Logs &amp; Inspect
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker logs &lt;container_id&gt;                          <span class="c"># View container logs</span>
docker logs myubuntu

docker inspect <span class="nb">test</span>                                  <span class="c"># Full container details (JSON)</span>
docker inspect <span class="nt">--format</span><span class="o">=</span><span class="s1">'{{.State.Status}}'</span> <span class="nb">test</span>     <span class="c"># Get specific info</span>
</code></pre>

</div>



<h3>
  
  
  Port Mapping (<code>-p</code>)
</h3>

<p>Port mapping connects a port on your HOST machine to a port inside the container.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Map host port 8081 to container port 80</span>
docker run <span class="nt">-d</span> <span class="nt">-p</span> 8081:80 nginx
<span class="c">#                ^     ^</span>
<span class="c">#                |     +-- container port (nginx listens on 80)</span>
<span class="c">#                +-------- host port (access via localhost:8081)</span>

<span class="c"># Run multiple web servers on different ports</span>
docker run <span class="nt">-d</span> <span class="nt">--name</span> web1 <span class="nt">-p</span> 8081:80 ubuntu/apache2
docker run <span class="nt">-d</span> <span class="nt">--name</span> web2 <span class="nt">-p</span> 8082:80 ubuntu/apache2

<span class="c"># Access in browser:</span>
<span class="c"># http://localhost:8081  --&gt;  web1</span>
<span class="c"># http://localhost:8082  --&gt;  web2</span>
</code></pre>

</div>



<blockquote>
<p><strong>🔥 EXAM TIP:</strong> Format is <code>-p HOST_PORT:CONTAINER_PORT</code>. The HOST port is what you type in the browser. The CONTAINER port is what the app listens on inside.</p>
</blockquote>

<h3>
  
  
  Volume Mounts (<code>-v</code>) / Bind Mounts
</h3>

<p>Volumes let you share files between your host machine and the container.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Mount current directory into container</span>
docker run <span class="nt">-it</span> <span class="nt">-v</span> <span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>:/scripts ubuntu bash
<span class="c"># $(pwd) = your current directory on host</span>
<span class="c"># /scripts = path inside the container</span>

<span class="c"># Mount a specific file into nginx</span>
<span class="nb">echo</span> <span class="s1">'&lt;h1&gt;Hello Docker!&lt;/h1&gt;'</span> <span class="o">&gt;</span> index.html
docker run <span class="nt">-d</span> <span class="nt">-p</span> 8081:80 <span class="nt">-v</span> <span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/index.html:/usr/share/nginx/html/index.html nginx
<span class="c"># You have just hosted a simple web app!</span>

<span class="c"># Mount a Python script</span>
<span class="nb">echo</span> <span class="s1">'print("Hello, world!")'</span> <span class="o">&gt;</span> hello.py
docker run <span class="nt">-it</span> <span class="nt">--rm</span> <span class="nt">-v</span> <span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/hello.py:/hello.py python:3.10-slim bash
<span class="c"># Then inside: python hello.py</span>
</code></pre>

</div>



<blockquote>
<p><strong>📝 NOTE:</strong> The <code>--rm</code> flag automatically removes the container when it exits. Great for temporary containers.</p>
</blockquote>

<h3>
  
  
  Complete Docker Command Reference
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>docker version</code></td>
<td>Check Docker client and server version</td>
</tr>
<tr>
<td><code>docker run &lt;image&gt;</code></td>
<td>Start a container from an image</td>
</tr>
<tr>
<td><code>docker run --name &lt;n&gt; &lt;img&gt;</code></td>
<td>Run with a custom name</td>
</tr>
<tr>
<td><code>docker run &lt;img&gt;:&lt;tag&gt;</code></td>
<td>Run a specific version</td>
</tr>
<tr>
<td><code>docker run -it &lt;img&gt; bash</code></td>
<td>Run interactively with terminal</td>
</tr>
<tr>
<td><code>docker run -d &lt;img&gt;</code></td>
<td>Run in detached (background) mode</td>
</tr>
<tr>
<td><code>docker run -p H:C &lt;img&gt;</code></td>
<td>Map host port H to container port C</td>
</tr>
<tr>
<td><code>docker run -v H:C &lt;img&gt;</code></td>
<td>Mount host path H to container path C</td>
</tr>
<tr>
<td><code>docker run --rm &lt;img&gt;</code></td>
<td>Auto-remove container on exit</td>
</tr>
<tr>
<td><code>docker ps</code></td>
<td>List running containers</td>
</tr>
<tr>
<td><code>docker ps -a</code></td>
<td>List ALL containers</td>
</tr>
<tr>
<td><code>docker stop &lt;id/name&gt;</code></td>
<td>Stop a running container</td>
</tr>
<tr>
<td><code>docker rm &lt;id/name&gt;</code></td>
<td>Remove a stopped container</td>
</tr>
<tr>
<td><code>docker images</code></td>
<td>List downloaded images</td>
</tr>
<tr>
<td><code>docker rmi &lt;image&gt;</code></td>
<td>Remove an image</td>
</tr>
<tr>
<td><code>docker pull &lt;image&gt;</code></td>
<td>Download image from Docker Hub</td>
</tr>
<tr>
<td><code>docker exec -it &lt;id&gt; bash</code></td>
<td>Open shell in running container</td>
</tr>
<tr>
<td><code>docker attach &lt;id&gt;</code></td>
<td>Attach to main process of container</td>
</tr>
<tr>
<td><code>docker logs &lt;id&gt;</code></td>
<td>View container output logs</td>
</tr>
<tr>
<td><code>docker inspect &lt;id&gt;</code></td>
<td>Detailed container info (JSON)</td>
</tr>
<tr>
<td><code>docker stop $(docker ps -q)</code></td>
<td>Stop ALL running containers</td>
</tr>
<tr>
<td><code>docker rm $(docker ps -aq)</code></td>
<td>Remove ALL containers</td>
</tr>
</tbody>
</table></div>

<p>— list, create, add to sudo, switch user, create group</p>

<h2>
  
  
  12. Quick Reference Cheat Sheet
</h2>

<h3>
  
  
  systemctl Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>systemctl start &lt;service&gt;     <span class="c"># Start a service NOW</span>
systemctl stop &lt;service&gt;      <span class="c"># Stop a service NOW</span>
systemctl restart &lt;service&gt;   <span class="c"># Restart a service</span>
systemctl reload &lt;service&gt;    <span class="c"># Reload config without stopping</span>
systemctl status &lt;service&gt;    <span class="c"># Check if service is running</span>
systemctl <span class="nb">enable</span> &lt;service&gt;    <span class="c"># Start on boot</span>
systemctl disable &lt;service&gt;   <span class="c"># Don't start on boot</span>
</code></pre>

</div>



<h3>
  
  
  Key File Paths
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Path</th>
<th>What</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/etc/ssh/sshd_config</code></td>
<td>SSH server configuration</td>
</tr>
<tr>
<td><code>~/.ssh/id_rsa</code></td>
<td>Your SSH private key</td>
</tr>
<tr>
<td><code>~/.ssh/id_rsa.pub</code></td>
<td>Your SSH public key</td>
</tr>
<tr>
<td><code>~/.ssh/authorized_keys</code></td>
<td>Public keys allowed to login</td>
</tr>
<tr>
<td><code>~/.ssh/config</code></td>
<td>Client SSH shortcuts</td>
</tr>
<tr>
<td><code>/var/www/html/</code></td>
<td>Default web root (Apache + Nginx)</td>
</tr>
<tr>
<td><code>/etc/apache2/sites-available/</code></td>
<td>Apache virtual host configs</td>
</tr>
<tr>
<td><code>/etc/apache2/sites-enabled/</code></td>
<td>Active Apache sites (symlinks)</td>
</tr>
<tr>
<td><code>/etc/nginx/sites-available/</code></td>
<td>Nginx server block configs</td>
</tr>
<tr>
<td><code>/etc/nginx/sites-enabled/</code></td>
<td>Active Nginx sites (symlinks)</td>
</tr>
<tr>
<td><code>/var/log/apache2/error.log</code></td>
<td>Apache error log</td>
</tr>
<tr>
<td><code>/var/log/nginx/error.log</code></td>
<td>Nginx error log</td>
</tr>
<tr>
<td><code>/etc/passwd</code></td>
<td>User account information</td>
</tr>
<tr>
<td><code>/etc/group</code></td>
<td>Group definitions</td>
</tr>
<tr>
<td><code>/etc/shadow</code></td>
<td>Encrypted passwords (root only)</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Permission Quick Reference
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>644 <span class="o">=</span> rw-r--r--   <span class="c"># Standard file permission</span>
755 <span class="o">=</span> rwxr-xr-x   <span class="c"># Standard directory / script permission</span>
700 <span class="o">=</span> rwx------   <span class="c"># Private (owner only)</span>
600 <span class="o">=</span> rw-------   <span class="c"># SSH keys</span>
777 <span class="o">=</span> rwxrwxrwx   <span class="c"># NEVER USE (security risk)</span>
</code></pre>

</div>



<h3>
  
  
  Docker Quick Reference
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">-it</span> ubuntu bash        <span class="c"># Interactive shell</span>
docker run <span class="nt">-d</span> <span class="nt">-p</span> 8080:80 nginx    <span class="c"># Background + port map</span>
docker run <span class="nt">-v</span> <span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>:/app ubuntu  <span class="c"># Volume mount</span>
docker ps <span class="nt">-a</span>                      <span class="c"># List all containers</span>
docker stop <span class="si">$(</span>docker ps <span class="nt">-q</span><span class="si">)</span>       <span class="c"># Stop all running</span>
docker <span class="nb">rm</span> <span class="si">$(</span>docker ps <span class="nt">-aq</span><span class="si">)</span>        <span class="c"># Remove all containers</span>
docker <span class="nb">exec</span> <span class="nt">-it</span> &lt;<span class="nb">id</span><span class="o">&gt;</span> bash         <span class="c"># Shell into running container</span>
CTRL+P <span class="k">then </span>CTRL+Q               <span class="c"># Detach without stopping</span>
</code></pre>

</div>



<h3>
  
  
  Bash Scripting Quick Reference
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash                        # Shebang (always first line)</span>
<span class="nv">$1</span> <span class="nv">$2</span> <span class="nv">$# $@</span> <span class="nv">$?</span>                    <span class="c"># Args, count, all, exit code</span>
<span class="nb">read </span>VAR                           <span class="c"># Read user input</span>
<span class="k">if</span> <span class="o">[</span> condition <span class="o">]</span><span class="p">;</span> <span class="k">then</span> ... <span class="k">fi</span>      <span class="c"># Conditional</span>
<span class="k">case</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="k">in </span>pattern<span class="p">)</span> ... <span class="p">;;</span> <span class="k">esac</span>  <span class="c"># Flag handling</span>
<span class="k">for </span>item <span class="k">in </span>list<span class="p">;</span> <span class="k">do</span> ... <span class="k">done</span>      <span class="c"># Loop</span>
<span class="nb">tar</span> <span class="nt">-czf</span> name.tar.gz <span class="nb">dir</span>/         <span class="c"># Create archive</span>
<span class="nb">awk</span> <span class="s1">'{print $1}'</span> file             <span class="c"># Text processing</span>
<span class="nb">cat</span> <span class="o">&lt;&lt;</span> <span class="no">EOF</span><span class="sh"> &gt; file ... EOF          # Heredoc
command &gt; file 2&gt;&amp;1               # Redirect all output
</span></code></pre>

</div>






<blockquote>
<p><strong>Good luck on your exam! Practice these commands in the terminal — reading is not enough, you need muscle memory!</strong> 🚀</p>
</blockquote>

