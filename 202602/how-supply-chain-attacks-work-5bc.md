---
Title: How Supply Chain Attacks Work
Description: 
Author: Ronaldo Modesto
Date: 2026-02-19T22:08:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>Acesse a versão em português aqui: <a href="https://dev.to/r9n/como-funciona-supply-chain-attack-ptbr-31i0">Clique Aqui</a></p>

<p>Hi everyone.</p>

<p>Today I want to share a little more knowledge about a type of attack that has become increasingly frequent: the Supply Chain Attack.</p>

<p>Let's see how this works in the npm ecosystem and what we can do to mitigate this risk.<br>
<strong>Remember that I've only used npm as an example here, but this type of attack can occur in other package managers as well!</strong></p>

<p>Here you'll find a tool I developed to mitigate this type of attack. <a href="https://github.com/R9n/safeinstall" rel="noopener noreferrer">Safeinstall</a></p>

<p>If you want to see how the tool works, take a look here <a href="https://dev.to/r9n/safeinstall-um-aliado-no-combate-a-ataques-de-supply-chain-47mh">SafeInstall</a></p>


<h2>
  
  
  Introduction
</h2>

<p>How many times a day do you run <code>npm install</code>? For most JavaScript and Node.js developers, the answer is: many. This seemingly innocent routine — installing a dependency to solve a problem — hides an attack vector increasingly exploited by cybercriminals: the <strong>supply chain attack</strong>.</p>

<p>In this article, we explore what these attacks are, how they work in practice using a real demonstration project, what the consequences would be in production environments, and how you can protect yourself.</p>
<h2>
  
  
  The project used in the article can be found here: <a href="https://github.com/R9n/supply-chain-attack-example" rel="noopener noreferrer">Example Project</a>
</h2>
<h2>
  
  
  What is a Supply Chain Attack?
</h2>

<p>A <strong>supply chain attack</strong> occurs when an attacker compromises a component that is part of the software supply chain — something that developers or systems trust and use without question. In the npm ecosystem, this materializes mainly through:</p>

<ol>
<li>
<strong>Malicious packages</strong> created from scratch to appear legitimate</li>
<li>
<strong>Compromised legitimate packages</strong> (abandoned maintenance, hacked account, typosquatting)</li>
<li>
<strong>Lifecycle scripts</strong> that execute automatically during <code>npm install</code>
</li>
</ol>

<p>The critical point is that <strong>the developer does not need to do anything beyond installing the package</strong>. No need to open a suspicious file, click a link, or run an unknown binary. The simple act of adding a dependency to <code>package.json</code> and running <code>npm install</code> can be enough to compromise the machine, repository, or infrastructure.</p>


<h2>
  
  
  Lifecycle Scripts: The Entry Point
</h2>

<p>npm defines various scripts that run at specific moments in a package's lifecycle:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Script</th>
<th>When it runs</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>preinstall</code></td>
<td>Before the package is installed</td>
</tr>
<tr>
<td><code>install</code></td>
<td>During installation</td>
</tr>
<tr>
<td><code>postinstall</code></td>
<td>
<strong>Immediately after installation</strong> — preferred target for attacks</td>
</tr>
<tr>
<td><code>preuninstall</code></td>
<td>Before uninstalling</td>
</tr>
<tr>
<td><code>postuninstall</code></td>
<td>After uninstalling</td>
</tr>
<tr>
<td><code>prepublish</code></td>
<td>Before publishing to the npm registry</td>
</tr>
</tbody>
</table></div>

<p><strong>Anyone who runs <code>npm install</code> will execute these scripts automatically</strong>, with no clear warning. That is where the danger lies.</p>


<h2>
  
  
  Demonstration Project: Structure and Code
</h2>

<p>To illustrate the attack vector, we built an educational project consisting of:</p>

<ol>
<li>
<strong>"Malicious" package</strong> — a package that appears useful but runs code during installation</li>
<li>
<strong>Victim project</strong> — a project that simply depends on that package</li>
</ol>
<h3>
  
  
  Malicious Package Structure
</h3>

