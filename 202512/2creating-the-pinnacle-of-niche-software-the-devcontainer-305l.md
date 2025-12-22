---
Title: (2)Creating the Pinnacle of Niche Software: The devcontainer
Description: 
Author: Theodor Heiselberg
Date: 2025-12-22T21:31:43.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Elm development environment
</h2>

<p>I prefer as ultra stable, shareable and reproducible an environment as possible so my preferred approach is using a <a href="https://code.visualstudio.com/docs/devcontainers/containers" rel="noopener noreferrer">devcontainer</a>.</p>

<p>Until recently I saw the .devcontainer as a intrinsically part of my project only containing the one true development container setup. Devs could use <a href="https://code.visualstudio.com/docs/devcontainers/containers#_personalizing-with-dotfile-repositories" rel="noopener noreferrer">dotfiles</a> to bring custom configs and setup that way.</p>

<p>However my perspective has changed recently and now I just add a folder with my own name to the <code>.devcontainer</code> folder and leave it up to each dev how they prefer to roll.</p>

<p>Here I'll share my current favorite setup when developing for Elm in the <a href="https://dev.to/sukkergris/creating-the-pinnacle-of-niche-software-consume-squidex-headless-cms-apis-in-elm-59i7">previously described</a> tech stack. And it dose involve quite a few moving parts - so hold on!</p>

<h2>
  
  
  Building the base container
</h2>

<p>The project for the base contaner - tailored for my needs - can be found here: <a href="https://github.com/sukkergris/elm-development-environment-builder" rel="noopener noreferrer">elm-development-environment-builder</a><br>
Don't get fooled by the name - it's tailored for my needs only - but do get inspired. :)</p>
<h2>
  
  
  The devcontainer setup
</h2>
<h3>
  
  
  Folder structure
</h3>

<p>I'm considering adding <code>theodor</code> to <code>.gitignore</code> but I'm undecided right now.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>project-root/
├── .devcontainer/
│   ├── vanilla/                 # Basic example
│   └── theodor/                 # Personal environment config
│       ├── ssl/
│       │   ├── squidex.crt
│       │   └── squidex.key
│       ├── devcontainer.json
│       ├── docker-compose.yml
│       ├── Dockerfile
│       ├── Dockerfile.omnia
│       └── nginx.conf
├── backend/
├── backup/
├── build/
├── Documentation/
├── dotnet/
└── elm/
</code></pre>

</div>



<h3>
  
  
  Comments
</h3>

<p>This setup fairly advanced!</p>

<ol>
<li>Using custom domain locally - not localhost:port!</li>
<li>Using nginx - production like</li>
<li>Setting environment variables using nginx and <a href="https://github.com/sukkergris/site-config-loader" rel="noopener noreferrer">site-config-loader</a>!</li>
</ol>

