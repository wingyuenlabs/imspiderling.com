---
Title: Why I Love Using Nix for All My Projects
Description: 
Author: Cristian Angulo
Date: 2026-01-10T21:18:46.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl8r4vfby4000p6g71hr2.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl8r4vfby4000p6g71hr2.jpeg" alt="Nix Development" width="800" height="446"></a></p>

<p>"It works on my machine" is a phrase that has haunted software development for decades. Whether it's a teammate having a different version of Python, a CI server missing a specific C library, or your global Node.js version conflicting with a legacy project, environment drift is a silent productivity killer.</p>

<p>I resolved this by moving my entire development workflow to <strong>Nix Flakes</strong>. </p>

<p>Nix guarantees that if a project works today, it will work exactly the same way six months from now, or on a completely different machine, without manual installation steps.</p>




<h3>
  
  
  Why Nix?
</h3>

<p>In modern software development, we don't just write code; we manage a fleet of tools. Nix provides a declarative way to handle:</p>

<ul>
<li>
<strong>Isolation:</strong> No more polluting your global <code>/usr/local/bin</code>. Every tool stays within the project.</li>
<li>
<strong>Reproducibility:</strong> Every dependency is pinned. If I use Python 3.12.1, everyone on the team uses 3.12.1.</li>
<li>
<strong>Automation:</strong> Using the <code>shellHook</code>, we can automate the startup of services, credential helpers, and environment variables.</li>
</ul>

<h3>
  
  
  The Project Orchestrator: <code>flake.nix</code>
</h3>

<p>I use a <code>flake.nix</code> in every repository. This specific configuration handles a complex stack: Python (managed by <strong>uv</strong>), AWS (with ECR automation), Docker, and various security tools.</p>

