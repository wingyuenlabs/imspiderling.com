---
Title: Building an Automated Docker Deployment Script: A Complete Beginner's Guide
Description: 
Author: PrimoCrypt
Date: 2025-12-09T21:53:18.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Have you ever wondered how professional developers deploy their applications to servers automatically? In this comprehensive guide, I'll walk you through creating a powerful Bash script that automates the entire deployment process for Docker-based applications. By the end of this article, you'll have a script that can:</p>

<p>✅ Clone your Git repository<br><br>
✅ Set up a remote server environment<br><br>
✅ Build and deploy Docker containers<br><br>
✅ Configure Nginx as a reverse proxy<br><br>
✅ Validate your deployment<br><br>
✅ Handle cleanup and log management</p>

<p><strong>Best of all</strong>: You'll understand every step, even if you're just starting your DevOps journey!</p>


<h2>
  
  
  What is Automated Deployment and Why Do We Need It?
</h2>

<p>In the early days of web development, deploying an application meant manually copying files to a server, installing dependencies, and configuring everything by hand. This process was:</p>

<ul>
<li>
<strong>Time-consuming</strong>: Could take hours for a single deployment</li>
<li>
<strong>Error-prone</strong>: Easy to forget a step or misconfigure something</li>
<li>
<strong>Not repeatable</strong>: Hard to deploy the same way twice</li>
<li>
<strong>Not scalable</strong>: Imagine doing this for 10 different applications!</li>
</ul>

<p><strong>Automated deployment solves all these problems</strong> by using scripts to perform all deployment steps consistently and reliably.</p>


<h2>
  
  
  Understanding the Deployment Workflow
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3vefmxj2fd8l3xf6iv36.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3vefmxj2fd8l3xf6iv36.png" alt="Deployment Workflow" width="800" height="800"></a></p>

<p>Before we dive into the code, let's understand the big picture of what happens during deployment:</p>

<ol>
<li>
<strong>Your local machine</strong> runs the deployment script</li>
<li>
<strong>The script clones</strong> your application code from Git</li>
<li>
<strong>It connects</strong> to your remote server via SSH</li>
<li>
<strong>It sets up</strong> Docker and Nginx on the server</li>
<li>
<strong>It builds</strong> your Docker container and runs it</li>
<li>
<strong>It configures</strong> Nginx to route traffic to your app</li>
<li>
<strong>It validates</strong> everything is working correctly</li>
</ol>


<h2>
  
  
  Prerequisites
</h2>

<p>Before you start, make sure you have:</p>

<ul>
<li>
<strong>Basic understanding of</strong>:

<ul>
<li>Linux command line basics</li>
<li>Git version control</li>
<li>What Docker is (don't worry, we'll explain as we go!)</li>
</ul>
</li>
<li>
<strong>Technical requirements</strong>:

<ul>
<li>A Linux/Mac machine (or WSL on Windows)</li>
<li>A remote server (AWS EC2, DigitalOcean, etc.)</li>
<li>SSH key access to your server</li>
<li>A Git repository with your application code</li>
</ul>
</li>
</ul>

<blockquote>
<p>[!TIP]<br>
If you're new to SSH keys, think of them as a secure password alternative that uses cryptographic keys instead of text passwords.</p>
</blockquote>


<h2>
  
  
  The 7 Stages of Our Deployment Script
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh9pc15nskzhjvyt9g3ey.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh9pc15nskzhjvyt9g3ey.png" alt="Deployment Stages" width="800" height="800"></a></p>

<p>Our deployment script is organized into 7 distinct stages. Let's explore each one in detail!</p>


<h3>
  
  
  Stage 0.5: Setup and Housekeeping
</h3>

<p><strong>What it does</strong>: Sets up logging, error handling, and cleanup functionality.</p>

<p><strong>Key concepts</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/usr/bin/env bash</span>

<span class="nb">set</span> <span class="nt">-o</span> errexit   <span class="c"># Exit on any error</span>
<span class="nb">set</span> <span class="nt">-o</span> nounset   <span class="c"># Exit on undefined variable</span>
<span class="nb">set</span> <span class="nt">-o</span> pipefail  <span class="c"># Exit if any command in a pipe fails</span>
</code></pre>

</div>



<p>These three lines are <strong>crucial safety features</strong>:</p>

<ul>
<li>
<code>errexit</code>: If any command fails, the script stops immediately</li>
<li>
<code>nounset</code>: Catches typos in variable names</li>
<li>
<code>pipefail</code>: Ensures errors in complex commands don't get hidden</li>
</ul>

<p><strong>Logging system</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">TIMESTAMP</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span><span class="nb">date</span> +%Y%m%d_%H%M%S<span class="si">)</span><span class="s2">"</span>
<span class="nv">LOGDIR</span><span class="o">=</span><span class="s2">"./logs"</span>
<span class="nv">LOGFILE</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">LOGDIR</span><span class="k">}</span><span class="s2">/deploy_</span><span class="k">${</span><span class="nv">TIMESTAMP</span><span class="k">}</span><span class="s2">.log"</span>

