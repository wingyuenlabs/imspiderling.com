---
Title: Getting Started with Vagrant
Description: 
Author: ZINSOU Trinité
Date: 2025-09-14T21:35:22.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  🧠 Introduction
</h3>

<blockquote>
<p><em>"What if you could magically spin up as many machines as you want for your test environments? No wand, just Vagrant."</em></p>

<p>Test environments can quickly become a nightmare: bad installs, version conflicts, etc. What if we could change that with a tool that makes you feel like a <strong>DevOps wizard</strong>? 🎩✨</p>
</blockquote>

<p>In this article, we will:</p>

<ul>
<li><p>Discover what Vagrant is (without falling asleep) 😴❌</p></li>
<li><p>Use it to create <strong>two Ubuntu VMs</strong> 🖥️🖥️ and prepare a test environment for our series on incremental backups with Postgres + Barman</p></li>
<li><p>And test our setup like pros 🧪</p></li>
</ul>

<h2>
  
  
  🪄 What is Vagrant?
</h2>

<blockquote>
<p><em>"Vagrant is kind of like Docker, but for VMs. It's your conductor for VirtualBox, VMware, Hyper-V, and other hypervisors. It's a super reliable tool to manage complete environments with real virtual machines in a simple and automated way."</em></p>
</blockquote>

<p>It's a tool developed by <a href="https://www.hashicorp.com" rel="noopener noreferrer">Hashicorp</a> that makes it easy to manage VMs with fully isolated dependencies and configurations, all in a unique and disposable environment.</p>

<p>You write a <code>Vagrantfile</code>, type <code>vagrant up</code>, and boom 💥: your machine(s) is(are) ready to use. No need to click through 27 buttons in VirtualBox or fight with Hyper-V.</p>

<p>Vagrant saves you from:</p>

<p>❌ Downloading an ISO<br><br>
❌ Clicking through the installer manually<br><br>
❌ Struggling with networking and scripts<br><br>
✅ You can <strong>do everything automatically</strong>, and <strong>reset your environment anytime</strong>, like a quick save in your dev life.</p>
<h3>
  
  
  🔍 Why is it useful?
</h3>

<ul>
<li><p>🔄 <strong>Easy testing</strong>: want to test a Postgres config? SSH? A cluster? A network lab? Boom, spin up a VM.</p></li>
<li><p>💥 <strong>Safe</strong>: easily prepare demos or tutorials without polluting your main machine. If you break everything, just run <code>vagrant destroy &amp;&amp; vagrant up</code>.</p></li>
<li><p>🤝 <strong>Shareable</strong>: working in a team? Commit your <code>Vagrantfile</code> and everyone gets <em>the same environment</em>.</p></li>
<li><p>🧪 <strong>Closer to production</strong>: you can create environments that <em>really mimic</em> a real server (unlike Docker). Perfect for learning sysadmin or DevOps.</p></li>
</ul>
<h3>
  
  
  🛠️ How does it work?
</h3>

<p>Vagrant relies on:</p>

<ul>
<li><p><strong>A provider</strong> (usually VirtualBox or Hyper-V) to run the machines</p></li>
<li><p><strong>Boxes</strong>: pre-built system images deployable in any Vagrant environment. They're essentially packaged distributions for uniform deployment. You can explore official boxes <a href="https://portal.cloud.hashicorp.com/vagrant/discover" rel="noopener noreferrer">here</a></p></li>
<li><p><strong>A Vagrantfile</strong> (your recipe): the file where the magic is defined</p></li>
<li><p>Scripts or tools to <strong>provision</strong> the machine (install software, configure base services)</p></li>
</ul>

<p>Vagrant will provision the machine based on what's described in the Vagrantfile (usually in the project root). A Vagrantfile is a Ruby file, but it doesn’t require deep Ruby knowledge. It defines the machine specs (storage, RAM, network, etc.) and sets up the required tools for it to run properly.</p>
<h2>
  
  
  🎓 Getting Started with Vagrant
</h2>