<p>devcontainer.json<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Dotnet 9/10 and Elm Dev Container (Debian + Compose)"</span><span class="p">,</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">---</span><span class="w"> </span><span class="err">Use</span><span class="w"> </span><span class="err">Docker</span><span class="w"> </span><span class="err">Compose</span><span class="w"> </span><span class="err">configuration</span><span class="w"> </span><span class="err">---</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">Point</span><span class="w"> </span><span class="err">to</span><span class="w"> </span><span class="err">your</span><span class="w"> </span><span class="err">docker-compose</span><span class="w"> </span><span class="err">file</span><span class="w"> </span><span class="err">(relative</span><span class="w"> </span><span class="err">to</span><span class="w"> </span><span class="err">.devcontainer</span><span class="w"> </span><span class="err">folder)</span><span class="w">
  </span><span class="nl">"dockerComposeFile"</span><span class="p">:</span><span class="w"> </span><span class="s2">"docker-compose.yml"</span><span class="p">,</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">The</span><span class="w"> </span><span class="err">name</span><span class="w"> </span><span class="err">of</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">service</span><span class="w"> </span><span class="err">defined</span><span class="w"> </span><span class="err">in</span><span class="w"> </span><span class="err">docker-compose.yml</span><span class="w"> </span><span class="err">that</span><span class="w"> </span><span class="err">VS</span><span class="w"> </span><span class="err">Code</span><span class="w"> </span><span class="err">should</span><span class="w"> </span><span class="err">attach</span><span class="w"> </span><span class="err">to</span><span class="w">
  </span><span class="nl">"service"</span><span class="p">:</span><span class="w"> </span><span class="s2">"dev"</span><span class="p">,</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">The</span><span class="w"> </span><span class="err">path</span><span class="w"> </span><span class="err">inside</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">container</span><span class="w"> </span><span class="err">where</span><span class="w"> </span><span class="err">your</span><span class="w"> </span><span class="err">project</span><span class="w"> </span><span class="err">files</span><span class="w"> </span><span class="err">are</span><span class="w"> </span><span class="err">mounted</span><span class="w"> </span><span class="err">by</span><span class="w"> </span><span class="err">docker-compose.yml</span><span class="w">
  </span><span class="nl">"workspaceFolder"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/workspace"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"mounts"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"source=ktk-elm-devcontainer,target=/home/container-user/.elm,type=volume"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"customizations"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"vscode"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"settings"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="err">//</span><span class="w"> </span><span class="err">Set</span><span class="w"> </span><span class="err">bash</span><span class="w"> </span><span class="err">as</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">default</span><span class="w"> </span><span class="err">terminal</span><span class="w"> </span><span class="err">(installed</span><span class="w"> </span><span class="err">in</span><span class="w"> </span><span class="err">Dockerfile)</span><span class="w">
        </span><span class="nl">"terminal.integrated.defaultProfile.linux"</span><span class="p">:</span><span class="w"> </span><span class="s2">"zsh"</span><span class="w">
        </span><span class="err">//</span><span class="w"> </span><span class="err">Optional</span><span class="w"> </span><span class="err">Neovim</span><span class="w"> </span><span class="err">integration</span><span class="w"> </span><span class="err">path:</span><span class="w">
        </span><span class="err">//</span><span class="w"> </span><span class="nl">"vscode-neovim.neovimExecutablePaths.linux"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/usr/bin/nvim"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"extensions"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"ms-vscode-remote.remote-containers"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"Elmtooling.elm-ls-vscode"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"ms-dotnettools.csharp"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"william-voyek.vscode-nginx"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"vscodevim.vim"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"ms-dotnettools.csdevkit"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"EditorConfig.EditorConfig"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"humao.rest-client"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"esbenp.prettier-vscode"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"DotJoshJohnson.xml"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"streetsidesoftware.code-spell-checker"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"streetsidesoftware.code-spell-checker-danish"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"bradlc.vscode-tailwindcss"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"kamikillerto.vscode-colorize"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"Ionide.Ionide-fsharp"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"ms-azuretools.vscode-containers"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"jebbs.plantuml"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"task.vscode-task"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"ecmel.vscode-html-css"</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">The</span><span class="w"> </span><span class="err">user</span><span class="w"> </span><span class="err">VS</span><span class="w"> </span><span class="err">Code</span><span class="w"> </span><span class="err">should</span><span class="w"> </span><span class="err">run</span><span class="w"> </span><span class="err">as</span><span class="w"> </span><span class="err">inside</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">service</span><span class="w"> </span><span class="err">container.</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">Should</span><span class="w"> </span><span class="err">match</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">user</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">service</span><span class="w"> </span><span class="err">runs</span><span class="w"> </span><span class="err">as</span><span class="w"> </span><span class="err">(container-user</span><span class="w"> </span><span class="err">in</span><span class="w"> </span><span class="err">this</span><span class="w"> </span><span class="err">case).</span><span class="w">
  </span><span class="nl">"remoteUser"</span><span class="p">:</span><span class="w"> </span><span class="s2">"container-user"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"portsAttributes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"3033"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"label"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Elm"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"5130"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"label"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Backend"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"5140"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"label"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Gateway"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"8314"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"label"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Nginx"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"8376"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"label"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Squidex"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"9876"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"label"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Elm"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>docker-compose.yml<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="c1"># Define our main development service, let's call it 'dev'</span>
  <span class="na">dev</span><span class="pi">:</span>
    <span class="c1"># Instructions on how to build the image for this service</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="c1"># Use the current directory (.devcontainer) as the build context</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">.</span>
      <span class="c1"># Specify the Dockerfile within the build context</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>

    <span class="c1"># Define volumes to mount</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="c1"># Mount the entire project directory (one level up from .devcontainer)</span>
      <span class="c1"># into the /workspace directory inside the container.</span>
      <span class="c1"># 'cached' optimizes mount performance on macOS/Windows.</span>
      <span class="pi">-</span> <span class="s">../..:/workspace:cached</span>
      <span class="pi">-</span> <span class="s">ktk-elm-devcontainer:/home/container-user/.elm</span>
      <span class="c1"># --- Alternative for Neovim config ---</span>
      <span class="c1"># Instead of COPY in Dockerfile, you could mount it here:</span>
      <span class="c1"># Ensure the source path '../.nvim' exists in your project root.</span>
      <span class="c1"># - ../.nvim:/root/.config/nvim:cached</span>
         <span class="c1"># --- ADD THIS LINE ---</span>
      <span class="c1"># Mount the project's nvim config into the root user's config dir</span>
      <span class="c1"># Source: ./.config/nvim (relative to this docker-compose.yml file)</span>
      <span class="c1"># Target: /root/.config/nvim (where nvim looks when run as root)</span>
      <span class="c1"># - ./.config/nvim:/root/.config/nvim:cached</span>
    <span class="c1"># Command to run when the container starts.</span>
    <span class="c1"># 'sleep infinity' keeps the container running indefinitely so VS Code can attach.</span>
    <span class="na">command</span><span class="pi">:</span> <span class="s">sleep infinity</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">internal</span>

    <span class="c1"># Environment variables can be defined here if needed</span>
    <span class="c1"># environment:</span>
    <span class="c1">#   - DATABASE_URL=...</span>

    <span class="c1"># The service will run as 'root' because that's the last USER in the Dockerfile.</span>
    <span class="c1"># You could explicitly set 'user: root' here if desired.</span>
  <span class="na">nginx</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">nginx:alpine</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">ktk_nginx</span>
    <span class="na">ports</span><span class="pi">:</span>
     <span class="pi">-</span> <span class="s2">"</span><span class="s">80:80"</span>
     <span class="pi">-</span> <span class="s2">"</span><span class="s">443:443"</span>
     <span class="pi">-</span> <span class="s2">"</span><span class="s">8314:80"</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">internal</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="c1"># - ../..:/workspace</span>
      <span class="pi">-</span> <span class="s">./nginx.conf:/etc/nginx/nginx.conf:ro</span>
      <span class="pi">-</span> <span class="s">./ssl:/etc/nginx/ssl:ro</span>
    <span class="na">extra_hosts</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">host.docker.internal:host-gateway"</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">dev</span>
      <span class="pi">-</span> <span class="s">squidex</span>
  <span class="na">mongo</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s2">"</span><span class="s">mongo:6"</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">ktk_mongo_data:/data/db</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">internal</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>
  <span class="na">squidex</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s2">"</span><span class="s">squidex/squidex:7"</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8376:5000"</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">URLS__BASEURL=https://squidex.ktk.dk</span>
      <span class="pi">-</span> <span class="s">IDENTITY__ALLOWHTTPSCHEME=false</span>
      <span class="pi">-</span> <span class="s">EVENTSTORE__MONGODB__CONFIGURATION=mongodb://mongo</span>
      <span class="pi">-</span> <span class="s">STORE__MONGODB__CONFIGURATION=mongodb://mongo</span>
      <span class="pi">-</span> <span class="s">IDENTITY__ADMINEMAIL=sukkerfrit@gmail.com</span>
      <span class="pi">-</span> <span class="s">IDENTITY__ADMINPASSWORD=0hSoS3cret!</span>
      <span class="pi">-</span> <span class="s">ASPNETCORE_URLS=http://+:5000</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">curl"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">-f"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">http://localhost:5000/healthz"</span><span class="pi">]</span>
      <span class="na">start_period</span><span class="pi">:</span> <span class="s">60s</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">mongo</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">ktk_squidex_assets:/app/Assets</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">internal</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">ktk-elm-devcontainer</span><span class="pi">:</span>
  <span class="na">ktk_squidex_assets</span><span class="pi">:</span>
  <span class="na">ktk_mongo_data</span><span class="pi">:</span>