log<span class="o">()</span> <span class="o">{</span>
  <span class="nb">printf</span> <span class="s2">"%s [%s] %s</span><span class="se">\n</span><span class="s2">"</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">date</span> <span class="s1">'+%Y-%m-%d %H:%M:%S'</span><span class="si">)</span><span class="s2">"</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="s2">"</span><span class="nv">$2</span><span class="s2">"</span> | <span class="nb">tee</span> <span class="nt">-a</span> <span class="s2">"</span><span class="k">${</span><span class="nv">LOGFILE</span><span class="k">}</span><span class="s2">"</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This creates a timestamped log file for every deployment, so you can troubleshoot issues later.</p>

<p><strong>Cleanup mode</strong>:</p>

<p>The script includes a special <code>--cleanup</code> flag that removes all deployed containers, images, and configurations. This is useful for starting fresh or removing old deployments.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./deploy.sh <span class="nt">--cleanup</span>
</code></pre>

</div>



<blockquote>
<p>[!CAUTION]<br>
Cleanup mode will remove ALL Docker containers and images on your server. Always backup important data first!</p>
</blockquote>




<h3>
  
  
  Stage 1: Collect Parameters and Basic Validation
</h3>

<p><strong>What it does</strong>: Gathers all the information needed for deployment through interactive prompts.</p>

<p><strong>What you'll be asked</strong>:</p>

<ol>
<li>
<strong>Git Repository URL</strong>: Where is your code?

<ul>
<li>Example: <code>https://github.com/yourusername/your-app.git</code>
</li>
</ul>
</li>
<li>
<strong>Personal Access Token (PAT)</strong>: For private repositories

<ul>
<li>Entered securely (won't show on screen)</li>
</ul>
</li>
<li>
<strong>Branch Name</strong>: Which branch to deploy?

<ul>
<li>Default: <code>main</code>
</li>
</ul>
</li>
<li>
<strong>SSH Details</strong>: How to connect to your server

<ul>
<li>Username: <code>ubuntu</code>
</li>
<li>Host: <code>54.123.45.67</code> (your server IP)</li>
<li>SSH Key Path: <code>~/.ssh/your-key.pem</code>
</li>
</ul>
</li>
<li>
<strong>Application Port</strong>: What port does your app use?

<ul>
<li>Example: <code>3000</code> for Node.js, <code>8080</code> for many others</li>
</ul>
</li>
</ol>

<p><strong>Example interaction</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Git repository URL: https://github.com/john/awesome-app.git
Branch name: main
Remote SSH username: ubuntu
Remote SSH host/IP: 54.123.45.67
Path to local SSH private key: ~/.ssh/deploy-key.pem
Application internal container port: 3000
</code></pre>

</div>



<p><strong>Why validation matters</strong>:</p>

<p>The script validates your inputs immediately:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">case</span> <span class="s2">"</span><span class="k">${</span><span class="nv">APP_PORT</span><span class="k">}</span><span class="s2">"</span> <span class="k">in</span>
  <span class="s1">''</span><span class="p">|</span><span class="k">*</span><span class="o">[!</span>0-9]<span class="k">*</span> <span class="p">)</span> die 12 <span class="s2">"Invalid port"</span><span class="p">;;</span>
<span class="k">esac</span>
</code></pre>

</div>



<p>This ensures the port is actually a number, preventing errors later.</p>




<h3>
  
  
  Stage 2: Repository Clone and Validation
</h3>

<p><strong>What it does</strong>: Downloads your code and verifies connectivity to the server.</p>

<p><strong>The cloning process</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">WORKDIR</span><span class="o">=</span><span class="s2">"./workspace_</span><span class="k">${</span><span class="nv">TIMESTAMP</span><span class="k">}</span><span class="s2">"</span>
<span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">"</span>

git clone <span class="nt">-b</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BRANCH</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">GIT_REPO_URL</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">/repo"</span>
</code></pre>

</div>



<p>This creates a timestamped workspace directory and clones your specific branch into it.</p>

<p><strong>Handling authentication</strong>:</p>

<p>For private repositories with HTTPS:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Inject PAT into the URL securely</span>
<span class="nv">AUTH_URL</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="k">${</span><span class="nv">GIT_REPO_URL</span><span class="k">}</span><span class="s2">"</span> | <span class="nb">sed</span> <span class="nt">-E</span> <span class="s2">"s#https://#https://</span><span class="k">${</span><span class="nv">GIT_PAT</span><span class="k">}</span><span class="s2">@#"</span><span class="si">)</span><span class="s2">"</span>
git clone <span class="nt">-b</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BRANCH</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">AUTH_URL</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">/repo"</span>
</code></pre>

</div>



<p>For SSH repositories:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">GIT_SSH_COMMAND</span><span class="o">=</span><span class="s2">"ssh -i </span><span class="k">${</span><span class="nv">SSH_KEY_PATH</span><span class="k">}</span><span class="s2"> -o StrictHostKeyChecking=no"</span> <span class="se">\</span>
  git clone <span class="nt">-b</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BRANCH</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">GIT_REPO_URL</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">/repo"</span>
</code></pre>

</div>



<p><strong>Docker setup detection</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">if</span> <span class="o">[</span> <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">/repo/Dockerfile"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span>log <span class="s2">"INFO"</span> <span class="s2">"Found Dockerfile."</span>
<span class="k">elif</span> <span class="o">[</span> <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">/repo/docker-compose.yml"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span>log <span class="s2">"INFO"</span> <span class="s2">"Found docker-compose.yml."</span>
<span class="k">else
  </span>log <span class="s2">"WARN"</span> <span class="s2">"No Docker setup found. Skipping container checks."</span>
<span class="k">fi</span>
</code></pre>

</div>



<p>The script intelligently detects whether you're using a simple <code>Dockerfile</code> or a complex <code>docker-compose.yml</code> setup.</p>

<p><strong>SSH connectivity test</strong>:</p>

<p>Before transferring files, the script tests the SSH connection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ssh <span class="nt">-i</span> <span class="s2">"</span><span class="k">${</span><span class="nv">SSH_KEY_PATH</span><span class="k">}</span><span class="s2">"</span> <span class="nt">-o</span> <span class="nv">BatchMode</span><span class="o">=</span><span class="nb">yes</span> <span class="nt">-o</span> <span class="nv">ConnectTimeout</span><span class="o">=</span>10 <span class="se">\</span>
  <span class="s2">"</span><span class="k">${</span><span class="nv">REMOTE_USER</span><span class="k">}</span><span class="s2">@</span><span class="k">${</span><span class="nv">REMOTE_HOST</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"echo 'SSH_OK'"</span> <span class="o">&gt;</span> /dev/null 2&gt;&amp;1
</code></pre>

</div>



<p>If this fails, you'll know immediately that there's an SSH problem, saving you time debugging later.</p>

<p><strong>Dry-run file transfer</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>rsync <span class="nt">-avz</span> <span class="nt">--dry-run</span> <span class="nt">-e</span> <span class="s2">"ssh -i </span><span class="k">${</span><span class="nv">SSH_KEY_PATH</span><span class="k">}</span><span class="s2">"</span> <span class="se">\</span>
  <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">/repo/"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">REMOTE_USER</span><span class="k">}</span><span class="s2">@</span><span class="k">${</span><span class="nv">REMOTE_HOST</span><span class="k">}</span><span class="s2">:/tmp/deploy-test/"</span>
</code></pre>

</div>



<p>This simulates the file transfer without actually copying anything, ensuring the process will work when we do it for real.</p>

<blockquote>
<p>[!NOTE] &gt; <code>rsync</code> is better than <code>scp</code> for deployments because it only transfers changed files, making updates much faster.</p>
</blockquote>




<h3>
  
  
  Stage 3: Prepare Remote Environment
</h3>

<p><strong>What it does</strong>: Installs and configures all necessary software on your server.</p>

<p>This is where the magic happens! The script automatically sets up:</p>

<ol>
<li>
<strong>Docker</strong>: The containerization platform</li>
<li>
<strong>Docker Compose</strong>: For managing multi-container applications</li>
<li>
<strong>Nginx</strong>: The web server and reverse proxy</li>
</ol>

<p><strong>The installation process</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Update package lists</span>
<span class="nb">sudo </span>apt-get update <span class="nt">-y</span>

<span class="c"># Install Docker if not present</span>
<span class="k">if</span> <span class="o">!</span> <span class="nb">command</span> <span class="nt">-v</span> docker &amp;&gt;/dev/null<span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"Installing Docker..."</span>
  <span class="c"># Add Docker's official GPG key</span>
  <span class="nb">sudo mkdir</span> <span class="nt">-p</span> /etc/apt/keyrings
  curl <span class="nt">-fsSL</span> https://download.docker.com/linux/ubuntu/gpg | <span class="se">\</span>
    <span class="nb">sudo </span>gpg <span class="nt">--dearmor</span> <span class="nt">-o</span> /etc/apt/keyrings/docker.gpg

  <span class="c"># Set up Docker repository</span>
  <span class="nb">echo</span> <span class="s2">"deb [arch=</span><span class="si">$(</span>dpkg <span class="nt">--print-architecture</span><span class="si">)</span><span class="s2"> signed-by=/etc/apt/keyrings/docker.gpg] </span><span class="se">\</span><span class="s2">
    https://download.docker.com/linux/ubuntu </span><span class="si">$(</span>lsb_release <span class="nt">-cs</span><span class="si">)</span><span class="s2"> stable"</span> | <span class="se">\</span>
    <span class="nb">sudo tee</span> /etc/apt/sources.list.d/docker.list <span class="o">&gt;</span> /dev/null

  <span class="c"># Install Docker</span>
  <span class="nb">sudo </span>apt-get update <span class="nt">-y</span>
  <span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> docker-ce docker-ce-cli containerd.io
<span class="k">fi</span>
</code></pre>

</div>



<p><strong>User permissions</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Add current user to Docker group</span>
<span class="k">if</span> <span class="o">!</span> <span class="nb">groups</span> <span class="nv">$USER</span> | <span class="nb">grep</span> <span class="nt">-q</span> docker<span class="p">;</span> <span class="k">then
  </span><span class="nb">sudo </span>usermod <span class="nt">-aG</span> docker <span class="nv">$USER</span>
<span class="k">fi</span>
</code></pre>

</div>



<p>This allows you to run Docker commands without <code>sudo</code>, which is a security best practice.</p>

<p><strong>Service management</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Enable and start services</span>
<span class="nb">sudo </span>systemctl <span class="nb">enable </span>docker
<span class="nb">sudo </span>systemctl start docker
<span class="nb">sudo </span>systemctl <span class="nb">enable </span>nginx
<span class="nb">sudo </span>systemctl start nginx
</code></pre>

</div>



<p><code>systemctl enable</code> ensures services start automatically when the server reboots.</p>

<p><strong>Verification</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo</span> <span class="s2">"--- Installed Versions ---"</span>
docker <span class="nt">--version</span>
docker-compose <span class="nt">--version</span>
nginx <span class="nt">-v</span>
</code></pre>

</div>



<p>This confirms everything installed correctly and logs the versions for troubleshooting.</p>

<blockquote>
<p>[!IMPORTANT]<br>
This stage is <strong>idempotent</strong>, meaning you can run it multiple times safely. If Docker is already installed, it won't reinstall it.</p>
</blockquote>




<h3>
  
  
  Stage 4: Deploy Application on Remote
</h3>

<p><strong>What it does</strong>: Transfers your code to the server, builds Docker images, and runs containers.</p>

<p><strong>File synchronization</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">APP_NAME</span><span class="o">=</span><span class="si">$(</span><span class="nb">basename</span> <span class="s2">"</span><span class="k">${</span><span class="nv">GIT_REPO_URL</span><span class="k">}</span><span class="s2">"</span> .git<span class="si">)</span>
<span class="nv">REMOTE_APP_PATH</span><span class="o">=</span><span class="s2">"/opt/</span><span class="k">${</span><span class="nv">APP_NAME</span><span class="k">}</span><span class="s2">"</span>

<span class="c"># Prepare directory with correct permissions</span>
ssh <span class="nt">-i</span> <span class="s2">"</span><span class="k">${</span><span class="nv">SSH_KEY_PATH</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">REMOTE_USER</span><span class="k">}</span><span class="s2">@</span><span class="k">${</span><span class="nv">REMOTE_HOST</span><span class="k">}</span><span class="s2">"</span> <span class="se">\</span>
  <span class="s2">"sudo mkdir -p '</span><span class="k">${</span><span class="nv">REMOTE_APP_PATH</span><span class="k">}</span><span class="s2">' &amp;&amp; </span><span class="se">\</span><span class="s2">
   sudo chown -R </span><span class="se">\$</span><span class="s2">(whoami):</span><span class="se">\$</span><span class="s2">(whoami) '</span><span class="k">${</span><span class="nv">REMOTE_APP_PATH</span><span class="k">}</span><span class="s2">'"</span>

<span class="c"># Transfer files</span>
rsync <span class="nt">-avz</span> <span class="nt">-e</span> <span class="s2">"ssh -i </span><span class="k">${</span><span class="nv">SSH_KEY_PATH</span><span class="k">}</span><span class="s2">"</span> <span class="nt">--delete</span> <span class="se">\</span>
  <span class="s2">"</span><span class="k">${</span><span class="nv">WORKDIR</span><span class="k">}</span><span class="s2">/repo/"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">REMOTE_USER</span><span class="k">}</span><span class="s2">@</span><span class="k">${</span><span class="nv">REMOTE_HOST</span><span class="k">}</span><span class="s2">:</span><span class="k">${</span><span class="nv">REMOTE_APP_PATH</span><span class="k">}</span><span class="s2">/"</span>
</code></pre>

</div>



<p>The <code>--delete</code> flag removes files on the server that don't exist in your repository, keeping everything in sync.</p>

<p><strong>Docker Compose deployment</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">if</span> <span class="o">[</span> <span class="nt">-f</span> docker-compose.yml <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">sudo </span>docker compose down <span class="o">||</span> <span class="nb">true</span>  <span class="c"># Stop old containers</span>
  <span class="nb">sudo </span>docker compose build         <span class="c"># Build new images</span>
  <span class="nb">sudo </span>docker compose up <span class="nt">-d</span>         <span class="c"># Start containers in background</span>
<span class="k">fi</span>
</code></pre>

</div>



<p><strong>Single Dockerfile deployment</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">elif</span> <span class="o">[</span> <span class="nt">-f</span> Dockerfile <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nv">APP_TAG</span><span class="o">=</span><span class="si">$(</span><span class="nb">basename</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span><span class="s2">"</span><span class="si">)</span>

  <span class="c"># Build image</span>
  <span class="nb">sudo </span>docker build <span class="nt">--build-arg</span> <span class="nv">PORT</span><span class="o">=</span><span class="s1">'${APP_PORT}'</span> <span class="nt">-t</span> <span class="s2">"</span><span class="k">${</span><span class="nv">APP_TAG</span><span class="k">}</span><span class="s2">:latest"</span> <span class="nb">.</span>

  <span class="c"># Remove old container</span>
  <span class="nb">sudo </span>docker stop <span class="s2">"</span><span class="k">${</span><span class="nv">APP_TAG</span><span class="k">}</span><span class="s2">"</span> <span class="o">||</span> <span class="nb">true
  sudo </span>docker <span class="nb">rm</span> <span class="s2">"</span><span class="k">${</span><span class="nv">APP_TAG</span><span class="k">}</span><span class="s2">"</span> <span class="o">||</span> <span class="nb">true</span>

  <span class="c"># Run new container</span>
  <span class="nb">sudo </span>docker run <span class="nt">-d</span> <span class="se">\</span>
    <span class="nt">-p</span> <span class="s1">'${APP_PORT}'</span>:<span class="s1">'${APP_PORT}'</span> <span class="se">\</span>
    <span class="nt">-e</span> <span class="nv">PORT</span><span class="o">=</span><span class="s1">'${APP_PORT}'</span> <span class="se">\</span>
    <span class="nt">--name</span> <span class="s2">"</span><span class="k">${</span><span class="nv">APP_TAG</span><span class="k">}</span><span class="s2">"</span> <span class="se">\</span>
    <span class="s2">"</span><span class="k">${</span><span class="nv">APP_TAG</span><span class="k">}</span><span class="s2">:latest"</span>
<span class="k">fi</span>
</code></pre>

</div>



<p><strong>Health check</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo</span> <span class="s2">"Waiting for app to start..."</span>
<span class="nb">sleep </span>5

<span class="k">if </span>curl <span class="nt">-fs</span> <span class="s2">"http://localhost:</span><span class="k">${</span><span class="nv">APP_PORT</span><span class="k">}</span><span class="s2">"</span> <span class="o">&gt;</span> /dev/null 2&gt;&amp;1<span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"Application reachable on port </span><span class="k">${</span><span class="nv">APP_PORT</span><span class="k">}</span><span class="s2">."</span>
<span class="k">else
  </span><span class="nb">echo</span> <span class="s2">"WARNING: Application not responding yet."</span>
<span class="k">fi</span>
</code></pre>

</div>



<p>This gives your application a few seconds to start up, then tests if it's responding.</p>

<blockquote>
<p>[!TIP]<br>
The <code>-d</code> flag in <code>docker run</code> means "detached mode" - the container runs in the background, not blocking your terminal.</p>
</blockquote>




<h3>
  
  
  Stage 5: Configure Nginx Reverse Proxy
</h3>

<p><strong>What it does</strong>: Sets up Nginx to route external traffic (port 80) to your Docker container.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fysg1e4e5d55nq75mpdhy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fysg1e4e5d55nq75mpdhy.png" alt="Nginx Reverse Proxy" width="800" height="800"></a></p>

<p><strong>Why use a reverse proxy?</strong></p>

<p>Without Nginx, users would need to access your app like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>http://your-server.com:3000
</code></pre>

</div>



<p>With Nginx as a reverse proxy, they can use:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>http://your-server.com
</code></pre>

</div>



<p>Much cleaner! Plus, Nginx handles:</p>

<ul>
<li>SSL/TLS certificates (HTTPS)</li>
<li>Load balancing</li>
<li>Static file serving</li>
<li>DDoS protection</li>
</ul>

<p><strong>The Nginx configuration</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">server</span> <span class="p">{</span>
    <span class="kn">listen</span> <span class="mi">80</span><span class="p">;</span>
    <span class="kn">server_name</span> <span class="s">_</span><span class="p">;</span>

    <span class="kn">location</span> <span class="n">/</span> <span class="p">{</span>
        <span class="kn">proxy_pass</span> <span class="s">http://localhost:3000</span><span class="p">;</span>
        <span class="kn">proxy_set_header</span> <span class="s">Host</span> <span class="nv">$host</span><span class="p">;</span>
        <span class="kn">proxy_set_header</span> <span class="s">X-Real-IP</span> <span class="nv">$remote_addr</span><span class="p">;</span>
        <span class="kn">proxy_set_header</span> <span class="s">X-Forwarded-For</span> <span class="nv">$proxy_add_x_forwarded_for</span><span class="p">;</span>
        <span class="kn">proxy_set_header</span> <span class="s">X-Forwarded-Proto</span> <span class="nv">$scheme</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Breaking it down</strong>:</p>

<ul>
<li>
<code>listen 80</code>: Nginx listens on port 80 (HTTP)</li>
<li>
<code>server_name _</code>: Matches any domain/IP (use your domain here in production)</li>
<li>
<code>proxy_pass http://localhost:3000</code>: Forwards requests to your app</li>
<li>
<code>proxy_set_header</code> lines: Pass client information to your app</li>
</ul>

<p><strong>Deployment process</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">NGINX_CONFIG_NAME</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">APP_NAME</span><span class="k">}</span><span class="s2">.conf"</span>
<span class="nv">NGINX_PATH</span><span class="o">=</span><span class="s2">"/etc/nginx/sites-available/</span><span class="k">${</span><span class="nv">NGINX_CONFIG_NAME</span><span class="k">}</span><span class="s2">"</span>
<span class="nv">NGINX_LINK</span><span class="o">=</span><span class="s2">"/etc/nginx/sites-enabled/</span><span class="k">${</span><span class="nv">NGINX_CONFIG_NAME</span><span class="k">}</span><span class="s2">"</span>

<span class="c"># Backup old config</span>
<span class="k">if</span> <span class="o">[</span> <span class="nt">-f</span> <span class="s2">"</span><span class="nv">$NGINX_PATH</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">sudo mv</span> <span class="s2">"</span><span class="nv">$NGINX_PATH</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">NGINX_PATH</span><span class="k">}</span><span class="s2">.bak_</span><span class="si">$(</span><span class="nb">date</span> +%s<span class="si">)</span><span class="s2">"</span>
<span class="k">fi</span>

<span class="c"># Write new config</span>
<span class="nb">sudo </span>bash <span class="nt">-c</span> <span class="s2">"cat &gt; </span><span class="nv">$NGINX_PATH</span><span class="s2">"</span> <span class="o">&lt;&lt;</span><span class="sh">'</span><span class="no">CONF</span><span class="sh">'
# ... nginx config here ...
</span><span class="no">CONF

</span><span class="c"># Enable site by creating symlink</span>
<span class="nb">sudo ln</span> <span class="nt">-sf</span> <span class="s2">"</span><span class="nv">$NGINX_PATH</span><span class="s2">"</span> <span class="s2">"</span><span class="nv">$NGINX_LINK</span><span class="s2">"</span>

<span class="c"># Test configuration and reload</span>
<span class="nb">sudo </span>nginx <span class="nt">-t</span> <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>systemctl reload nginx
</code></pre>

</div>



<p>The <code>nginx -t</code> command tests the configuration before applying it, preventing syntax errors from breaking your web server.</p>




<h3>
  
  
  Stage 6: Validate Deployment
</h3>

<p><strong>What it does</strong>: Runs automated checks to ensure everything is working correctly.</p>

<p><strong>Service status checks</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check if Docker is running</span>
<span class="nb">sudo </span>systemctl is-active <span class="nt">--quiet</span> docker <span class="o">&amp;&amp;</span> <span class="se">\</span>
  <span class="nb">echo</span> <span class="s2">"Docker: Active"</span> <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"Docker: Inactive"</span>

<span class="c"># Check if Nginx is running</span>
<span class="nb">sudo </span>systemctl is-active <span class="nt">--quiet</span> nginx <span class="o">&amp;&amp;</span> <span class="se">\</span>
  <span class="nb">echo</span> <span class="s2">"Nginx: Active"</span> <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"Nginx: Inactive"</span>
</code></pre>

</div>



<p><strong>Container health</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>docker ps <span class="nt">--format</span> <span class="s2">"table {{.Names}}</span><span class="se">\t</span><span class="s2">{{.Status}}</span><span class="se">\t</span><span class="s2">{{.Ports}}"</span>
</code></pre>

</div>



<p>This shows all running containers with their status and port mappings.</p>

<p><strong>End-to-end test</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">if </span>curl <span class="nt">-fs</span> <span class="s2">"http://localhost"</span> <span class="o">&gt;</span> /dev/null 2&gt;&amp;1<span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"SUCCESS: Application reachable via Nginx on port 80!"</span>
<span class="k">else
  </span><span class="nb">echo</span> <span class="s2">"ERROR: Application not responding through Nginx!"</span>
<span class="k">fi</span>
</code></pre>

</div>



<p>This simulates a real user accessing your application through Nginx.</p>

<blockquote>
<p>[!NOTE]<br>
The <code>-f</code> flag in curl makes it fail silently on HTTP errors, and <code>-s</code> makes it quiet. Perfect for scripting!</p>
</blockquote>




<h3>
  
  
  Stage 7: Final Cleanup and Log Management
</h3>

<p><strong>What it does</strong>: Performs maintenance tasks to keep your server clean and efficient.</p>

<p><strong>Remove broken symlinks</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">if</span> <span class="o">[</span> <span class="nt">-d</span> /etc/nginx/sites-enabled <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span>find /etc/nginx/sites-enabled <span class="nt">-xtype</span> l <span class="nt">-delete</span> <span class="o">||</span> <span class="nb">true
</span><span class="k">fi</span>
</code></pre>

</div>



<p><strong>Clean up old Docker resources</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Remove stopped containers older than 7 days</span>
docker ps <span class="nt">-a</span> <span class="nt">--filter</span> <span class="s2">"status=exited"</span> <span class="nt">--format</span> <span class="s1">'{{.ID}}'</span> | <span class="se">\</span>
  xargs <span class="nt">-r</span> <span class="nt">-n1</span> docker <span class="nb">rm</span> <span class="nt">-v</span> <span class="o">||</span> <span class="nb">true</span>

<span class="c"># Remove unused images</span>
docker image prune <span class="nt">-f</span> <span class="o">||</span> <span class="nb">true</span>

<span class="c"># Remove unused networks</span>
docker network prune <span class="nt">-f</span> <span class="o">||</span> <span class="nb">true</span>
</code></pre>

</div>



<p>This prevents your server from filling up with old Docker artifacts.</p>

<p><strong>Log rotation</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Compress current log</span>
<span class="nb">gzip</span> <span class="nt">-c</span> <span class="s2">"</span><span class="k">${</span><span class="nv">LOGFILE</span><span class="k">}</span><span class="s2">"</span> <span class="o">&gt;</span> <span class="s2">"</span><span class="k">${</span><span class="nv">LOGFILE</span><span class="k">}</span><span class="s2">.gz"</span> <span class="o">||</span> <span class="nb">true</span>

<span class="c"># Delete logs older than 30 days</span>
find <span class="s2">"</span><span class="k">${</span><span class="nv">LOGDIR</span><span class="k">}</span><span class="s2">"</span> <span class="nt">-type</span> f <span class="nt">-name</span> <span class="s2">"deploy_*.log.gz"</span> <span class="nt">-mtime</span> +30 <span class="nt">-exec</span> <span class="nb">rm</span> <span class="nt">-f</span> <span class="o">{}</span> <span class="se">\;</span>
</code></pre>

</div>



<p>This keeps your deployment history while managing disk space.</p>




<h2>
  
  
  Advanced Features
</h2>

<h3>
  
  
  Idempotency
</h3>

<p><strong>What is idempotency?</strong></p>

<p>A script is idempotent if running it multiple times produces the same result as running it once. Our script is idempotent because:</p>

<ul>
<li>
<code>rsync --delete</code> ensures remote files match local files exactly</li>
<li>Old Docker containers are removed before creating new ones</li>
<li>Nginx symlinks use <code>-sf</code> (force) to overwrite existing links</li>
<li>Service installations check if already installed</li>
</ul>

<p><strong>Why it matters</strong>: You can safely re-run the deployment without worrying about duplicate resources or conflicts.</p>

<h3>
  
  
  Error Handling
</h3>

<p>The script includes comprehensive error handling:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>die<span class="o">()</span> <span class="o">{</span>
  log <span class="s2">"ERROR"</span> <span class="s2">"</span><span class="nv">$2</span><span class="s2">"</span>
  <span class="nb">exit</span> <span class="s2">"</span><span class="k">${</span><span class="nv">1</span><span class="k">:-</span><span class="nv">1</span><span class="k">}</span><span class="s2">"</span>
<span class="o">}</span>

<span class="c"># Usage example:</span>
<span class="k">if</span> <span class="o">[</span> <span class="nt">-z</span> <span class="s2">"</span><span class="k">${</span><span class="nv">GIT_REPO_URL</span><span class="k">}</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span>die 10 <span class="s2">"Git repository URL is required"</span>
<span class="k">fi</span>
</code></pre>

</div>



<p>Each error has a unique exit code for easier troubleshooting:</p>

<ul>
<li>
<code>10-19</code>: Parameter validation errors</li>
<li>
<code>20-29</code>: Repository and SSH errors</li>
<li>
<code>30-39</code>: Remote setup errors</li>
<li>
<code>40-49</code>: Deployment errors</li>
<li>
<code>50-59</code>: Nginx configuration errors</li>
<li>
<code>60+</code>: Validation errors</li>
</ul>

<h3>
  
  
  Comprehensive Logging
</h3>

<p>Every action is logged with timestamps:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>log <span class="s2">"INFO"</span> <span class="s2">"Starting Stage 3: remote environment preparation..."</span>
</code></pre>

</div>



<p>Logs are:</p>

<ul>
<li>Written to timestamped files</li>
<li>Compressed automatically</li>
<li>Rotated after 30 days</li>
<li>Displayed in real-time during deployment</li>
</ul>




<h2>
  
  
  Common Issues and Troubleshooting
</h2>

<h3>
  
  
  Issue 1: SSH Connection Fails
</h3>

<p><strong>Symptoms</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ERROR: Cannot SSH into remote. Check SSH key, username, host IP, or firewall.
</code></pre>

</div>



<p><strong>Solutions</strong>:</p>

<ol>
<li>Verify SSH key permissions: <code>chmod 600 ~/.ssh/your-key.pem</code>
</li>
<li>Test SSH manually: <code>ssh -i ~/.ssh/your-key.pem user@host</code>
</li>
<li>Check server firewall allows port 22</li>
<li>Verify the SSH key is added to server's <code>~/.ssh/authorized_keys</code>
</li>
</ol>

<h3>
  
  
  Issue 2: Docker Build Fails
</h3>

<p><strong>Symptoms</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ERROR: The command '/bin/sh -c npm install' returned a non-zero code
</code></pre>

</div>



<p><strong>Solutions</strong>:</p>

<ol>
<li>Check your <code>Dockerfile</code> syntax</li>
<li>Verify your app's dependencies are correct</li>
<li>Look at the full error in the log file</li>
<li>Try building locally first: <code>docker build -t test .</code>
</li>
</ol>

<h3>
  
  
  Issue 3: Application Not Responding
</h3>

<p><strong>Symptoms</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>WARNING: Application not responding yet.
</code></pre>

</div>



<p><strong>Solutions</strong>:</p>

<ol>
<li>Check container logs: <code>docker logs &lt;container-name&gt;</code>
</li>
<li>Verify the port number is correct</li>
<li>Ensure your app is binding to <code>0.0.0.0</code>, not <code>localhost</code>
</li>
<li>Check if the container is running: <code>docker ps</code>
</li>
</ol>

<h3>
  
  
  Issue 4: Nginx Configuration Error
</h3>

<p><strong>Symptoms</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>nginx: [emerg] unexpected "}" in /etc/nginx/sites-available/app.conf:10
</code></pre>

</div>



<p><strong>Solutions</strong>:</p>

<ol>
<li>The script tests configuration before applying: <code>nginx -t</code>
</li>
<li>Check for syntax errors in the Nginx config section</li>
<li>Verify the <code>APP_PORT</code> variable is correctly substituted</li>
</ol>




<h2>
  
  
  Best Practices and Security Considerations
</h2>

<h3>
  
  
  1. SSH Key Management
</h3>

<p><strong>Do</strong>:</p>

<ul>
<li>Use separate SSH keys for deployment (not your personal key)</li>
<li>Restrict key permissions: <code>chmod 600 key.pem</code>
</li>
<li>Use SSH key passphrases for extra security</li>
</ul>

<p><strong>Don't</strong>:</p>

<ul>
<li>Commit SSH keys to Git</li>
<li>Share deployment keys between projects</li>
<li>Use weak key types (use RSA 4096-bit or ed25519)</li>
</ul>

<h3>
  
  
  2. Secrets Management
</h3>

<p><strong>Do</strong>:</p>

<ul>
<li>Use environment variables for sensitive data</li>
<li>Consider using secret management tools (HashiCorp Vault, AWS Secrets Manager)</li>
<li>Rotate credentials regularly</li>
</ul>

<p><strong>Don't</strong>:</p>

<ul>
<li>Hardcode passwords or API keys in your code</li>
<li>Commit <code>.env</code> files to Git</li>
<li>Log sensitive information</li>
</ul>

<h3>
  
  
  3. Server Security
</h3>

<p><strong>Do</strong>:</p>

<ul>
<li>Keep server software updated: <code>sudo apt update &amp;&amp; sudo apt upgrade</code>
</li>
<li>Use firewall rules (UFW or security groups)</li>
<li>Enable automatic security updates</li>
<li>Implement SSL/TLS with Let's Encrypt</li>
</ul>

<p><strong>Don't</strong>:</p>

<ul>
<li>Run containers as root unnecessarily</li>
<li>Expose Docker daemon to the internet</li>
<li>Use default passwords</li>
</ul>

<h3>
  
  
  4. Deployment Strategy
</h3>

<p><strong>Do</strong>:</p>

<ul>
<li>Test deployments in a staging environment first</li>
<li>Implement health checks</li>
<li>Keep backups before deploying</li>
<li>Use version tags for Docker images</li>
</ul>

<p><strong>Don't</strong>:</p>

<ul>
<li>Deploy directly to production without testing</li>
<li>Deploy during peak traffic hours</li>
<li>Skip validation checks</li>
</ul>




<h2>
  
  
  Taking It Further: Next Steps
</h2>

<p>Now that you have a working deployment script, consider these enhancements:</p>

<h3>
  
  
  1. Add CI/CD Integration
</h3>

<p>Integrate with GitHub Actions or GitLab CI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/deploy.yml</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">Deploy</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">]</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">deploy</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v3</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy to server</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">echo "${{ secrets.SSH_KEY }}" &gt; key.pem</span>
          <span class="s">chmod 600 key.pem</span>
          <span class="s">./deploy.sh</span>
</code></pre>

</div>



<h3>
  
  
  2. Implement Blue-Green Deployment
</h3>

<p>Run two versions of your app and switch between them:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start new version on different port</span>
docker run <span class="nt">-d</span> <span class="nt">-p</span> 3001:3000 <span class="nt">--name</span> app-blue app:v2

<span class="c"># After validation, update Nginx to point to new version</span>
<span class="c"># Then remove old version</span>
</code></pre>

</div>



<h3>
  
  
  3. Add Monitoring
</h3>

<p>Use tools like:</p>

<ul>
<li>
<strong>Prometheus</strong> for metrics</li>
<li>
<strong>Grafana</strong> for visualization</li>
<li>
<strong>Sentry</strong> for error tracking</li>
<li>
<strong>Uptime Robot</strong> for availability monitoring</li>
</ul>

<h3>
  
  
  4. Database Management
</h3>

<p>Add database backup and migration steps:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Backup before deployment</span>
docker <span class="nb">exec </span>postgres pg_dump <span class="nt">-U</span> user dbname <span class="o">&gt;</span> backup.sql

<span class="c"># Run migrations</span>
docker <span class="nb">exec </span>app npm run migrate
</code></pre>

</div>



<h3>
  
  
  5. Multi-Environment Support
</h3>

<p>Extend the script to handle dev/staging/production:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">read</span> <span class="nt">-rp</span> <span class="s2">"Environment (dev/staging/prod): "</span> ENVIRONMENT
<span class="c"># Use different configs based on environment</span>
</code></pre>

</div>






<h2>
  
  
  Conclusion
</h2>

<p>Congratulations! 🎉 You now have a comprehensive understanding of automated Docker deployment using Bash scripts. Let's recap what we've covered:</p>

<p>✅ <strong>The basics</strong>: What deployment automation is and why it matters<br><br>
✅ <strong>The workflow</strong>: How code gets from your computer to a live server<br><br>
✅ <strong>The 7 stages</strong>: Every step of the deployment process in detail<br><br>
✅ <strong>Advanced concepts</strong>: Idempotency, error handling, and logging<br><br>
✅ <strong>Troubleshooting</strong>: Common issues and how to fix them<br><br>
✅ <strong>Best practices</strong>: Security and deployment strategies<br><br>
✅ <strong>Next steps</strong>: How to enhance your deployment pipeline</p>

<h3>
  
  
  Key Takeaways
</h3>

<ol>
<li>
<strong>Automation saves time</strong>: What used to take hours now takes minutes</li>
<li>
<strong>Consistency matters</strong>: The same process every time reduces errors</li>
<li>
<strong>Logging is crucial</strong>: Good logs make troubleshooting 10x easier</li>
<li>
<strong>Security is paramount</strong>: Never compromise on security practices</li>
<li>
<strong>Iteration is key</strong>: Start simple, then add features as needed</li>
</ol>

<h3>
  
  
  Your Deployment Journey
</h3>

<p>This script is a foundation, not a final destination. As you grow in your DevOps journey:</p>

<ul>
<li>Customize it for your specific needs</li>
<li>Add features that solve your problems</li>
<li>Share your improvements with the community</li>
<li>Keep learning and experimenting</li>
</ul>

<h3>
  
  
  Additional Resources
</h3>

<p>Want to dive deeper? Check out these resources:</p>

<ul>
<li>
<strong>Docker Documentation</strong>: <a href="https://docs.docker.com" rel="noopener noreferrer">docs.docker.com</a>
</li>
<li>
<strong>Nginx Beginner's Guide</strong>: <a href="http://nginx.org/en/docs/beginners_guide.html" rel="noopener noreferrer">nginx.org/en/docs/beginners_guide.html</a>
</li>
<li>
<strong>Bash Scripting Guide</strong>: <a href="https://tldp.org/LDP/abs/html/" rel="noopener noreferrer">tldp.org/LDP/abs/html/</a>
</li>
<li>
<strong>DevOps Roadmap</strong>: <a href="https://roadmap.sh/devops" rel="noopener noreferrer">roadmap.sh/devops</a>
</li>
</ul>




<h2>
  
  
  Get Started Today!
</h2>

<p>Ready to deploy your first application automatically? Here's your action plan:</p>

<ol>
<li>
<strong>Clone the script</strong> from the repository</li>
<li>
<strong>Set up a test server</strong> (DigitalOcean has a $5/month droplet)</li>
<li>
<strong>Create a simple Dockerized app</strong> (a "Hello World" Node.js app works great)</li>
<li>
<strong>Run the deployment script</strong> following this guide</li>
<li>
<strong>Celebrate</strong> when you see your app live! 🚀</li>
</ol>

<p>Remember: Every expert was once a beginner. Don't be afraid to experiment, break things (in a test environment!), and learn from mistakes.</p>




<h2>
  
  
  Questions or Feedback?
</h2>

<p>I'd love to hear about your deployment journey! Did this guide help you? What challenges did you face? What would you like to see covered in a follow-up article?</p>

<p>Drop your thoughts in the comments below, and happy deploying! 💻✨</p>




<h2>
  
  
  About This Article
</h2>

<p>This guide was created as part of the HNG Internship Stage 1 DevOps task. The script implements best practices for production-ready deployments while remaining accessible to beginners.</p>

<p><strong>Connect with me</strong>: [Your social links here]</p>

<p><strong>Want to learn more about DevOps?</strong> Check out the <a href="https://hng.tech/internship" rel="noopener noreferrer">HNG Internship program</a> for hands-on learning opportunities.</p>




<p><em>Tags: #DevOps #Docker #Nginx #BashScripting #Deployment #Automation #Linux #CloudComputing #WebDevelopment #Tutorial</em></p>