<p>Next, we’ll get hands-on with Vagrant and set up a lab for our incremental backup series with Postgres.<br><br>
In the end, we’ll boot two VMs: one with Postgres, and another with Barman.</p>
<h3>
  
  
  🚀 <strong>Installing Vagrant</strong>
</h3>

<p>You can install Vagrant via the CLI or by using an executable from the <a href="https://developer.hashicorp.com/vagrant/downloads#linux" rel="noopener noreferrer">official documentation</a>.<br><br>
You’ll also need a working VirtualBox setup, which is the provider we’ll use.</p>
<h3>
  
  
  🛠️ Using Vagrant
</h3>

<p>Let's create our first <code>Vagrantfile</code> with the minimal config to run a VM. Create a folder and inside it, a <code>Vagrantfile</code> like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Vagrant</span><span class="p">.</span><span class="nf">configure</span><span class="p">(</span><span class="s2">"2"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">config</span><span class="o">|</span>
  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">box</span> <span class="o">=</span> <span class="s2">"ubuntu/focal64"</span>
  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">hostname</span> <span class="o">=</span> <span class="s2">"vagrantbox"</span>
<span class="k">end</span>
</code></pre>

</div>



<p>Vagrant provides a wide variety of commands to manage the different components of the Vagrant ecosystem. For example, <code>vagrant box list</code> and <code>vagrant box add box_name</code> respectively list the boxes available locally and add a box to the local collection.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>vagrant box add ubuntu/focal64
<span class="o">==&gt;</span> box: Loading metadata <span class="k">for </span>box <span class="s1">'ubuntu/focal64'</span>
    box: URL: https://vagrantcloud.com/api/v2/vagrant/ubuntu/focal64
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1<span class="o">)</span> hyperv
2<span class="o">)</span> libvirt
3<span class="o">)</span> virtualbox
4<span class="o">)</span> vmware_desktop

Enter your choice: 3
<span class="o">==&gt;</span> box: Adding box <span class="s1">'ubuntu/focal64'</span> <span class="o">(</span>v2004.01<span class="o">)</span> <span class="k">for </span>provider: virtualbox
    box: Downloading: https://vagrantcloud.com/ubuntu/boxes/focal64/versions/20240821.0.1/providers/virtualbox/unknown/vagrant.box
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>vagrant box list             
ubuntu/bionic64 <span class="o">(</span>virtualbox, 20230607.0.0<span class="o">)</span>
ubuntu/focal64  <span class="o">(</span>virtualbox, 20240220.0.0<span class="o">)</span>
</code></pre>

</div>



<p>More info on box management <a href="https://developer.hashicorp.com/vagrant/docs/cli/box" rel="noopener noreferrer">here</a>.<br><br>
Next, let's verify that the syntax of the file is correct using <code>vagrant validate</code>.<br><br>
If there are no errors, you will see the following message: Vagrantfile validated successfully.</p>

<p>Then launch your VM:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>vagrant up             
Bringing machine <span class="s1">'default'</span> up with <span class="s1">'virtualbox'</span> provider...
<span class="o">==&gt;</span> default: Importing base box <span class="s1">'ubuntu/focal64'</span>...
<span class="o">==&gt;</span> default: Matching MAC address <span class="k">for </span>NAT networking...
<span class="o">==&gt;</span> default: Checking <span class="k">if </span>box <span class="s1">'ubuntu/focal64'</span> version <span class="s1">'20240220.0.0'</span> is up to date...
<span class="o">==&gt;</span> default: A newer version of the box <span class="s1">'ubuntu/focal64'</span> <span class="k">for </span>provider <span class="s1">'virtualbox'</span> is
<span class="o">==&gt;</span> default: available! You currently have version <span class="s1">'20240220.0.0'</span><span class="nb">.</span> The latest is version
<span class="o">==&gt;</span> default: <span class="s1">'20240821.0.1'</span><span class="nb">.</span> Run <span class="sb">`</span>vagrant box update<span class="sb">`</span> to update.
<span class="o">==&gt;</span> default: Setting the name of the VM: ocr_default_1749933808776_2967
<span class="o">==&gt;</span> default: Clearing any previously <span class="nb">set </span>network interfaces...
<span class="o">[</span>...]

