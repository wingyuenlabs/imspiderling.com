---
Title: Day 23: Jenkins Freestyle Project for DevOps Engineers
Description: 
Author: Udoh Deborah
Date: 2025-08-13T21:37:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’ve ever wanted to automate building, testing, and deploying your applications, this is your playground.</p>




<h3>
  
  
  <strong>What is CI/CD?</strong>
</h3>

<ul>
<li>
<strong>CI (Continuous Integration)</strong>: Automatically integrates code changes into a shared repository and runs builds/tests to catch bugs early.</li>
<li>
<strong>CD (Continuous Delivery)</strong>: Ensures code is always in a release-ready state with automated deployments to staging or production.</li>
</ul>

<p>Together, CI/CD keeps development fast, reliable, and efficient.</p>

<h3>
  
  
  <strong>What is a Build Job?</strong>
</h3>

<p>In Jenkins, a <strong>build job</strong> is a set of instructions to automate tasks — from compiling code, testing, and packaging, to deploying it anywhere.</p>

<h3>
  
  
  <strong>What is a Freestyle Project?</strong>
</h3>

<p>A <strong>Freestyle Project</strong> in Jenkins is the most beginner-friendly way to:</p>

<ul>
<li>Build &amp; test code</li>
<li>Run Docker commands</li>
<li>Deploy applications</li>
<li>Chain together multiple automation steps</li>
</ul>

<h3>
  
  
  <strong>Today’s Challenge Tasks:</strong>
</h3>

<p>** Task 1**</p>

<ol>
<li>Create an agent for your Dockerized app.</li>
<li>Make a new <strong>Freestyle Project</strong> in Jenkins.</li>
<li>Add a build step to:</li>
</ol>

<ul>
<li>Run <code>docker build</code> to create your image.</li>
<li>Run <code>docker run</code> to launch your container.</li>
</ul>

<p><strong>Task 2</strong></p>

<ol>
<li>Create a Jenkins project to run:</li>
</ol>

<ul>
<li>
<p><code>docker-compose up -d</code> to start multiple containers (from your Day 19 project).</p>

<ol>
<li>Add a cleanup step with:</li>
</ol>
</li>
<li><p><code>docker-compose down</code> to stop and remove containers.</p></li>
</ul>

<p>This is a <strong>hands-on step</strong> toward mastering CI/CD automation with Jenkins and Docker.</p>




<h2>
  
  
  Prerequisite (do once)
</h2>

<ol>
<li><strong>Confirm Jenkins is up</strong></li>
</ol>

<ul>
<li>Open: <code>http://&lt;your-host&gt;:8080</code>
</li>
<li>Unlock with the password from:
<code>sudo cat /var/lib/jenkins/secrets/initialAdminPassword</code>
</li>
<li>Create admin user → “Install suggested plugins”</li>
</ul>

<ol>
<li>
<strong>Give Jenkins access to Docker</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># On the machine where Jenkins runs</span>
<span class="nb">sudo </span>usermod <span class="nt">-aG</span> docker jenkins
<span class="nb">sudo </span>systemctl restart jenkins
<span class="c"># Verify the jenkins user can talk to Docker</span>
<span class="nb">sudo</span> <span class="nt">-u</span> jenkins <span class="nt">-H</span> sh <span class="nt">-c</span> <span class="s1">'docker version &amp;&amp; docker ps'</span>
</code></pre>

</div>



<ol>
<li>
<strong>Install Git &amp; Compose on the agent/host</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt-get update
<span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> git
docker-compose version <span class="o">||</span> <span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> docker-compose-plugin
</code></pre>

</div>






<h2>
  
  
  Create/prepare an Agent (Node)
</h2>

<p>You can use the <strong>Built-in Node</strong> if Jenkins and Docker are on the same machine:</p>

<ul>
<li>Jenkins → <strong>Manage Jenkins</strong> → <strong>Nodes</strong> → <strong>Built-in Node</strong> → <strong>Configure</strong>
</li>
<li>Add a <strong>Label</strong> (e.g., <code>docker</code>)</li>
<li>Set <strong># of executors</strong> (start with 1) → <strong>Save</strong>
</li>
</ul>

