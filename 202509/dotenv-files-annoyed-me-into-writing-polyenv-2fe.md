---
Title: Dotenv files annoyed me into writing Polyenv
Description: 
Author: Philip Meholm
Date: 2025-09-28T22:10:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>Tired of leaking secrets into <code>.env</code> files, and frustrated by the friction of existing tools, I built a simple CLI to separate config from secrets without the overhead of vaults.</p>

<p>If you don't live under a rock, you have at one time used dotenv files in some project to create/edit your configuration, or used it as a config file for something you made or cloned for self hosting.</p>

<p>Most frameworks and languages support them, and if not, its pretty easy to create something to import and use them (at least at a basic level). This is notable because it provides simplicity in a ocean of configuration frameworks while serving A LOT of the need for the developer and users.</p>

<p>But, trouble starts when you decide you want to store secrets inside dotenv files. Don't get me wrong, dotenv are great for simplicity. But when you start slipping secrets in there, it’s a bit like playing with matches in a fireworks factory. Commit + push the wrong .env file to git, and you’re basically performing version-control seppuku.</p>

<p>This isn't just paranoia, its happening <em>a lot</em>:</p>

<ul>
<li><a href="https://www.gitguardian.com/files/the-state-of-secrets-sprawl-report-2024" rel="noopener noreferrer">in 2023 about 11% of authors leaked a secret through dotenv on GitHub, 12 million secrets</a></li>
<li><a href="https://www.gitguardian.com/files/the-state-of-secrets-sprawl-report-2025" rel="noopener noreferrer">in 2024 about 15% of authors leaked a secret, in total 23 million secrets!</a></li>
<li><a href="https://unit42.paloaltonetworks.com/large-scale-cloud-extortion-operation/" rel="noopener noreferrer">just last year Palo Alto discovered a extensive cloud extortion campaign exploiting public .env files</a></li>
</ul>

<p>So what can you do about it?</p>

<h2>
  
  
  Current Alternatives
</h2>

<ul>
<li><p><strong>Mozilla SOPS</strong> - Encrypts values while leaving keys readable. really flexible (multiple formats, key stores, rotation  etc), Works well in teams comfortable with encrypt/decrypt as part of their pipeline.</p></li>
<li><p><strong>Dotenvx</strong> - Encrypts the entire file with a private key stored alongside. Straight forward setup and tooling, though sharing keys with teammates/CI requires extra coordination (or a ~$19/yr offering from Radar). </p></li>
<li><p><strong>Platform specific Secret managers</strong> - Hashicorp Vault, AWS/GCP Secret Manager, Azure Key Vault, CyberArk, etc. Centralized, battle tested and secure, Typically works best if your stack is tied to a given platform.</p></li>
<li><p><strong>.env.local + .gitignore + ci secrets</strong>  - Keeps sensitive values of version control while remaining simple and lightweight. Works fine for small projects, but contributors need some extra documentation or shared reference to know what variables exists.</p></li>
<li><p><strong>Pulling secrets at runtime</strong> - Fetch secrets directly from your enterprise vault during CI or app startup. Secure, 'simple' and increasingly common in cloud native setups. may add complexity to local development workflows, especially where hot-reload or offline work may be important.</p></li>
</ul>

<p>All of these approaches solve a real problem and each has its ideal context. For my everyday workflow though these solution either partially hit my use-case or i felt something was missing.</p>

<h2>
  
  
  Problem statement: treating all envs as secrets is overkill
</h2>

<p>Many tools treat <em>all</em> values in dotenv files as if they are secrets by default, or have some platform specific approaches.<br><br>
This often adds setup complexity and friction that may make developers or maintainers to take shortcuts "just for now" that end up being the default. This is really dangerous.</p>

<p><strong><em>Not every <code>env</code> value is a secret</em></strong>.<br><br>
Sometimes its just boring config like <code>PORT=3000</code>. why treat them all like secrets if only a handful need the proper protection?</p>

<p>That's the thinking behind polyenv: a tool I built to make handling secrets second nature and a natural fit for any dev workflow</p>
<h2>
  
  
  <a href="https://github.com/WithHolm/polyenv/tree/dev" rel="noopener noreferrer">Polyenv</a>
</h2>

<p>The core idea: keep secret references in a dedicated config instead of the values. these references are safe to commit since the actual access is controlled by your identity or CI. any secrets then would get downloaded to a git-ignored file, keeping them safe from git's purview</p>

<ul>
<li>
<strong>For anyone dealing with .env and possibly secrets</strong>

