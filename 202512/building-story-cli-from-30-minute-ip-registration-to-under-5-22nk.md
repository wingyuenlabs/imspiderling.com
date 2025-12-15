---
Title: Building Story CLI: From 30-Minute IP Registration to Under 5
Description: 
Author: Ola Adesoye
Date: 2025-12-15T21:43:25.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The problem that sparked this project
</h2>

<p>Here's something that bothered me about Web3 developer tools: they're often built <em>by</em> experienced blockchain developers <em>for</em> experienced blockchain developers. </p>

<p>In the case of IP registration via blockchain, if you're a solo creator who just wants to register your artwork, music, or code as intellectual property on-chain, you're looking at 15-30 minutes of fumbling through documentation, manually formatting JSON metadata, and praying your transaction doesn't fail after you've already spent gas.</p>

<p>I built <strong>Story CLI</strong> to fix that.</p>




<h2>
  
  
  What I built
</h2>

<p>Story CLI is a command-line toolkit for registering IP assets on <a href="https://story.foundation" rel="noopener noreferrer">Story Protocol</a>—a blockchain designed specifically for programmable intellectual property. Instead of writing code or crafting API calls, you get an interactive wizard:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>story register ./my-artwork.jpg
</code></pre>

</div>



<p>That's it. The CLI walks you through license selection, handles IPFS uploads, executes the blockchain transaction, and gives you a shareable portfolio visualization of all your registered IP.</p>

<p>The goal was simple: <strong>take a 15-30 minute process and compress it to under 5 minutes</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj8xe1mw96kb30qes8v3w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj8xe1mw96kb30qes8v3w.png" alt=" " width="800" height="292"></a></p>

<h3>
  
  
  1. The license wizard state machine
</h3>

<p>One of the trickiest parts was translating legal license configurations into something a human could answer in 30 seconds. Story Protocol uses PIL (Programmable IP License) with multiple parameters—commercial use permissions, derivative rights, royalty percentages.</p>

<p>I mapped this to a 3-question decision tree:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Commercial use? → Yes/No
Allow derivatives? → Yes/No
Revenue share? → 0-100% (only if commercial + derivatives)
</code></pre>

</div>



<p>These three questions deterministically map to one of 4 license configurations:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Commercial</th>
<th>Derivatives</th>
<th>Result</th>
</tr>
</thead>
<tbody>
<tr>
<td>No</td>
<td>No</td>
<td>Non-commercial Only</td>
</tr>
<tr>
<td>No</td>
<td>Yes</td>
<td>Non-commercial Derivatives</td>
</tr>
<tr>
<td>Yes</td>
<td>No</td>
<td>Commercial No-Derivatives</td>
</tr>
<tr>
<td>Yes</td>
<td>Yes</td>
<td>Commercial Remix (+ royalty %)</td>
</tr>
</tbody>
</table></div>

<p>The state machine approach meant I could validate answers in real-time and prevent invalid combinations before they ever hit the blockchain.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fytgbexj4sd9xcdkw9y07.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fytgbexj4sd9xcdkw9y07.png" alt=" " width="800" height="1031"></a></p>

<h3>
  
  
  2. Fail-fast everything
</h3>

<p>Blockchain transactions cost gas. Failed transactions <em>still</em> cost gas. This created a design imperative: <strong>validate everything before touching the chain</strong>.</p>

<ul>
<li>Wallet address format? Checked before any network call</li>
<li>IPFS hash format? Validated at input</li>
<li>Sufficient balance? Queried before transaction submission</li>
<li>Royalty percentage? Bounded 0-100 at prompt time</li>
</ul>

<p>The philosophy: if something's going to fail, fail in the first 2 seconds, not after a 30-second transaction attempt.</p>

<h3>
  
  
  3. Three-part error messages
</h3>

<p>Every error in Story CLI follows a structure: <strong>What</strong> went wrong → <strong>Why</strong> it matters → <strong>How</strong> to fix it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✖ Pinata API key not found
  IPFS uploads require Pinata authentication for metadata storage.
  Run: story config set pinataApiKey YOUR_KEY
</code></pre>

</div>



<p>This sounds obvious, but most CLI tools just dump "Error: invalid credentials" and leave you to figure it out. I spent real time ensuring every failure state had actionable guidance.</p>

<h3>
  
  
  4. Self-contained portfolio HTML
</h3>

<p>After registration, users needed to <em>see</em> their IP. I chose Mermaid.js for graph visualization because it lets me generate a <strong>single HTML file</strong> with everything embedded—CSS, JavaScript, and relationship diagrams.</p>

<p>No server required. Download the file, email it to someone, and open it. It works.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>story portfolio
<span class="c"># → generates story-portfolio.html with interactive graphs</span>
</code></pre>

</div>



<p>The tradeoff was less visual customization than D3.js would offer, but for an MVP, shipping &gt; perfection.</p>




<h2>
  
  
  Challenges &amp; what I learned