<span class="nv">$ </span>vagrant status
Current machine states:

default                   running <span class="o">(</span>virtualbox<span class="o">)</span>
<span class="o">[</span>...]Check the status:
</code></pre>

</div>



<p>Once the VM is up and running, we can connect to it via SSH to perform various operations using the command <code>vagrant ssh machine_name</code>.</p>

<p>Vagrant can also be effectively used in a context that requires a multi-tier architecture — i.e., multiple machines (sometimes with different operating systems) at once, as is often the case in production. You simply need to define as many VM configurations as needed.</p>

<p>Let’s now connect to the machine we’ve prepared so far.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>vagrant ssh default             
Welcome to Ubuntu 20.04.6 LTS <span class="o">(</span>GNU/Linux 5.4.0-172-generic x86_64<span class="o">)</span>

 <span class="k">*</span> Documentation:  https://help.ubuntu.com
 <span class="k">*</span> Management:     https://landscape.canonical.com
 <span class="k">*</span> Support:        https://ubuntu.com/pro
<span class="o">[</span>...]
vagrant@vagrantbox:~<span class="err">$</span>
</code></pre>

</div>



<p>The diagram below illustrates the <strong>typical lifecycle of a Vagrant virtual machine</strong>, from initialization to complete removal.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4agxxf1cm6b5lklm5huj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4agxxf1cm6b5lklm5huj.png" alt="Vagrant Lifecycle" width="800" height="626"></a></p>

<h3>
  
  
  Main Steps
</h3>

<ol>
<li><p><code>vagrant init</code> creates a Vagrantfile</p></li>
<li>
<p><code>Configure the Vagrantfile</code></p>

<p>This is an important step in setting up the <code>Vagrantfile</code>.<br><br>
Here, we define everything needed to prepare the machine (provisioning and more). Then, using the <code>vagrant up</code> command, we start the virtual machine. If it doesn’t exist yet, it will be created and provisioned.</p>
</li>
<li><p><code>Running</code><br><br>
The machine is now running. From this point on, you have several options available to you:</p></li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>* `vagrant reload` : restart the VM with the updated config

* `vagrant suspend` / `vagrant resume` : to suspend and resume the VM

* `vagrant halt` : to shut down the VM properly

* `vagrant provision` : rerun the provisioning scripts
</code></pre>

</div>

<ol>
<li>
<code>Destroy</code>
With <code>vagrant destroy</code>, the VM is completely removed.
You then return to a clean state.</li>
</ol>

<h2>
  
  
  🚀 Provisioning Vagrant VMs
</h2>

<p>Now that our base environment is defined in the Vagrantfile, it's time to prepare each machine so they are ready to use that is, install everything necessary to operate them. This is the provisioning step.</p>

<p>Vagrant provides several interfaces to provision our VMs: shell, file, Chef, Docker, Ansible, etc.<br><br>
Next, we will explore some of these provisioners and prepare what’s needed to provision our VMs for the Postgres and Vagrant lab.</p>
<h3>
  
  
  1. Shell Provisioner
</h3>

