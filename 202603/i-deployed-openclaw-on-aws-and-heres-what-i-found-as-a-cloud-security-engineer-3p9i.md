---
Title: I Deployed OpenClaw on AWS and Here's What I Found as a Cloud Security Engineer
Description: 
Author: Gerardo Castro Arica
Date: 2026-03-05T21:32:03.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 1 of a series:</strong> Secure setup, real findings, and attack surface analysis of an autonomous AI agent on AWS Lightsail.</p>
</blockquote>

<p>AWS just announced the general availability of <strong>OpenClaw on Amazon Lightsail</strong> — an open-source, self-hosted autonomous AI agent that connects to WhatsApp, Telegram, Discord, and executes tasks independently: running code, managing files, browsing the web.</p>

<p>The community is fired up testing it. So did I — but with a <strong>Cloud Security Engineer</strong> hat on.</p>

<p>This post isn't about how to use OpenClaw as an assistant. It's about <strong>what I found while setting it up from a security perspective</strong>, what decisions I made, and why.</p>

<h2>
  
  
  What exactly is OpenClaw?
</h2>

<p>Before talking security, let's align on concepts.</p>

<p>An <strong>LLM</strong> (like Claude or GPT) receives a prompt and returns text. That's it.</p>

<p>An <strong>autonomous agent</strong> is different: it has access to tools (terminal, browser, APIs), decides the order in which to use them, interprets the results, and acts — without you directing each step. The LLM is the "brain," but the agent is the full system that operates in the real world.</p>

<p>OpenClaw is exactly that: an agent running on your server, using an LLM (in this case via <strong>Amazon Bedrock</strong>) as its reasoning engine, capable of executing tasks autonomously through messaging channels.</p>

<p>That autonomy is what makes it powerful. And also what makes it interesting from a security standpoint.</p>

<h2>
  
  
  The Lightsail Setup
</h2>

<p>AWS packaged OpenClaw as a Lightsail blueprint — meaning you can have an instance running in minutes with no manual configuration.</p>

<h3>
  
  
  What I saw in the wizard (and what caught my attention)
</h3>

<p><strong>The SSH keypair:</strong></p>

<p>Lightsail gives you two options: let AWS generate the keypair, or upload your own.</p>

<p>The security recommendation is clear: <strong>generate your own locally</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ssh-keygen <span class="nt">-t</span> ed25519 <span class="nt">-C</span> <span class="s2">"openclaw-sandbox"</span>
</code></pre>

</div>



<p>Why? When AWS generates the keypair, the private key is created on their servers and travels to you for download. That transmission moment is an avoidable risk. If you generate it yourself, the private key <strong>never leaves your machine</strong>.</p>

<p>The analogy: the public key is the padlock you put on the server. The private key is the key only you hold. Anyone can see the padlock, but no one else can open it.</p>

<blockquote>
<p>💡 <strong>Remember:</strong> always run <code>chmod 400</code> on your private key. If it has <code>644</code> permissions, other system users can read it — and the SSH client itself will warn you.</p>
</blockquote>

<p><strong>The firewall (Security Group):</strong></p>

<p>By default, Lightsail opened ports 80, 443, and 22 to <code>0.0.0.0/0</code> — any IP in the world.</p>

<p>For a personal sandbox that's unnecessary. I changed all three rules to restrict them <strong>to my IP only</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl ifconfig.me  <span class="c"># get your public IP</span>
</code></pre>

</div>



<p>Less exposed attack surface = less blast radius if something goes wrong.</p>

<h2>
  
  
  First Finding: Outdated OS
</h2>

<p>When I connected via SSH, the welcome message was straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>44 updates can be applied immediately.
31 of these updates are standard security updates.
</code></pre>

</div>



<p>The AWS blueprint shipped with <strong>31 unpatched security updates</strong>. Including a kernel patch and <code>intel-microcode</code> update.</p>

<p>Why does the kernel matter? Because it's the core of the OS — it controls memory, processes, and permissions. Known kernel vulnerabilities like <strong>Dirty Pipe</strong> or <strong>Spectre/Meltdown</strong> allow privilege escalation or reading memory from other processes.</p>

