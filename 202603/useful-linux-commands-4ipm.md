---
Title: Useful Linux Commands
Description: 
Author: Vincent Caunegre
Date: 2026-03-03T21:30:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>In this article I will show you the commands that I use the most on my Ubuntu vps.</p>

<h2>
  
  
  Navigation
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Action</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ls -alth</code></td>
<td>show content of folder</td>
</tr>
<tr>
<td><code>wc -l &lt;path to file&gt;</code></td>
<td>display the number of lines in a file</td>
</tr>
<tr>
<td><code>pushd &lt;path&gt;</code></td>
<td>go to a directory and save the path in history</td>
</tr>
<tr>
<td><code>popd</code></td>
<td>rollback to previous directory from pushd</td>
</tr>
<tr>
<td><code>dirs -v</code></td>
<td>show pushd history</td>
</tr>
<tr>
<td><code>cd -</code></td>
<td>get back to previous directory (reset history)</td>
</tr>
<tr>
<td><code>mkdir -p /folder/folder2</code></td>
<td>create a new directory with non existing folders</td>
</tr>
<tr>
<td><code>find . -name "*.yaml"</code></td>
<td>find all files that end by ".yaml" in current directory</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  File editing
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Action</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>pwd &gt; test.txt</code></td>
<td>send the content of a command to a file (erase the previous content)</td>
</tr>
<tr>
<td><code>pwd &gt;&gt; test.txt</code></td>
<td>add the content of a command to the next line of a file</td>
</tr>
<tr>
<td>pwd | tee test.txt</td>
<td>same as pwd &gt; test.txt but print it in cli</td>
</tr>
<tr>
<td>pwd | tee -a test.txt</td>
<td>same as pwd &gt;&gt; test.txt but print it in cli</td>
</tr>
<tr>
<td>grep "ubuntu" test.txt | wc -l</td>
<td>connect commands (for instance to get number of line with a word)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  System
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Action</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ps aux</code></td>
<td>list all processes for all users</td>
</tr>
<tr>
<td>ps aux | grep </td>
<td>Find a running process by name</td>
</tr>
<tr>
<td>
<code>top</code> and <code>htop</code>
</td>
<td>List current running process by cpu usage</td>
</tr>
<tr>
<td><code>free -m</code></td>
<td>get memory usage</td>
</tr>
<tr>
<td><code>df -h</code></td>
<td>get volumes % still available, always check that biggest volume is not &gt; 90%</td>
</tr>
<tr>
<td>sudo du -sh * | sort -h</td>
<td>sort biggest folders</td>
</tr>
<tr>
<td>sudo du -ahx / | sort -rh</td>
<td>head -n 20</td>
</tr>
<tr>
<td>sudo ss -tulpn | cat | grep :8080 =&gt;  kill -15 </td>
<td>find if a port is already used by a process, then kill it</td>
</tr>
<tr>
<td><code>dig google.com</code></td>
<td>check if a domain name is valid</td>
</tr>
<tr>
<td><code>curl -v &lt;url&gt;</code></td>
<td>execute an http request</td>
</tr>
<tr>
<td><code>ip a</code></td>
<td>see private and public ip adress</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  systemctl
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Action</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>sudo systemctl start my-api.service</code></td>
<td>start systemctl</td>
</tr>
<tr>
<td><code>journalctl -u my-api.service -f</code></td>
<td>show logs of a systemctl service in real time</td>
</tr>
<tr>
<td><code>journalctl -u my-api.service -p err</code></td>
<td>show only errors for a systemctl service</td>
</tr>
<tr>
<td><code>sudo journalctl --vacuum-time=2d</code></td>
<td>remove logs older than 2 days</td>
</tr>
</tbody>
</table></div>