</h2>

<h3>
  
  
  Challenge 1: SDK documentation gaps
</h3>

<p>Story Protocol is relatively new. The SDK documentation had gaps—especially around error responses and edge cases. I ended up reading SDK source code directly and building a mock implementation (<code>STORY_CLI_MOCK=true</code>) so I could develop offline without burning testnet ETH.</p>

<p><strong>Lesson:</strong> When integrating new SDKs, budget time for exploration. Mock modes aren't just for testing—they're essential for iteration speed.</p>

<h3>
  
  
  Challenge 2: Terminal UX is harder than it looks
</h3>

<p>Making a CLI feel "good" requires attention to details you take for granted in web UIs:</p>

<ul>
<li>Spinners during async operations (Ora)</li>
<li>Color-coded output for visual hierarchy (Chalk)</li>
<li>Boxed success messages for celebration moments (Boxen)</li>
<li>Clear prompt validation with inline feedback (Inquirer)</li>
</ul>

<p>Each library handles a specific UX need. Combining them into a cohesive experience took more iteration than expected.</p>

<p><strong>Lesson:</strong> CLI UX is a real discipline. Users notice when it's done well—they just don't notice consciously.</p>

<h3>
  
  
  Challenge 3: Config file security
</h3>

<p>Storing API keys and wallet information in <code>~/.storyrc</code> required thinking about permissions. The config file is created with <code>chmod 600</code> (owner read/write only), and sensitive values can be overridden via environment variables for CI/CD pipelines.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">STORY_WALLET_PRIVATE_KEY</span><span class="o">=</span>0x...  <span class="c"># Override config file</span>
story register ./asset.png
</code></pre>

</div>



<p><strong>Lesson:</strong> Security isn't a feature—it's a constraint that shapes your entire design.</p>




<h2>
  
  
  The Tech Stack
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F41oqatwbubn3bay5vu75.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F41oqatwbubn3bay5vu75.png" alt=" " width="800" height="497"></a></p>

<h2>
  
  
  Architecture at a glance
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User terminal
    │
    ▼
Command router (Commander.js)
    │
    ├──► Register command
    │    ├── License wizard (Inquirer.js)
    │    ├── Metadata prompts
    │    ├── IPFS upload (Pinata SDK)
    │    └── Blockchain transaction (Story SDK)
    │
    ├──► Portfolio command
    │    ├── Asset fetching (Story API)
    │    ├── Graph building (Mermaid.js)
    │    └── HTML rendering
    │
    ├──► Config command
    │    └── ~/.storyrc management
    │
    └──► Status command
         └── Wallet &amp; network info
</code></pre>

</div>



<p>Key patterns used:</p>

<ul>
<li>
<strong>Command pattern</strong> — Each CLI command is an isolated handler</li>
<li>
<strong>Facade pattern</strong> — <code>StoryClient</code> wraps the complex SDK</li>
<li>
<strong>Singleton pattern</strong> — <code>ConfigManager</code> prevents redundant file reads</li>
<li>
<strong>Fail-fast validation</strong> — Errors surface immediately, not after blockchain calls</li>
</ul>




<h2>
  
  
  Why this matters (Beyond the code)
</h2>

<p>IP registration on blockchain is one of those "obviously useful" ideas that's been stuck behind technical barriers. The people who would benefit most—independent artists, open-source developers, content creators—are often the least equipped to navigate Web3 tooling.</p>

<p>Story CLI is my attempt at a bridge. Not a dumbed-down version, but a <em>well-designed</em> version that respects users' time and treats errors as communication opportunities rather than dead ends.</p>

<p>The goal was never to abstract away the blockchain entirely—it was to remove the <em>friction</em> while preserving the power.</p>




<h2>
  
  
  Try it yourself
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-g</span> story-cli
story <span class="nt">--help</span>
</code></pre>

</div>



<h3>
  
  
  Quick start
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Configure your wallet</span>
story config <span class="nb">set </span>walletAddress 0xYourAddress
story config <span class="nb">set </span>network testnet

<span class="c"># Set up IPFS (Pinata)</span>
story config <span class="nb">set </span>pinataApiKey YOUR_KEY
story config <span class="nb">set </span>pinataApiSecret YOUR_SECRET

<span class="c"># Register your first IP asset</span>
story register ./my-artwork.jpg

<span class="c"># View your portfolio</span>
story portfolio
</code></pre>

</div>






<h2>
  
  
  What's next
</h2>

<ul>
<li>Batch registration for multiple assets</li>
<li>License template sharing</li>
<li>Integration with popular creative tools</li>
<li>Mainnet deployment guides</li>
</ul>




<p><strong>Links:</strong></p>

<ul>
<li><a href="https://github.com/Zolldyk/story-cli" rel="noopener noreferrer">GitHub repository</a></li>
<li><a href="https://story.foundation" rel="noopener noreferrer">Story Protocol</a></li>
</ul>