<p>It allows you to specify a series of instructions to execute in order to provision the machine. This interface supports two options by default: <code>inline</code> and <code>path</code>.<br><br>
These let you specify either a command directly within the Vagrantfile or the path to a ready-to-use bash script to set up the machine, as shown in the example below.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Vagrant</span><span class="p">.</span><span class="nf">configure</span><span class="p">(</span><span class="s2">"2"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">config</span><span class="o">|</span>
  <span class="c1"># ... others configuration</span>
  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">provision</span> <span class="s2">"shell"</span><span class="p">,</span>
    <span class="ss">inline: </span><span class="s2">"echo Hello, World"</span>
<span class="k">end</span>

<span class="no">Vagrant</span><span class="p">.</span><span class="nf">configure</span><span class="p">(</span><span class="s2">"2"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">config</span><span class="o">|</span>
  <span class="c1"># ... others configuration</span>
  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">provision</span> <span class="s2">"shell"</span><span class="p">,</span> <span class="ss">path: </span><span class="sr">/path/</span><span class="n">to</span><span class="o">/</span><span class="n">my</span><span class="o">/</span><span class="n">bash</span><span class="o">/</span><span class="n">script</span>
<span class="k">end</span>
</code></pre>

</div>



<h3>
  
  
  2. File Provisioner
</h3>

<p>It allows you to <strong>mount a file or folder</strong> from the host machine to the virtual machine.<br><br>
This mechanism is especially useful for <strong>sharing provisioning scripts, configuration files, or datasets</strong> between the host and the VM without having to manually copy them each time.</p>

<p>For example, to copy a file to the host machine, the syntax is as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Vagrant</span><span class="p">.</span><span class="nf">configure</span><span class="p">(</span><span class="s2">"2"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">config</span><span class="o">|</span>
  <span class="c1"># ... others configuration</span>

  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">provision</span> <span class="s2">"file"</span><span class="p">,</span> <span class="ss">source: </span><span class="s2">"~/path/to/host/folder"</span><span class="p">,</span> <span class="ss">destination: </span><span class="s2">"$HOME/remote/newfolder"</span>
<span class="k">end</span>
</code></pre>

</div>



<p>This copy is one-time and does not synchronize the file state.<br><br>
To synchronize the file state between the host and the VM, the configuration should be as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Vagrant</span><span class="p">.</span><span class="nf">configure</span><span class="p">(</span><span class="s2">"2"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">config</span><span class="o">|</span>
  <span class="c1"># ... others configuration</span>

  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">synced_folder</span> <span class="s2">"src/"</span><span class="p">,</span> <span class="s2">"/srv/website"</span>
<span class="k">end</span>
</code></pre>

</div>



<p>For more information about synchronization, please consult the official documentation <a href="https://developer.hashicorp.com/vagrant/docs/synced-folders/basic_usage" rel="noopener noreferrer">here</a>.</p>

<h3>
  
  
  3. Ansible Provisioner
</h3>

<p>Unlike traditional shell scripts, the Ansible provisioner allows you to <strong>describe the desired state of a system in a declarative way</strong>.<br><br>
This means you write "playbooks" (YAML files) that define what you want to achieve (e.g., PostgreSQL installed, a user created, a service started), and Ansible takes care of making the necessary changes to reach that state, <strong>without unnecessarily repeating tasks that have already been done</strong>.</p>

<p><strong>Why use Ansible with vagrant?</strong></p>

<ul>
<li><p>✅ More readable, structured, and reusable code.</p></li>
<li><p>🔁 Idempotence: running a playbook again doesn’t change anything if everything is already in place.</p></li>
<li><p>⚙️ Easy to maintain, especially for complex or multi-machine environments.</p></li>
</ul>

<p>Vagrant can run Ansible as <strong>either a local or remote provisioner</strong>, depending on whether Ansible is installed on the host or on the VM itself.</p>

<p>Here is a minimal example in a Vagrantfile:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">provision</span> <span class="s2">"ansible"</span> <span class="k">do</span> <span class="o">|</span><span class="n">ansible</span><span class="o">|</span>
  <span class="n">ansible</span><span class="p">.</span><span class="nf">playbook</span> <span class="o">=</span> <span class="s2">"playbook.yml"</span>
  <span class="n">ansible</span><span class="p">.</span><span class="nf">inventory_path</span> <span class="o">=</span> <span class="s2">"inventory.ini"</span>
<span class="k">end</span>
</code></pre>

</div>



<h2>
  
  
  🧪 Practical: Building the PostgreSQL + Barman Lab
</h2>

<p>It’s time to get hands-on and build our test environment.<br><br>
The goal of this phase is to <strong>create two virtual machines</strong> using Vagrant:</p>

<ul>
<li><p>🐘 A first VM with <strong>PostgreSQL</strong> installed; it will simulate our source database,</p></li>
<li><p>📦 A second VM with <strong>Barman</strong> installed; it will represent an administrator workstation or a control machine from which we can manage our backups.</p></li>
</ul>

<p>We will automate the installation of each component using <strong>Bash provisioning scripts</strong>.The <code>Vagrantfile</code> will define the architecture of our lab and execute the scripts automatically when the machines start.</p>

<p>Let's start by creating the Vagrantfile and the scripts needed to install Postgres and Barman.</p>

<p>Vagrantfile<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># Vagrantfile</span>
<span class="no">Vagrant</span><span class="p">.</span><span class="nf">configure</span><span class="p">(</span><span class="s2">"2"</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">config</span><span class="o">|</span>
  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">define</span> <span class="s2">"vm1_postgres"</span> <span class="k">do</span> <span class="o">|</span><span class="n">vm1</span><span class="o">|</span>
    <span class="n">vm1</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">box</span> <span class="o">=</span> <span class="s2">"ubuntu/jammy64"</span>
    <span class="n">vm1</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">hostname</span> <span class="o">=</span> <span class="s2">"postgres"</span>
    <span class="n">vm1</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">network</span> <span class="s2">"private_network"</span><span class="p">,</span> <span class="ss">ip: </span><span class="s2">"192.168.56.10"</span>
    <span class="n">vm1</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">provision</span> <span class="s2">"shell"</span><span class="p">,</span> <span class="ss">path: </span><span class="s2">"scripts/setup_postgres.sh"</span>
  <span class="k">end</span>

  <span class="n">config</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">define</span> <span class="s2">"vm2_vagrant"</span> <span class="k">do</span> <span class="o">|</span><span class="n">vm2</span><span class="o">|</span>
    <span class="n">vm2</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">box</span> <span class="o">=</span> <span class="s2">"ubuntu/jammy64"</span>
    <span class="n">vm2</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">hostname</span> <span class="o">=</span> <span class="s2">"vagrant"</span>
    <span class="n">vm2</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">network</span> <span class="s2">"private_network"</span><span class="p">,</span> <span class="ss">ip: </span><span class="s2">"192.168.56.11"</span>
    <span class="n">vm2</span><span class="p">.</span><span class="nf">vm</span><span class="p">.</span><span class="nf">provision</span> <span class="s2">"shell"</span><span class="p">,</span> <span class="ss">path: </span><span class="s2">"scripts/setup_vagrant.sh"</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<h3>
  
  
  <code>scripts/setup_postgres.sh</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-e</span>

<span class="nb">echo</span> <span class="s2">"[INFO] Installing PostgreSQL..."</span>

<span class="nb">sudo </span>apt-get update
<span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> postgresql postgresql-contrib

psql <span class="nt">--version</span> <span class="o">&amp;&amp;</span> <span class="nb">echo</span> <span class="s2">"[OK] PostgreSQL installed"</span>
</code></pre>

</div>



<h3>
  
  
  <code>scripts/setup_barman.sh</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-e</span>

<span class="nb">echo</span> <span class="s2">"[INFO] Installing Barman..."</span>

<span class="nb">sudo </span>apt-get update
<span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> barman

barman <span class="nt">--version</span> <span class="o">&amp;&amp;</span> <span class="nb">echo</span> <span class="s2">"[OK] Barman installed"</span>
</code></pre>

</div>



<h2>
  
  
  🧩 Conclusion
</h2>

<p>We have just discovered Vagrant and, in practice, laid the foundation for a test environment by setting up two virtual machines: one with <strong>PostgreSQL</strong> installed, and the other with <strong>Barman</strong>, our backup tool in the incremental backups series.</p>

<p>Thanks to <strong>Vagrant</strong> and provisioning scripts, we have automated the deployment of this environment, allowing us to easily reproduce our tests, save time, and avoid manual errors.</p>

<p>What we set up here is not limited to the context of Postgres and Barman: this type of lab can serve as a <strong>generic base for all kinds of DevOps practices</strong>, system experiments, or technical tests in isolated environments.</p>

<blockquote>
<p>🧰 A simple, reproducible, and extensible foundation — ideal for learning, testing, breaking… and starting over!</p>
</blockquote>

<p>See you soon in the next post on this blog.</p>