<p>(If you have a separate Docker host, add it as an SSH agent with label <code>docker</code> and ensure Docker is installed there.)</p>




<h2>
  
  
  Task 1: Freestyle job to <strong>build</strong> and <strong>run</strong> your container
</h2>

<p><strong>A. Prepare your repository</strong><br>
Your app repo should have a <code>Dockerfile</code> at the root. Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>my-app/
  ├─ Dockerfile
  ├─ src/...
  └─ ...
</code></pre>

</div>



<p><strong>B. Create the job</strong></p>

<ol>
<li>Jenkins → <strong>New Item</strong> → <strong>Freestyle project</strong> → name it <code>app-build-run</code> → <strong>OK</strong>
</li>
<li>
<strong>General</strong>: Check <strong>Restrict where this project can be run</strong> → Label: <code>docker</code>
</li>
<li>
<strong>Source Code Management</strong> → <strong>Git</strong> → Repo URL (add credentials if private)</li>
<li>
<strong>Build Triggers</strong> → <strong>Build periodically</strong> → Schedule: <code>H * * * *</code> (once every hour, nicely scattered)</li>
<li>
<strong>Build</strong> → <strong>Add build step</strong> → <strong>Execute shell</strong>
Paste:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">set</span> <span class="nt">-e</span>

   <span class="nv">APP_NAME</span><span class="o">=</span><span class="s2">"myapp"</span>
   <span class="nv">IMAGE</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">APP_NAME</span><span class="k">}</span><span class="s2">:</span><span class="k">${</span><span class="nv">BUILD_NUMBER</span><span class="k">}</span><span class="s2">"</span>
   <span class="nv">PORT</span><span class="o">=</span>8080   <span class="c"># change if your app uses a different port</span>

   <span class="nb">echo</span> <span class="s2">"[INFO] Building image </span><span class="k">${</span><span class="nv">IMAGE</span><span class="k">}</span><span class="s2">"</span>
   docker build <span class="nt">-t</span> <span class="s2">"</span><span class="k">${</span><span class="nv">IMAGE</span><span class="k">}</span><span class="s2">"</span> <span class="nb">.</span>

   <span class="nb">echo</span> <span class="s2">"[INFO] Stopping previous container (if any)"</span>
   docker <span class="nb">rm</span> <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">APP_NAME</span><span class="k">}</span><span class="s2">"</span> 2&gt;/dev/null <span class="o">||</span> <span class="nb">true

   echo</span> <span class="s2">"[INFO] Running new container"</span>
   docker run <span class="nt">-d</span> <span class="nt">--name</span> <span class="s2">"</span><span class="k">${</span><span class="nv">APP_NAME</span><span class="k">}</span><span class="s2">"</span> <span class="nt">-p</span> <span class="k">${</span><span class="nv">PORT</span><span class="k">}</span>:<span class="k">${</span><span class="nv">PORT</span><span class="k">}</span> <span class="s2">"</span><span class="k">${</span><span class="nv">IMAGE</span><span class="k">}</span><span class="s2">"</span>

   <span class="nb">echo</span> <span class="s2">"[INFO] Container status:"</span>
   docker ps <span class="nt">--filter</span> <span class="s2">"name=</span><span class="k">${</span><span class="nv">APP_NAME</span><span class="k">}</span><span class="s2">"</span>
</code></pre>

</div>



<ol>
<li>
<strong>Save</strong> → <strong>Build Now</strong>
</li>
<li>Test the app at <code>http://&lt;host&gt;:8080</code> (or your mapped port).</li>
</ol>

<blockquote>
<p>Tip: If your app listens on a different internal port, map accordingly (e.g., <code>-p 8000:8000</code>).</p>
</blockquote>




<h2>
  
  
  Task 2: Freestyle job to run <strong>docker compose up -d</strong> and clean up
</h2>

<p><strong>A. Repo structure</strong><br>
Your repo should contain a <code>docker-compose.yml</code>. Example minimal:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">web</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span> <span class="s">.</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8080:8080"</span>
  <span class="na">db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:16</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="na">POSTGRES_PASSWORD</span><span class="pi">:</span> <span class="s">example</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">db_data:/var/lib/postgresql/data</span>