<p>The package's <code>package.json</code> defines the scripts that will be executed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"utilidades-uteis"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Pacote útil que parece legítimo mas executa código no post-install"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"main"</span><span class="p">:</span><span class="w"> </span><span class="s2">"index.js"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"postinstall"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node postinstall.js"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"preinstall"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node preinstall.js"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"keywords"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"utility"</span><span class="p">,</span><span class="w"> </span><span class="s2">"helper"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Atacante Anônimo"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"license"</span><span class="p">:</span><span class="w"> </span><span class="s2">"MIT"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Note that <code>postinstall</code> and <code>preinstall</code> point to Node.js scripts. These scripts run automatically during installation.</p>

<h3>
  
  
  preinstall.js Script
</h3>

<p>This script runs <strong>before</strong> the package is installed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="cm">/**
 * PREINSTALL - Runs BEFORE the package is installed
 * Another phase where malicious code can execute
 */</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="se">\</span><span class="s1">u001b[35m[preinstall]</span><span class="se">\</span><span class="s1">u001b[0m This script runs even before the package is installed!</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p>In a real attack, initial data collection or environment preparation could happen here.</p>

<h3>
  
  
  postinstall.js Script — The Heart of the Attack
</h3>

<p>Here is the script that simulates exfiltration of sensitive data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">fs</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">path</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">os</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">os</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// Simulates what an attacker COULD collect (only shows, does not send)</span>
<span class="kd">const</span> <span class="nx">dadosSensiveis</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">executadoEm</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
  <span class="na">usuario</span><span class="p">:</span> <span class="nx">os</span><span class="p">.</span><span class="nf">userInfo</span><span class="p">().</span><span class="nx">username</span><span class="p">,</span>
  <span class="na">diretorioAtual</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nf">cwd</span><span class="p">(),</span>
  <span class="na">platform</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">platform</span><span class="p">,</span>
  <span class="na">nodeVersion</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">version</span><span class="p">,</span>
  <span class="c1">// A real attacker would try to read:</span>
  <span class="c1">// env: process.env,  // Tokens, passwords, API keys</span>
  <span class="c1">// arquivos: fs.readdirSync(process.env.HOME)</span>
<span class="p">};</span>

<span class="c1">// Creates "proof" file - in real attack would be sent to server</span>
<span class="kd">const</span> <span class="nx">arquivoProva</span> <span class="o">=</span> <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nf">cwd</span><span class="p">(),</span> <span class="dl">'</span><span class="s1">PROVA_ATAQUE_SUPPLY_CHAIN.json</span><span class="dl">'</span><span class="p">);</span>
<span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span><span class="nx">arquivoProva</span><span class="p">,</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">dadosSensiveis</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span> <span class="dl">'</span><span class="s1">utf8</span><span class="dl">'</span><span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`\u001b[31m[SIMULATED ATTACK]\u001b[0m Data collected saved to: </span><span class="p">${</span><span class="nx">arquivoProva</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="se">\n</span><span class="s1">In a REAL attack, this would be sent to the attacker</span><span class="se">\'</span><span class="s1">s server.</span><span class="se">\n</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p>Example of a file created after the script runs:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnlu8u1men5p4vzrl8qr6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnlu8u1men5p4vzrl8qr6.png" alt="exfiltrated information image" width="800" height="203"></a></p>

<p>And when running the application, the user doesn't even notice what happened:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3k5rdxd80cd30px0vos6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3k5rdxd80cd30px0vos6.png" alt="Image showing that the user will not see the malware execution" width="800" height="390"></a></p>

<p>In a real malicious version, the attacker would replace <code>fs.writeFileSync</code> with an HTTP call (e.g., using <code>https.request</code>) to send this data to a server under their control. The package also exposes a legitimate module (<code>index.js</code>) that does something useful — making the package plausible and reducing suspicion.</p>

<h3>
  
  
  Attack Flow
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Developer: npm install utilidades-uteis
2. npm downloads the package
3. npm runs preinstall  → malicious code #1
4. npm runs postinstall → malicious code #2 (collects data)
5. Package installed normally
6. Developer does not realize they have been compromised
</code></pre>

</div>






<h2>
  
  
  Consequences in Real Environments
</h2>

<p>What could happen if this were a real attack? The consequences vary depending on the victim's context.</p>

<h3>
  
  
  1. Exfiltration of Credentials and Secrets
</h3>

<ul>
<li>
<strong>Environment variables</strong> (<code>process.env</code>): API tokens (AWS, GitHub, Stripe), database keys, passwords</li>
<li>
<strong><code>.env</code> files</strong>: credentials in plain text across multiple projects</li>
<li>
<strong>Configuration files</strong>: <code>~/.npmrc</code>, <code>~/.aws/credentials</code>, <code>~/.ssh/config</code>
</li>
</ul>

<p><strong>Impact</strong>: Access to cloud accounts, databases, private repositories, and third-party systems.</p>

<h3>
  
  
  2. Theft of SSH Keys and Certificates
</h3>

<ul>
<li>Reading <code>~/.ssh/</code> (private keys, <code>known_hosts</code>)</li>
<li>Using the keys to access servers, GitHub, private repositories</li>
</ul>

<p><strong>Impact</strong>: Server intrusion, cloning of private repositories, malicious commits in the victim's name.</p>

<h3>
  
  
  3. Cryptojacking
</h3>

<ul>
<li>Running a cryptocurrency miner in the background</li>
<li>Consuming the victim's machine or server CPU and power</li>
</ul>

<p><strong>Impact</strong>: High infrastructure costs, performance degradation, possible violation of cloud usage policies.</p>

<h3>
  
  
  4. Backdoors and Persistence
</h3>

<ul>
<li>Installing remote access tools</li>
<li>Adding scheduled tasks or startup scripts</li>
</ul>

<p><strong>Impact</strong>: Prolonged control of the machine, espionage, preparation for future attacks.</p>

<h3>
  
  
  5. Modification of Other Packages
</h3>

<ul>
<li>Altering code in <code>node_modules</code> of other dependencies</li>
<li>Injecting backdoors into libraries used in production</li>
</ul>

<p><strong>Impact</strong>: Compromise at scale, propagation of the attack to all application users.</p>

<h3>
  
  
  6. In CI/CD Environments
</h3>

<ul>
<li>Access to pipeline secrets (tokens, credentials)</li>
<li>Ability to modify build artifacts or Docker images</li>
<li>Deployment of compromised versions to production</li>
</ul>

<p><strong>Impact</strong>: Compromise of the entire delivery chain, from build to production.</p>




<h2>
  
  
  Documented Real-World Cases
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Case</th>
<th>Year</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>event-stream</strong></td>
<td>2018</td>
<td>Package with ~2M weekly downloads. Malicious code added to steal Bitcoin wallets.</td>
</tr>
<tr>
<td><strong>ua-parser-js</strong></td>
<td>2021</td>
<td>Popular library compromised; ran cryptocurrency miner.</td>
</tr>
<tr>
<td>
<strong>coa</strong> and <strong>rc</strong>
</td>
<td>2021</td>
<td>Typosquatting; packages stole environment variables and sent them to a remote server.</td>
</tr>
<tr>
<td><strong>node-ipc</strong></td>
<td>2022</td>
<td>Added code that modified files on machines of developers from certain geographic regions.</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Protection Measures
</h2>

<ol>
<li>
<strong>Use <code>--ignore-scripts</code></strong> when possible: <code>npm install --ignore-scripts</code>
</li>
<li>
<strong>Audit dependencies</strong>: <code>npm audit</code>, <code>npm audit fix</code>
</li>
<li>
<strong>Verify scripts</strong>: <code>npm view package-name</code> before installing</li>
<li>
<strong>Specialized tools</strong>: Socket.dev, Snyk for detecting suspicious behavior</li>
<li>
<strong>Keep <code>package-lock.json</code></strong> in version control and review changes</li>
<li>
<strong>Verify package provenance</strong>: download counts, active maintenance, open repository</li>
</ol>




<h2>
  
  
  Conclusion
</h2>

<p>The npm ecosystem is extremely convenient, but that convenience carries risks. The seemingly trivial act of <code>npm install</code> can execute arbitrary code on your machine. Awareness of supply chain attacks and adopting security best practices are essential to reduce the attack surface and protect projects and infrastructure.</p>

<p>The demonstration project is available so you can test the flow in a controlled environment and understand how these attacks work in practice.</p>




<p><em>This article was written for educational purposes. The demonstration project contains only simulated code and does not perform any real malicious actions.</em></p>