<p>On a server running Docker containers (like OpenClaw), an unpatched kernel can be exploited for a <strong>container escape</strong> — breaking out of the isolated container and taking full control of the host.</p>

<p>The fix is simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt upgrade <span class="nt">-y</span>
<span class="nb">sudo </span>reboot
</code></pre>

</div>



<blockquote>
<p>🔍 <strong>Finding #1:</strong> OpenClaw blueprint on Lightsail deployed with outdated kernel and system libraries. Any client using it in production without applying patches is exposed to known CVEs.</p>
</blockquote>

<h2>
  
  
  The Most Interesting Part: OpenClaw Security Settings
</h2>

<p>After setup, OpenClaw presents 5 security configurations. This is where things get interesting.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. File &amp; folder protection     → Status: ✓ Protected
2. Browser remote control       → Status: ✓ Disabled
3. Exec host policy             → Status: ~ Not set (defaults to sandbox)
4. Shell command approval       → Status: ~ Unrestricted
5. Access token                 → Status: ~ Never rotated
</code></pre>

</div>



<h3>
  
  
  Setting 3: Exec host policy
</h3>

<p>This controls <strong>where</strong> the agent executes shell commands.</p>

<ul>
<li>
<strong><code>sandbox</code></strong> (default): commands run inside an isolated Docker container.</li>
<li>
<strong><code>gateway</code></strong>: commands run directly on the server OS.</li>
</ul>

<p>The blast radius difference is enormous. With <code>gateway</code>, if the agent is compromised, the attacker has direct access to the filesystem, environment variables, the IAM role — everything.</p>

<h3>
  
  
  Setting 4: Shell command approval
</h3>

<p>This controls <strong>whether the agent asks for permission</strong> before executing a command.</p>

<ul>
<li>
<strong><code>deny</code></strong>: blocks all shell commands.</li>
<li>
<strong><code>allow</code></strong>: executes them freely without asking.</li>
</ul>

<h3>
  
  
  The attack chain you can't ignore
</h3>

<p>Here's the most dangerous scenario, and it's completely possible if you don't configure things properly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>exec host policy: gateway  +  shell command approval: allow
</code></pre>

</div>



<p><strong>Result:</strong> the agent can execute any command directly on the server, without isolation and without asking for permission.</p>

<p>Add a <strong>prompt injection</strong> attack on top of this — where someone inserts malicious instructions into content the agent consumes (an email, a file, a web page) — and the attacker can take control of the server without you noticing.</p>

<p>This chaining of weak configurations is called <strong>attack chaining</strong>, and it's exactly the type of analysis that needs to happen before putting an agent into production.</p>

<blockquote>
<p>🔍 <strong>Finding #2:</strong> The combination of <code>exec host policy: gateway</code> + <code>shell command approval: allow</code> eliminates all agent isolation layers. In production, this represents a critical risk.</p>
</blockquote>

<h3>
  
  
  Setting 5: Gateway Token visible in the dashboard
</h3>

<p>The Gateway Token is the agent's "password" — whoever has it controls OpenClaw completely.</p>

<p>The problem: <strong>it's displayed in plaintext in the dashboard</strong>.</p>

<p>Any screenshot of the dashboard, any screen recording, anyone looking over your shoulder — compromises access to the agent.</p>

<p>AWS recommends rotating it frequently and not hardcoding it in configuration files. But the fact that it's visible by default in the UI is a design consideration worth noting.</p>

<blockquote>
<p>🔍 <strong>Finding #3:</strong> The Gateway Token is displayed in plaintext in the dashboard. Combined with an internet-exposed dashboard, this represents direct credential exposure.</p>
</blockquote>

<h2>
  
  
  IPv6 Enabled by Default
</h2>

<p>A detail that flies under the radar: the blueprint comes with <strong>dual-stack (IPv4 + IPv6)</strong> enabled by default.</p>

<p>The security problem: many firewall rules are written with IPv4 in mind. IPv6 traffic can go unnoticed if you're not reviewing both protocol families in your controls.</p>