<h3>
  
  
  flake.nix
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight nix"><code><span class="p">{</span>
  <span class="nv">description</span> <span class="o">=</span> <span class="s2">"DevShell con Terraform, Docker, Python y ECR login"</span><span class="p">;</span>

  <span class="nv">inputs</span> <span class="o">=</span> <span class="p">{</span>
    <span class="nv">nixpkgs</span><span class="o">.</span><span class="nv">url</span> <span class="o">=</span> <span class="s2">"github:NixOS/nixpkgs/nixos-unstable"</span><span class="p">;</span>
    <span class="nv">flake-utils</span><span class="o">.</span><span class="nv">url</span> <span class="o">=</span> <span class="s2">"github:numtide/flake-utils"</span><span class="p">;</span>
  <span class="p">};</span>

  <span class="nv">outputs</span> <span class="o">=</span> <span class="p">{</span> <span class="nv">self</span><span class="p">,</span> <span class="nv">nixpkgs</span><span class="p">,</span> <span class="nv">flake-utils</span> <span class="p">}:</span>
    <span class="nv">flake-utils</span><span class="o">.</span><span class="nv">lib</span><span class="o">.</span><span class="nv">eachDefaultSystem</span> <span class="p">(</span><span class="nv">system</span><span class="p">:</span>
      <span class="kd">let</span>
        <span class="nv">pkgs</span> <span class="o">=</span> <span class="kr">import</span> <span class="nv">nixpkgs</span> <span class="p">{</span> <span class="kn">inherit</span> <span class="nv">system</span><span class="p">;</span> <span class="p">};</span>

        <span class="nv">pythonEnv</span> <span class="o">=</span> <span class="nv">pkgs</span><span class="o">.</span><span class="nv">python312</span><span class="o">.</span><span class="nv">withPackages</span> <span class="p">(</span><span class="nv">ps</span><span class="p">:</span> <span class="kn">with</span> <span class="nv">ps</span><span class="p">;</span> <span class="p">[</span>
          <span class="nv">pip</span>
        <span class="p">]);</span>

        <span class="nv">commonDeps</span> <span class="o">=</span> <span class="kn">with</span> <span class="nv">pkgs</span><span class="p">;</span> <span class="p">[</span>
          <span class="nv">pythonEnv</span>
          <span class="nv">uv</span>
          <span class="nv">git</span>
          <span class="nv">terraform</span>
          <span class="nv">python312</span>
          <span class="nv">awscli</span>
          <span class="nv">gcc</span>
          <span class="nv">stdenv</span><span class="o">.</span><span class="nv">cc</span><span class="o">.</span><span class="nv">cc</span><span class="o">.</span><span class="nv">lib</span>
          <span class="nv">httpie</span>
          <span class="nv">go</span>
          <span class="nv">go-task</span>
          <span class="nv">docker</span>
          <span class="nv">amazon-ecr-credential-helper</span>
          <span class="nv">jq</span>
          <span class="nv">nodejs_20</span>
          <span class="nv">nodePackages</span><span class="o">.</span><span class="nv">lerna</span>
          <span class="nv">google-chrome</span>
          <span class="nv">subfinder</span>
          <span class="nv">amass</span>
          <span class="nv">imagemagick</span>
          <span class="nv">yarn</span>
        <span class="p">];</span>
      <span class="kn">in</span> <span class="p">{</span>
        <span class="nv">devShells</span><span class="o">.</span><span class="nv">default</span> <span class="o">=</span> <span class="nv">pkgs</span><span class="o">.</span><span class="nv">mkShell</span> <span class="p">{</span>
          <span class="nv">packages</span> <span class="o">=</span> <span class="nv">commonDeps</span><span class="p">;</span>

          <span class="nv">shellHook</span> <span class="o">=</span> <span class="s2">''</span><span class="err">
</span><span class="s2">            # 1. Python &amp; uv Setup</span><span class="err">
</span><span class="s2">            pyenv global system</span><span class="err">
</span><span class="s2">            export pythonEnv=</span><span class="si">${</span><span class="nv">pythonEnv</span><span class="si">}</span><span class="err">
</span><span class="s2">            export PATH=$PATH:</span><span class="si">${</span><span class="nv">pythonEnv</span><span class="si">}</span><span class="s2">/bin</span><span class="err">

</span><span class="s2">            # Ensure uv uses the Nix-provided Python interpreter</span><span class="err">
</span><span class="s2">            export UV_PYTHON=</span><span class="si">${</span><span class="nv">pythonEnv</span><span class="si">}</span><span class="s2">/bin/python</span><span class="err">

</span><span class="s2">            # 2. Service Orchestration</span><span class="err">
</span><span class="s2">            docker compose up --build -d</span><span class="err">
</span><span class="s2">            docker compose ps -a</span><span class="err">
</span><span class="s2">            task dependencies</span><span class="err">

</span><span class="s2">            # 3. AWS &amp; Docker Credential Automation</span><span class="err">
</span><span class="s2">            mkdir -p ~/.docker</span><span class="err">
</span><span class="s2">            echo '{</span><span class="err">
</span><span class="s2">              "credsStore": "ecr-login"</span><span class="err">
</span><span class="s2">            }' &gt; ~/.docker/config.json</span><span class="err">
</span><span class="s2">            echo "Docker config set to use docker-credential-ecr-login"</span><span class="err">
</span><span class="s2">          ''</span><span class="p">;</span>

          <span class="nv">LD_LIBRARY_PATH</span> <span class="o">=</span> <span class="nv">pkgs</span><span class="o">.</span><span class="nv">lib</span><span class="o">.</span><span class="nv">makeLibraryPath</span> <span class="p">[</span>
            <span class="nv">pkgs</span><span class="o">.</span><span class="nv">stdenv</span><span class="o">.</span><span class="nv">cc</span><span class="o">.</span><span class="nv">cc</span><span class="o">.</span><span class="nv">lib</span>
          <span class="p">];</span>
        <span class="p">};</span>
      <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  How it works: A Deep Dive
</h3>

<ol>
<li>The Declarative Environment
Inside commonDeps, I list everything from compilers (gcc) to specialized tools like amazon-ecr-credential-helper. When someone joins the project, they don't need to "install" these manually. Nix handles the fetching and linking into an isolated environment, ensuring that your system remains clean and the project remains portable.</li>
</ol>

<h3>
  
  
  2. The uv Advantage
</h3>

<p>I use uv because of its incredible speed and its seamless integration with existing Python environments. By setting export UV_PYTHON=${pythonEnv}/bin/python in the shellHook, I tell uv to use the exact Python binary managed by Nix. This ensures total consistency between the package manager and OS-level dependencies.</p>

<h3>
  
  
  3. The shellHook Magic
</h3>

<p>This is where Nix transcends being a simple package manager and becomes a project manager:</p>

<p>Automatic Infrastructure: As soon as I enter the shell, docker compose up runs. My databases and local services are ready before I even type my first line of code.</p>

<p>Automated Config: It writes the ~/.docker/config.json automatically. This allows me to push/pull from AWS ECR without ever having to run manual aws ecr get-login-password steps.</p>

<p>Task Execution: Running task dependencies ensures that any sub-requirements (like npm install or go mod download) are checked and verified as soon as the shell opens.</p>

<h3>
  
  
  Entering the Environment
</h3>

<p>To activate this entire setup, I use the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
nix <span class="nt">--extra-experimental-features</span> <span class="s1">'nix-command flakes'</span> develop <span class="nt">--impure</span> <span class="nt">--command</span> zsh
</code></pre>

</div>



<p>--extra-experimental-features 'nix-command flakes': Enables the modern Nix Flake commands required for this setup.</p>

<p>--impure: This is essential because the shellHook needs to interact with the outside world (your home directory for ~/.docker and the system Docker socket).</p>

<p>--command zsh: Drops me directly into my preferred shell with the entire environment already loaded and ready to go.</p>

<h3>
  
  
  Conclusion
</h3>

<p>Nix has fundamentally transformed how I approach project setups. Instead of a long README.md containing 20 manual installation steps that are prone to failure, I provide a single flake.nix that defines the entire universe for the project.</p>

<p>It is faster, safer, and 100% reproducible. If it works for me, it will work for you.</p>