<ul>
<li>if you have one or many <code>.env</code> files or deal with secrets in <code>.env</code> files this can help you</li>
</ul>
</li>
<li>
<strong>Not a replacement</strong>

<ul>
<li>this is meant to be used <strong>with</strong> your dotenv config, not instead of.</li>
</ul>
</li>
<li>
<strong>"Any" enterprise vault</strong>

<ul>
<li>its a thin cli wrapper for many different enterprise vault readers, so you could replace the different "vault readers" with this one tool</li>
</ul>
</li>
<li>
<strong>Separate secrets from config</strong>

<ul>
<li>Secrets are saved in a separate <code>.env.secret</code> file that can be git-ignored</li>
</ul>
</li>
<li>
<strong>References, not values</strong>

<ul>
<li>Instead of leaking values, a separate polyenv config file is created to host metadata about the secrets (where to pull from) </li>
</ul>
</li>
<li>
<strong>Pull secrets on demand</strong>

<ul>
<li>running <strong>pull</strong> will pull down secrets using your identity (eg. Azure key vault via your logged in <code>az cli</code>) or secrets stored on your local credential manager</li>
</ul>
</li>
<li>
<strong>Workflow-friendly and local-first</strong>

<ul>
<li>secret rollover? <code>pull</code> to get the latest version</li>
<li>need to add secret from defined vault? <code>add secret</code> and you can select what secret you want to add</li>
<li>Supports multiple vaults in same environment</li>
<li>(planned) CI support so you don't have to change tooling in a different environment</li>
</ul>
</li>
</ul>

<p>Config and references are stored in a <code>.polyenv.toml</code> file. I considered dotenv, but its not the right fit for structured metadata. TOML offers a nice balance: simpler than JSON, less fragile than YAML. While it lacks native schema support, the tradeoff feels worth it for readability and ease of use.</p>
<h2>
  
  
  QuickStart
</h2>

<p>You have to download polyenv cli from <a href="https://github.com/WithHolm/polyenv/releases/latest" rel="noopener noreferrer">github releases</a>. Package manager and native code packages coming soon.</p>

<p>Every command is backed up with a interactive wizard in case you miss any arguments. I don't like argument-heavy cli's that just errors out if not specified correctly, so I'm trying a "yes, both" approach for this cli.</p>

<p>for now i just have to assume you know how to download a cli and set it as part of your session.</p>
<h3>
  
  
  Init
</h3>

<p>When you have downloaded the cli and its part of your session, you can go to your git repo and type in <code>polyenv init</code>. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhn90hc2dfzdnlkrb8tnp.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhn90hc2dfzdnlkrb8tnp.gif" alt="Init" width="1200" height="600"></a></p>

<p>it will ask you for environment name and give you some options for setting up and lead you through a wizard where you can add new vaults and secrets</p>

<p>Once you are done, you should have a config named <code>{yourenv}.polyenv.toml</code> in git root. If you want, you can move this file pretty much anywhere in your repo, except under <code>.git</code>,<code>vendor</code> or <code>node_modules</code>. the cli will search through your repo from git root to find it each time it spins up.<br>
if you defined "no env".. ie <code>.env</code> the name will be <code>.polyenv.toml</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="nn">[options]</span>
  <span class="py">hyphens_to_underscores</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="py">uppercase_locally</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="py">use_dot_secret_file_for_secrets</span> <span class="p">=</span> <span class="kc">true</span>

<span class="nn">[vault]</span>
  <span class="nn">[vault."Manual Input"]</span>
    <span class="py">service</span> <span class="p">=</span> <span class="s">"myservice"</span>
    <span class="py">type</span> <span class="p">=</span> <span class="s">"local"</span>
  <span class="nn">[vault.mydev]</span>
    <span class="py">store</span> <span class="p">=</span> <span class="s">"mystore"</span>
    <span class="py">type</span> <span class="p">=</span> <span class="s">"devvault"</span>

<span class="nn">[secret]</span>
  <span class="nn">[secret.GITHUB_TOKEN]</span>
    <span class="py">vault</span> <span class="p">=</span> <span class="s">"Manual Input"</span>
    <span class="py">content_type</span> <span class="p">=</span> <span class="s">"text/plain"</span>
    <span class="py">remote_key</span> <span class="p">=</span> <span class="s">"github-token"</span>
</code></pre>

</div>



<h3>
  
  
  Add vault
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgq1z5sebq66rc3dmw4f1.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgq1z5sebq66rc3dmw4f1.gif" alt="adding a vault" width="1200" height="600"></a></p>