<span class="na">networks</span><span class="pi">:</span>
  <span class="na">internal</span><span class="pi">:</span>
    <span class="na">driver</span><span class="pi">:</span> <span class="s">bridge</span>
</code></pre>

</div>



<p>Dockerfile<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="s"> isuperman/elm-devcontainer-foundation:0.1.7</span>
<span class="c"># Project specifics could be added here</span>
</code></pre>

</div>



<p>Here is the config for running extremely production like!<br>
I'm even using a custom domain for local development.<br>
This setup make it possible to test routing, a <a href="https://blog.nginx.org/blog/rate-limiting-nginx" rel="noopener noreferrer">tarpit</a> and the likes.<br>
nginx.conf<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight conf"><code><span class="n">worker_processes</span> <span class="m">1</span>;
<span class="n">events</span> {
    <span class="n">worker_connections</span> <span class="m">1024</span>;
}

<span class="n">http</span> {
    <span class="n">include</span>       /<span class="n">etc</span>/<span class="n">nginx</span>/<span class="n">mime</span>.<span class="n">types</span>;
    <span class="n">default_type</span>  <span class="n">application</span>/<span class="n">octet</span>-<span class="n">stream</span>;
    <span class="n">sendfile</span>        <span class="n">on</span>;
    <span class="n">keepalive_timeout</span>  <span class="m">65</span>;

    <span class="c"># Define resolver to handle host.docker.internal
</span>    <span class="n">resolver</span> <span class="m">127</span>.<span class="m">0</span>.<span class="m">0</span>.<span class="m">11</span>;

    <span class="n">server</span> {
        <span class="n">listen</span> <span class="m">80</span>;
        <span class="n">server_name</span> <span class="n">localhost</span> <span class="n">ktk</span>.<span class="n">dk</span>;

        <span class="n">location</span> /<span class="n">api</span>/ {
            <span class="n">set</span> $<span class="n">backend_upstream</span> <span class="n">host</span>.<span class="n">docker</span>.<span class="n">internal</span>:<span class="m">5130</span>;
            <span class="n">rewrite</span> ^/<span class="n">api</span>/(.*)$ /$<span class="m">1</span> <span class="n">break</span>;
            <span class="n">proxy_pass</span> <span class="n">http</span>://$<span class="n">backend_upstream</span>;
            <span class="n">proxy_http_version</span> <span class="m">1</span>.<span class="m">1</span>;
            <span class="n">proxy_set_header</span> <span class="n">Upgrade</span> $<span class="n">http_upgrade</span>;
            <span class="n">proxy_set_header</span> <span class="n">Connection</span> <span class="s1">'upgrade'</span>;
            <span class="n">proxy_set_header</span> <span class="n">Host</span> $<span class="n">host</span>;
            <span class="n">proxy_cache_bypass</span> $<span class="n">http_upgrade</span>;

        }

        <span class="n">location</span> /<span class="n">cms</span>/ {
          <span class="n">set</span> $<span class="n">backend_upstream</span> <span class="n">host</span>.<span class="n">docker</span>.<span class="n">internal</span>:<span class="m">5140</span>;
          <span class="n">rewrite</span> ^/<span class="n">cms</span>/(.*)$ /$<span class="m">1</span> <span class="n">break</span>;
          <span class="n">proxy_pass</span> <span class="n">http</span>://$<span class="n">backend_upstream</span>;
          <span class="n">proxy_http_version</span> <span class="m">1</span>.<span class="m">1</span>;
          <span class="n">proxy_set_header</span> <span class="n">Upgrade</span> $<span class="n">http_upgrade</span>;
          <span class="n">proxy_set_header</span> <span class="n">Connection</span> <span class="s1">'upgrade'</span>;
          <span class="n">proxy_set_header</span> <span class="n">Host</span> $<span class="n">host</span>;
          <span class="n">proxy_cache_bypass</span> $<span class="n">http_upgrade</span>;
        }
        <span class="c"># Proxy frontend requests to Elm dev server
</span>        <span class="n">location</span> / {
            <span class="n">set</span> $<span class="n">frontend_upstream</span> <span class="n">host</span>.<span class="n">docker</span>.<span class="n">internal</span>:<span class="m">3033</span>;
            <span class="n">proxy_pass</span> <span class="n">http</span>://$<span class="n">frontend_upstream</span>;
            <span class="n">proxy_http_version</span> <span class="m">1</span>.<span class="m">1</span>;
            <span class="n">proxy_set_header</span> <span class="n">Upgrade</span> $<span class="n">http_upgrade</span>;
            <span class="n">proxy_set_header</span> <span class="n">Connection</span> <span class="s1">'upgrade'</span>;
            <span class="n">proxy_set_header</span> <span class="n">Host</span> $<span class="n">host</span>;
            <span class="n">proxy_cache_bypass</span> $<span class="n">http_upgrade</span>;
            <span class="n">sub_filter</span> <span class="s1">'&lt;/head&gt;'</span> <span class="s1">'&lt;meta name="environment-name" content="local" /&gt;&lt;/head&gt;'</span>;
            <span class="n">sub_filter_once</span> <span class="n">on</span>;
            <span class="c"># Only apply sub_filter to index.html
</span>            <span class="c"># sub_filter_types text/html;
</span>        }

        <span class="n">location</span> = /<span class="n">robots</span>.<span class="n">txt</span> {
            <span class="n">set</span> $<span class="n">frontend_upstream</span> <span class="n">host</span>.<span class="n">docker</span>.<span class="n">internal</span>:<span class="m">3033</span>;
            <span class="n">proxy_pass</span> <span class="n">http</span>://$<span class="n">frontend_upstream</span>/<span class="n">robots</span>.<span class="n">txt</span>;
            <span class="n">proxy_http_version</span> <span class="m">1</span>.<span class="m">1</span>;
            <span class="n">proxy_set_header</span> <span class="n">Host</span> $<span class="n">host</span>;
        }

        <span class="n">location</span> = /<span class="n">humans</span>.<span class="n">txt</span> {
            <span class="n">set</span> $<span class="n">frontend_upstream</span> <span class="n">host</span>.<span class="n">docker</span>.<span class="n">internal</span>:<span class="m">3033</span>;
            <span class="n">proxy_pass</span> <span class="n">http</span>://$<span class="n">frontend_upstream</span>/<span class="n">humans</span>.<span class="n">txt</span>;
            <span class="n">proxy_http_version</span> <span class="m">1</span>.<span class="m">1</span>;
            <span class="n">proxy_set_header</span> <span class="n">Host</span> $<span class="n">host</span>;
        }
    }
    <span class="n">server</span> {
        <span class="n">listen</span> <span class="m">80</span>;
        <span class="n">server_name</span> <span class="n">squidex</span>.<span class="n">ktk</span>.<span class="n">dk</span>;
        <span class="n">return</span> <span class="m">301</span> <span class="n">https</span>://$<span class="n">host</span>$<span class="n">request_uri</span>;
    }

    <span class="n">server</span> {
        <span class="n">listen</span> <span class="m">443</span> <span class="n">ssl</span>;
        <span class="n">server_name</span> <span class="n">squidex</span>.<span class="n">ktk</span>.<span class="n">dk</span> <span class="n">www</span>.<span class="n">squidex</span>.<span class="n">ktk</span>.<span class="n">dk</span>;

        <span class="n">ssl_certificate</span> /<span class="n">etc</span>/<span class="n">nginx</span>/<span class="n">ssl</span>/<span class="n">squidex</span>.<span class="n">crt</span>;
        <span class="n">ssl_certificate_key</span> /<span class="n">etc</span>/<span class="n">nginx</span>/<span class="n">ssl</span>/<span class="n">squidex</span>.<span class="n">key</span>;
        <span class="n">ssl_protocols</span> <span class="n">TLSv1</span>.<span class="m">2</span> <span class="n">TLSv1</span>.<span class="m">3</span>;

        <span class="n">location</span> / {
            <span class="n">proxy_pass</span> <span class="n">http</span>://<span class="n">squidex</span>:<span class="m">5000</span>;
            <span class="n">proxy_http_version</span> <span class="m">1</span>.<span class="m">1</span>;
            <span class="n">proxy_set_header</span> <span class="n">Host</span> $<span class="n">host</span>;
            <span class="n">proxy_set_header</span> <span class="n">X</span>-<span class="n">Forwarded</span>-<span class="n">For</span> $<span class="n">proxy_add_x_forwarded_for</span>;
            <span class="n">proxy_set_header</span> <span class="n">X</span>-<span class="n">Forwarded</span>-<span class="n">Proto</span> $<span class="n">scheme</span>;
            <span class="n">proxy_set_header</span> <span class="n">X</span>-<span class="n">Forwarded</span>-<span class="n">Host</span> $<span class="n">host</span>;

            <span class="c"># WebSocket support
</span>            <span class="n">proxy_set_header</span> <span class="n">Upgrade</span> $<span class="n">http_upgrade</span>;
            <span class="n">proxy_set_header</span> <span class="n">Connection</span> <span class="s1">'upgrade'</span>;

            <span class="c"># Authentication flows
</span>            <span class="n">proxy_cookie_path</span> / /;
        }
    }
}

</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>Here we see an example of how one could use a devcontainer to set up a development environment. Using this approach has definitely helped me a lot. This is very production-like, and by using most of these settings for a local <code>docker-compose.yml</code>, I am extremely confident when pushing my containers to production. To emphasize, I rarely have bugs in production due to this setup.</p>

<p>Next up we will focus on how to setup <a href="https://github.com/ryan-haskell/vite-plugin-elm-watch" rel="noopener noreferrer">vite-plugin-elm-watch</a> which will require some tingling in order to run inside a devcontainer.</p>

