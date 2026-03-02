---
Title: Self-Hosting Remote VSCode with Cloudflare Tunnel and Authentik SSO
Description: 
Author: Charlie
Date: 2026-03-02T21:47:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>Working remotely on lab projects meant a VPN or SSH keys on every device. code-server fixed that — full VS Code in a browser tab, locked behind Google SSO, running 24/7 on the Mac Mini. Here's how it works.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2samsq8y8xik6vufue79.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2samsq8y8xik6vufue79.png" alt="VS Code running in a browser tab with Projects, vault, and Claude Code chat panel open" width="800" height="688"></a></p>

<h2>
  
  
  What's in the stack
</h2>

<ul>
<li>
<strong><a href="https://github.com/coder/code-server" rel="noopener noreferrer">code-server</a></strong> by <a href="https://coder.com/" rel="noopener noreferrer">Coder</a> — VS Code in the browser, packaged as a Docker image by <a href="https://www.linuxserver.io/" rel="noopener noreferrer">LinuxServer.io</a>
</li>
<li>
<strong><a href="https://www.cloudflare.com/products/tunnel/" rel="noopener noreferrer">Cloudflare Tunnel</a></strong> — outbound-only tunnel, no open ports on the router</li>
<li>
<strong><a href="https://goauthentik.io/" rel="noopener noreferrer">Authentik</a></strong> by <a href="https://github.com/goauthentik/authentik" rel="noopener noreferrer">Authentik Security</a> — self-hosted SSO with Google OAuth2</li>
<li>
<strong><a href="https://nginxproxymanager.com/" rel="noopener noreferrer">Nginx Proxy Manager</a></strong> by <a href="https://github.com/NginxProxyManager/nginx-proxy-manager" rel="noopener noreferrer">jc21</a> — reverse proxy that enforces forward auth on every request</li>
</ul>

<h2>
  
  
  Why not just use SSH?
</h2>

<p>SSH works, but requires a client and keys on every device, and you lose the full editor experience. code-server gives you extensions, an integrated terminal, and Claude Code in any modern browser. Once it's running and proxied, it works on iPad just as well as a laptop.</p>

<h2>
  
  
  How the auth flow works
</h2>

<p>Every request to <code>code.yourdomain.com</code> goes through this chain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Browser → Cloudflare Tunnel Edge → Nginx Proxy Manager → Authentik outpost check
                                                         ↓ (if authenticated)
                                                     code-server
</code></pre>

</div>



<p>Nginx Proxy Manager uses <code>auth_request</code> to check every request against Authentik's embedded outpost. If you're not authenticated, you land on the Authentik login page — in this case, a Google OAuth2 prompt.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9idqmxchmwdow07gho5y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9idqmxchmwdow07gho5y.png" alt="Authentik login — " width="800" height="500"></a></p>

<h2>
  
  
  Deployment
</h2>

<h3>
  
  
  1. The compose file
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">code-server</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">lscr.io/linuxserver/code-server:latest</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">code-server</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">PUID=501</span>
      <span class="pi">-</span> <span class="s">PGID=20</span>
      <span class="pi">-</span> <span class="s">TZ=America/Chicago</span>
      <span class="pi">-</span> <span class="s">PASSWORD=${CODE_SERVER_PASSWORD}</span>
      <span class="pi">-</span> <span class="s">SUDO_PASSWORD=${CODE_SERVER_PASSWORD}</span>
      <span class="pi">-</span> <span class="s">DEFAULT_WORKSPACE=/config/workspace</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">/your/config:/config</span>
      <span class="pi">-</span> <span class="s">/your/projects:/config/workspace/Projects</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">8484:8443</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>
</code></pre>

</div>



<p>Put real credentials in a <code>.env</code> file alongside the compose file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CODE_SERVER_PASSWORD=your-password-here
</code></pre>

</div>



<p>Then <code>chmod 600 .env</code> so only your user can read it.</p>

<p><strong>Important:</strong> always use <code>docker compose up -d</code> — not <code>docker restart</code> — when you change env vars. <code>docker restart</code> reuses the original environment from when the container was first created. <code>docker compose up -d</code> re-reads the compose file and <code>.env</code>.</p>

<h3>
  
  
  2. Nginx Proxy Manager config
</h3>