<p>to add a vault you can either use <code>polyenv !{yourenv} add</code> and select vault or <code>polyenv !{yourenv} add vault</code> to get to the <code>add vault wizard</code><br>
you will then be given a choice between several vault providers and then lead through a wizard for the vault of your choice.</p>

<h3>
  
  
  Add secret
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6cp3nxeo666rx6uzs7jh.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6cp3nxeo666rx6uzs7jh.gif" alt="adding a secret" width="1200" height="600"></a></p>

<p><code>polyenv !{yourenv} add secret</code> will add a new secret. you get to select from your defined vaults, when within that vault select your secret. the actual "wizard" for this is defined per vault for best use</p>

<h3>
  
  
  Emitting data
</h3>

<p><code>polyenv !{yourenv} export</code></p>

<p>by default export will show the stats of all values so you don't accidentally show data in your current session.</p>

<p>in order to grant the best user-experience this single command can both format the data in many different ways and write it to many different "outs".</p>

<h4>
  
  
  formatting
</h4>

<p>to specify a formatter, you can append <code>--as</code> with your given format: </p>

<ul>
<li>
<code>json</code>

<ul>
<li><code>{"MY_KEY":"MYVAL"}</code></li>
</ul>


</li>

<li>
<code>dotenv</code>

<ul>
<li><code>MYKEY="MYVAL"</code></li>
</ul>


</li>

<li>
<code>azdevops</code>

<ul>
<li><code>##vso[task.setvariable..isSecret=true;MYKEY]MYVAL</code></li>
<li>Loads all env vars into your current Azure Devops pipeline variable</li>
</ul>


</li>

<li>
<code>pwsh</code>

<ul>
<li><code>polyenv !env export --to pwsh|Iex</code></li>
<li>Loads all env vars into your current PowerShell session.</li>
</ul>


</li>

<li>
<code>bash</code>

<ul>
<li><code>eval "$(polyenv !{yourenv} export --as bash)</code></li>
<li>Exports all vars into your current Bash session</li>
</ul>


</li>

<li>
<code>stats</code> (default)

<ul>
<li>shows where all your env variables comes from (what files or vaults) and if they should be deemed a secret.</li>
</ul>


</li>

</ul>

<p>all <code>--as</code> formatters have the ability of checking if each env value is a secret via regex rules and act accordingly to help you not omit any values polyenv deemes a secret. both <code>stats</code> and <code>azdevops</code> uses this feature.</p>

<h4>
  
  
  writing
</h4>

<p>to specify a writer you can append <code>--to</code> with the set writer:</p>

<ul>
<li>
<code>stdout</code>

<ul>
<li>standard output</li>
</ul>


</li>

<li>
<code>github-env</code>

<ul>
<li>standard github env</li>
</ul>


</li>

<li>
<code>github-out</code>

<ul>
<li>standard github output</li>
</ul>


</li>

</ul>

<p>each writer have a set of preferred and allowed formatters to give you the best possible chances of success, so you don't need to give for example <code>--as github-env</code> a <code>--as dotenv</code>, as the only possible option for exporting to gh is dotenv, so this is implied when using the writer: <code>polyenv !yourenv export --to github-out</code></p>

<h2>
  
  
  Future work
</h2>

<p>the export topic is one of the core elements of this as it allows you to be really dynamic when setting up your pipeline or automation. I really want this to be like "rclone" of env management, and hopefully by focusing on  writers and formatters and distribution, this will be easier to achieve than other methods</p>

<ul>
<li>i want to create better and more formatters and writers so this could be used more places

<ul>
<li>export to onetimesecret: not affiliated, i just really like their service. planning a wizard where you can select keys and then format and get a link back you can share with collaborators</li>
</ul>


</li>

<li>Addition of imports. How its going to work i dont know yet..</li>

<li>more implementations of vaults

<ul>
<li>currently only keyvault and local keyring/cred store</li>
<li><a href="https://github.com/WithHolm/polyenv/issues/56" rel="noopener noreferrer">issue tracker</a></li>
</ul>


</li>

<li>better control over secrets

<ul>
<li>add env to a vault</li>
<li>change vaults for a secret</li>
<li>update secret value</li>
</ul>


</li>

<li>better documentation</li>

<li>better TUI elements for managing the environment</li>

<li>packages so it can be used </li>

<li>better handling of arguments-&gt;wizard actions</li>

</ul>

<p>If you have any other suggestion, are interested to help out, no need to deliver actual code, please feel free to join the project. </p>