<p>If you're not using IPv6, disable it. Unnecessary attack surface.</p>

<h2>
  
  
  What's Installed in This Blueprint?
</h2>

<p>Among the services restarted after <code>apt upgrade</code>, an interesting one appeared:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>systemctl restart apache2.service
</code></pre>

</div>



<p>The OpenClaw dashboard runs on <strong>Apache2</strong>. That means Apache is another attack surface to consider — with its own CVEs and configurations to review.</p>

<p>An attacker familiar with Apache vulnerabilities could attempt to bypass gateway authentication without ever needing the token.</p>

<h2>
  
  
  Findings Summary
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>#</th>
<th>Finding</th>
<th>Severity</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Blueprint deployed with outdated kernel and system libraries (31 pending security updates)</td>
<td>High</td>
</tr>
<tr>
<td>2</td>
<td>
<code>gateway</code> + <code>allow</code> combination eliminates isolation and command approval</td>
<td>Critical</td>
</tr>
<tr>
<td>3</td>
<td>Gateway Token displayed in plaintext in the dashboard</td>
<td>High</td>
</tr>
<tr>
<td>4</td>
<td>IPv6 enabled by default with no option to disable it in the wizard</td>
<td>Medium</td>
</tr>
<tr>
<td>5</td>
<td>Apache2 as web server with no documented hardening</td>
<td>Medium</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Secure Configuration Recommendations
</h2>

<p>If you're deploying OpenClaw in a real environment:</p>

<ol>
<li>
<strong>Generate your keypair locally</strong> — never let the provider generate it for you.</li>
<li>
<strong>Restrict the firewall</strong> to known IPs from the very beginning.</li>
<li>
<strong>Apply patches immediately</strong> after deploy — don't trust that the blueprint is up to date.</li>
<li>
<strong>Keep <code>exec host policy</code> on <code>sandbox</code></strong> — Docker isolation is your first line of defense.</li>
<li>
<strong>Set <code>shell command approval</code> to <code>deny</code></strong> or require explicit approval — human in the loop for irreversible actions.</li>
<li>
<strong>Rotate the Gateway Token</strong> regularly and never expose it in screenshots.</li>
<li>
<strong>Disable IPv6</strong> if you're not using it.</li>
<li>
<strong>Don't expose the dashboard to the internet</strong> — if you need remote access, use a VPN or secure tunnel.</li>
</ol>

<h2>
  
  
  Why This Matters
</h2>

<p>Autonomous AI agents are arriving in production at companies across LATAM. OpenClaw is just the first of many.</p>

<p>Most teams deploying them aren't thinking about attack surface, blast radius from a misconfiguration, or how a prompt injection can chain with a misconfiguration to fully compromise a server.</p>

<p>That gap is exactly where Cloud Security Engineers need to be.</p>

<h2>
  
  
  What's Next
</h2>

<p>In <strong>Part 2</strong> I'll explore the surfaces we didn't touch today: Channels (WhatsApp/Telegram integrations), Agents, Cron Jobs, and how each one expands the system's attack surface.</p>

<p>I'll also run a full threat model using the <strong>OWASP Top 10 for Agentic Applications 2026</strong> and the <strong>AWS Agentic AI Security Scoping Matrix</strong>.</p>

<p>This article is part of the <strong>Road to CloudSec LATAM</strong> series. Original version in Spanish on Hashnode.</p>

<p>Have questions or found something different in your deployment? Drop a comment — I'm building the complete threat model for Part 2.</p>

<h2>
  
  
  About the Author
</h2>

<p>Gerardo Castro is an AWS Security Hero and Cloud Security Engineer focused on LATAM. He believes the best way to learn cloud security is by building real things — not memorizing frameworks. He writes about what he builds, what he finds, and what he learns along the way.</p>

<p>🔗 GitHub: <a href="https://github.com/gerardokaztro" rel="noopener noreferrer">https://github.com/gerardokaztro</a><br>
🔗 LinkedIn: <a href="https://linkedin.com/in/gerardokaztro" rel="noopener noreferrer">https://linkedin.com/in/gerardokaztro</a></p>