<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">db_data</span><span class="pi">:</span>
</code></pre>

</div>



<p><strong>B. Create the job</strong></p>

<ol>
<li>Jenkins → <strong>New Item</strong> → <strong>Freestyle project</strong> → name it <code>compose-up</code> → <strong>OK</strong>
</li>
<li>
<strong>Restrict where this project can be run</strong> → Label: <code>docker</code>
</li>
<li>
<strong>Source Code Management</strong> → <strong>Git</strong> → Your repo URL</li>
<li>
<strong>Build Triggers</strong> (optional) → <strong>Build periodically</strong> (e.g., <code>H/30 * * * *</code> to run every ~30 mins)</li>
<li>
<strong>Build</strong> → <strong>Add build step</strong> → <strong>Execute shell</strong>
If you have <strong><code>docker compose</code></strong>:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">set</span> <span class="nt">-e</span>

   <span class="nv">FILE</span><span class="o">=</span><span class="s2">"docker-compose.yml"</span>

   <span class="nb">echo</span> <span class="s2">"[INFO] Pull/build &amp; start in detached mode"</span>
   docker compose <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">FILE</span><span class="k">}</span><span class="s2">"</span> up <span class="nt">-d</span> <span class="nt">--build</span>

   <span class="nb">echo</span> <span class="s2">"[INFO] Current stack:"</span>
   docker compose <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">FILE</span><span class="k">}</span><span class="s2">"</span> ps
</code></pre>

</div>



<p>If your system only has the old binary:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   docker-compose <span class="nt">-f</span> docker-compose.yml up <span class="nt">-d</span> <span class="nt">--build</span>
   docker-compose <span class="nt">-f</span> docker-compose.yml ps
</code></pre>

</div>



<h3>
  
  
  Cleanup: <code>docker compose down</code>
</h3>

<p>Trap cleanup in the same step (runs even if the build fails)**<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">set</span> <span class="nt">-e</span>
<span class="nv">FILE</span><span class="o">=</span><span class="s2">"docker-compose.yml"</span>

cleanup<span class="o">()</span> <span class="o">{</span>
  <span class="nb">echo</span> <span class="s2">"[CLEANUP] Bringing stack down"</span>
  docker compose <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">FILE</span><span class="k">}</span><span class="s2">"</span> down <span class="nt">--volumes</span> <span class="nt">--remove-orphans</span>
<span class="o">}</span>
<span class="nb">trap </span>cleanup EXIT

docker compose <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">FILE</span><span class="k">}</span><span class="s2">"</span> up <span class="nt">-d</span> <span class="nt">--build</span>
docker compose <span class="nt">-f</span> <span class="s2">"</span><span class="k">${</span><span class="nv">FILE</span><span class="k">}</span><span class="s2">"</span> ps

<span class="c"># Put your tests here if any; cleanup will still run on EXIT</span>
</code></pre>

</div>



<blockquote>
<p>If you want the stack to <strong>stay up</strong> for manual testing and only clean in dedicated jobs, skip <code>down</code> here and add a separate Freestyle job called <code>compose-down</code> with:<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker compose <span class="nt">-f</span> docker-compose.yml down <span class="nt">--volumes</span> <span class="nt">--remove-orphans</span>
</code></pre>

</div>






<h2>
  
  
  If your repo is private
</h2>

<ul>
<li>Jenkins → <strong>Manage Jenkins</strong> → <strong>Credentials</strong> → <strong>System</strong> → <strong>Global</strong> → <strong>Add Credentials</strong>
</li>
<li>Choose <strong>Username with password</strong> or <strong>SSH Username with private key</strong>
</li>
<li>Reference it in the job’s Git settings.</li>
</ul>




<p>Add another <strong>Execute shell</strong> step at the end:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>bash
echo "[VERIFY] Hitting health endpoint"
curl -fsS http://localhost:8080/ || (echo "App not responding" &amp;&amp; exit 1)
</code></pre>

</div>



<p>That’s it! Run both jobs, check the <strong>Console Output</strong> to confirm each stage, and you’re done.</p>