<p>Create a proxy host for <code>code.yourdomain.com</code>:</p>

<ul>
<li>Forward Hostname: your server's local IP (not <code>localhost</code>)</li>
<li>Forward Port: <code>8484</code>
</li>
<li>
<strong>WebSockets Support: ON</strong> — code-server won't work without this</li>
</ul>

<p><strong>Force SSL must be OFF.</strong> Cloudflare terminates TLS at the edge and sends plain HTTP to NPM. If you enable Force SSL in NPM, it will redirect to HTTPS and get redirected again — an infinite loop. NPM receives HTTP and your browser sees HTTPS because Cloudflare handles the certificate.</p>

<h3>
  
  
  3. Authentik forward auth
</h3>

<p>After NPM creates the proxy host config file, patch it to add the Authentik blocks before the main <code>location /</code> block:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">auth_request</span> <span class="n">/outpost.goauthentik.io/auth/nginx</span><span class="p">;</span>
<span class="k">error_page</span> <span class="mi">401</span> <span class="p">=</span> <span class="s">@goauthentik_proxy_signin</span><span class="p">;</span>
<span class="k">auth_request_set</span> <span class="nv">$auth_cookie</span> <span class="nv">$upstream_http_set_cookie</span><span class="p">;</span>
<span class="k">add_header</span> <span class="s">Set-Cookie</span> <span class="nv">$auth_cookie</span><span class="p">;</span>

<span class="k">location</span> <span class="n">/outpost.goauthentik.io</span> <span class="p">{</span>
    <span class="kn">proxy_pass</span> <span class="s">http://your-server-ip:9010/outpost.goauthentik.io</span><span class="p">;</span>
    <span class="kn">proxy_set_header</span> <span class="s">Host</span> <span class="nv">$http_host</span><span class="p">;</span>
    <span class="kn">proxy_set_header</span> <span class="s">X-Real-IP</span> <span class="nv">$remote_addr</span><span class="p">;</span>
    <span class="kn">proxy_set_header</span> <span class="s">X-Forwarded-For</span> <span class="nv">$proxy_add_x_forwarded_for</span><span class="p">;</span>
    <span class="kn">proxy_set_header</span> <span class="s">X-Forwarded-Proto</span> <span class="nv">$scheme</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">location</span> <span class="s">@goauthentik_proxy_signin</span> <span class="p">{</span>
    <span class="kn">internal</span><span class="p">;</span>
    <span class="kn">return</span> <span class="mi">302</span> <span class="n">/outpost.goauthentik.io/start?rd=https://</span><span class="nv">$http_host$request_uri</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>rd=https://</code> redirect URL is what brings you back to code-server after logging in. Without it, Authentik sends you to its own dashboard instead of where you were going.</p>

<p>Reload nginx after editing the conf: <code>docker exec nginx-proxy-manager nginx -s reload</code></p>

<h2>
  
  
  Three things I got wrong the first time
</h2>

<p><strong>Credentials not updating after editing compose.</strong> Edited the compose file, ran <code>docker restart</code>, and the old password still worked. The fix is always <code>docker compose up -d</code>.</p>

<p><strong>YAML special characters in passwords.</strong> A password containing <code>&amp;</code> broke the compose file — YAML treats <code>&amp;</code> as an anchor. Fix: move credentials to a <code>.env</code> file and reference them as <code>${VAR}</code>.</p>

<p><strong>Force SSL redirect loop.</strong> Enabled Force SSL in NPM because the site "should be HTTPS." Every request looped forever because NPM kept redirecting to HTTPS while Cloudflare kept sending plain HTTP. Remove the Force SSL block — Cloudflare handles TLS end to end.</p>

<h2>
  
  
  Extensions on code-server
</h2>

<p>code-server uses the <a href="https://open-vsx.org/" rel="noopener noreferrer">Open VSX Registry</a>, not the Microsoft marketplace. Most extensions are available, but a few Microsoft-owned ones aren't. The ones I run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">for </span>ext <span class="k">in</span> <span class="se">\</span>
  anthropic.claude-code <span class="se">\</span>
  llvm-vs-code-extensions.lldb-dap <span class="se">\</span>
  mechatroner.rainbow-csv <span class="se">\</span>
  ms-azuretools.vscode-containers <span class="se">\</span>
  ms-python.debugpy <span class="se">\</span>
  ms-python.python <span class="se">\</span>
  swiftlang.swift-vscode <span class="se">\</span>
  tomoki1207.pdf<span class="p">;</span> <span class="k">do
  </span>docker <span class="nb">exec</span> <span class="nt">-u</span> abc code-server /app/code-server/bin/code-server <span class="nt">--install-extension</span> <span class="s2">"</span><span class="nv">$ext</span><span class="s2">"</span>
<span class="k">done</span>
</code></pre>

</div>



<p>Note: <code>github.copilot-chat</code> and <code>ms-python.vscode-pylance</code> are not on Open VSX and can't be installed.</p>

<h2>
  
  
  Authenticating Claude Code in a container
</h2>

<p>The Claude Code extension needs Node.js to run its CLI agent. The LinuxServer.io code-server image doesn't include it, so you need to install it — and make it survive container recreations.</p>

<p>Create a startup script at <code>config/custom-cont-init.d/install-claude.sh</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="k">if</span> <span class="o">!</span> <span class="nb">command</span> <span class="nt">-v</span> node &amp;&gt; /dev/null<span class="p">;</span> <span class="k">then
  </span>apt-get update <span class="nt">-qq</span> <span class="o">&amp;&amp;</span> apt-get <span class="nb">install</span> <span class="nt">-y</span> <span class="nt">-qq</span> nodejs npm <span class="o">&gt;</span> /dev/null 2&gt;&amp;1
<span class="k">fi
if</span> <span class="o">!</span> <span class="nb">command</span> <span class="nt">-v</span> claude &amp;&gt; /dev/null<span class="p">;</span> <span class="k">then
  </span>npm <span class="nb">install</span> <span class="nt">-g</span> @anthropic-ai/claude-code <span class="o">&gt;</span> /dev/null 2&gt;&amp;1
<span class="k">fi</span>
</code></pre>

</div>



<p>LinuxServer.io images run anything in <code>custom-cont-init.d/</code> at startup, so Node and Claude CLI persist across <code>docker compose up -d</code> cycles.</p>

<p><strong>The billing gotcha:</strong> Claude Code supports two authentication methods — OAuth login (uses your Claude Pro/Max subscription) and API key (pay-as-you-go billing against your Anthropic API account). In a headless container, the OAuth browser flow doesn't work because the callback can't reach the container.</p>

<p>The workaround: if you're already authenticated on a desktop machine, your OAuth token lives in the system keychain. On macOS:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>security find-generic-password <span class="nt">-s</span> <span class="s2">"Claude Code-credentials"</span> <span class="nt">-w</span>
</code></pre>

</div>



<p>That returns a JSON blob with an <code>accessToken</code> field — an <code>sk-ant-oat</code> prefixed token. Set that as your <code>ANTHROPIC_API_KEY</code> environment variable in the container's <code>.env</code> file. Despite the name, OAuth tokens route through your subscription, not pay-as-you-go.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ANTHROPIC_API_KEY=sk-ant-oat01-your-token-here
</code></pre>

</div>



<p><strong>Important:</strong> <code>sk-ant-api</code> tokens bill against your API account. <code>sk-ant-oat</code> tokens bill against your subscription. If you generate a key from <code>console.anthropic.com</code>, that's an API key and you'll be charged per token. The OAuth token from the keychain uses your existing plan.</p>

<h2>
  
  
  Giving Claude deeper access without opening new attack surface
</h2>

<p>Once Claude Code is running in the container, the natural next question is: can it actually manage infrastructure? Out of the box, the answer is mostly no. It can read and write files in the workspace, but it can't run Docker commands, push to GitHub, or use any custom skills you've built.</p>

<p>The naive fixes — mount the Docker socket, add SSH keys, install tools with root — each add real risk. Here's what I did instead.</p>

<h3>
  
  
  GitHub CLI without sudo
</h3>

<p>The container doesn't have <code>gh</code> installed and you can't <code>sudo apt install</code> without a password prompt. The workaround is installing the binary directly to a directory you own:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">GH_VERSION</span><span class="o">=</span><span class="si">$(</span>curl <span class="nt">-s</span> https://api.github.com/repos/cli/cli/releases/latest | <span class="nb">grep</span> <span class="s1">'"tag_name"'</span> | <span class="nb">cut</span> <span class="nt">-d</span><span class="s1">'"'</span> <span class="nt">-f4</span> | <span class="nb">sed</span> <span class="s1">'s/v//'</span><span class="si">)</span>
curl <span class="nt">-sL</span> <span class="s2">"https://github.com/cli/cli/releases/download/v</span><span class="k">${</span><span class="nv">GH_VERSION</span><span class="k">}</span><span class="s2">/gh_</span><span class="k">${</span><span class="nv">GH_VERSION</span><span class="k">}</span><span class="s2">_linux_arm64.tar.gz"</span> <span class="se">\</span>
  <span class="nt">-o</span> /tmp/gh.tar.gz
<span class="nb">mkdir</span> <span class="nt">-p</span> /config/bin
<span class="nb">tar</span> <span class="nt">-xz</span> <span class="nt">-f</span> /tmp/gh.tar.gz <span class="nt">-C</span> /tmp
<span class="nb">cp</span> /tmp/gh_<span class="k">${</span><span class="nv">GH_VERSION</span><span class="k">}</span>_linux_arm64/bin/gh /config/bin/gh
<span class="nb">echo</span> <span class="s1">'export PATH="/config/bin:$PATH"'</span> <span class="o">&gt;&gt;</span> /config/.bashrc
</code></pre>

</div>



<p>Since <code>/config</code> is a persistent volume, the binary survives container recreations. Then <code>gh auth login</code> to authenticate — use a fine-grained personal access token scoped to only the repos Claude needs, rather than full OAuth. Smaller blast radius if the container is ever compromised.</p>

<h3>
  
  
  Docker access via Portainer API
</h3>

<p>Mounting the Docker socket into a container is effectively handing out root on the host. Anyone who can talk to the socket can escape the container entirely. Don't do it.</p>

<p>If you're already running Portainer, it exposes a REST API that Claude can call with a token. Same outcome — create containers, inspect services, pull images — but the token is scoped, auditable, and revocable without touching the container.</p>

<p>In Portainer: <strong>Account Settings → Access Tokens → Add access token</strong>. Give it a name like <code>claude-code-server</code>.</p>

<p>Store the token as an environment variable in the container's <code>.env</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>PORTAINER_URL=https://portainer.yourdomain.com
PORTAINER_TOKEN=your-token-here
</code></pre>

</div>



<p>Test from inside the container:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-s</span> <span class="nt">-H</span> <span class="s2">"X-API-Key: </span><span class="nv">$PORTAINER_TOKEN</span><span class="s2">"</span> <span class="nv">$PORTAINER_URL</span>/api/endpoints
</code></pre>

</div>



<p>Claude can now call the Portainer API directly without any elevated host access.</p>

<h3>
  
  
  Skills and agents
</h3>

<p>Custom Claude Code skills (slash commands) live in <code>~/.claude/commands/</code> on whatever machine Claude Code is running on. Inside the container, that resolves to <code>/config/.claude/commands/</code> — and that directory doesn't exist by default.</p>

<p>The fix is a single volume mount in the compose file that points the container's commands directory at the one on the host:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">volumes</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s">/your/config:/config</span>
  <span class="pi">-</span> <span class="s">/your/projects:/config/workspace/Projects</span>
  <span class="pi">-</span> <span class="s">~/.claude/commands:/config/.claude/commands:ro</span>   <span class="c1"># add this</span>
</code></pre>

</div>



<p>The <code>:ro</code> flag mounts it read-only — Claude can use the skills but can't modify them from inside the container. Restart the container and the skills show up in the next session.</p>

<p>Agent definitions stored as vault notes are already accessible since the vault is mounted as a workspace folder. No extra setup needed there.</p>

<h2>
  
  
  End result
</h2>

<p>Authenticated VS Code at <code>https://code.yourdomain.com</code> — works on iPad, iPhone, any laptop. Sessions persist between visits, and extensions, settings, and workspace survive image updates since they live in the config volume.</p>

<p>The Obsidian vault is mounted as a workspace folder too, so notes edited in the browser sync to iCloud immediately.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcb5w241us22nthl01udy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcb5w241us22nthl01udy.png" alt="code-server open on iPad at code.seaynicroute.com" width="800" height="558"></a></p>

