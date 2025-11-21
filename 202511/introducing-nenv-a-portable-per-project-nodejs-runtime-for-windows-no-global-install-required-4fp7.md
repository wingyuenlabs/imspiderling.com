---
Title: Introducing nenv — A portable, per-project Node.js runtime for Windows (no global install required)
Description: 
Author: Sajjad Tabreez
Date: 2025-11-21T21:09:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most of us have dealt with Node.js version conflicts at some point:</p>

<ul>
<li>One project needs Node 18.</li>
<li>Another needs Node 20.</li>
<li>A global upgrade breaks something.</li>
<li>Corporate or restricted laptops don’t allow installers.</li>
<li>CI/CD behaves differently from local environments.</li>
<li>Teammates run “slightly different” versions and bugs magically appear.</li>
</ul>

<p>On Linux/macOS, tools like nvm, asdf, volta, and fnm help…<br>
But on Windows, especially without admin rights, things get messy.</p>

<p>So I started experimenting with a simple idea:</p>

<p>What if every project had its own Node.js runtime, completely local, portable, and isolated?</p>

<p>Just like Python has .venv.</p>

<p>That experiment grew into a small open-source tool called nenv.</p>

<p>🚀 <strong>nenv — Portable Node.js per project (Windows)</strong></p>

<p>nenv is a lightweight script that downloads Node.js directly into your project folder and makes all Node/npm commands use that local runtime — even if no global Node is installed on the system.</p>

<p>It's basically:<br>
node.exe + npm + npx<br><br>
... stored inside the project</p>

<p>So project A can use Node 18<br>
and project B can use Node 20<br>
with zero conflict.</p>

<p><strong>📂 Example Project Structure</strong></p>

<p>After initializing:</p>

<p>project/<br>
│ nenv.cmd<br>
│ .nenv-version<br>
└── .nenv/<br>
    └── node-v20.11.1-win-x64/<br>
        ├── node.exe<br>
        ├── npm.cmd<br>
        ├── npx.cmd</p>

<p>Everything stays inside your project.<br>
Nothing touches your system installation.</p>

<p><strong>📦 Installation</strong></p>

<p>Run this inside your project root:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Invoke-WebRequest -Uri "https://cdn.jsdelivr.net/gh/tabreezsajjad/nenv@v0.5.0/nenv.txt" -OutFile "nenv.cmd"
</code></pre>

</div>



<p>Initialize:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.\nenv.cmd init

</code></pre>

</div>



<p>Enter the Node version you want (e.g., 20.11.1).</p>

<p>nenv downloads the ZIP from the official Node.js site and sets it up inside .nenv/.</p>

<p>🧪 <strong>Usage</strong><br>
Check Node version:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.\nenv.cmd node --version
</code></pre>

</div>



<p>Install dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.\nenv.cmd npm install

</code></pre>

</div>



<p>Run scripts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.\nenv.cmd dev
.\nenv.cmd build
.\nenv.cmd start
</code></pre>

</div>



<p>These behave exactly like:</p>

<p>npm run dev<br>
npm run build<br>
npm run start</p>

<p>but always using the project-local Node runtime.</p>

<p>✔ <strong>Supported Versions</strong></p>

<p>nenv supports any official Node.js Windows x64 ZIP:</p>

<ul>
<li>14.x (legacy)</li>
<li>16.x (legacy)</li>
<li>18.x (LTS)</li>
<li>20.x (LTS)</li>
<li>22.x (Current)</li>
<li>24.x (Latest)</li>
</ul>

<p>💡 <strong>Why this approach?</strong></p>

<ul>
<li>No admin rights needed</li>
<li>No global Node installation required</li>
<li>Fully self-contained environments</li>
<li>Perfect for Windows developers</li>
<li>Great for corporate/restricted laptops</li>
<li>Reproducible environments across teams</li>
<li>CI/CD matches local dev</li>
</ul>

<p>One project’s Node version never affects another</p>

<p>🔗 GitHub Repo</p>

<p><a href="https://github.com/tabreezsajjad/nenv" rel="noopener noreferrer">https://github.com/tabreezsajjad/nenv</a></p>

<p>The project is still early, but it works reliably and I’d love feedback, feature requests, or edge cases to test.</p>

<p>🙏 Thanks for reading!</p>

<p>If you try it out, I’d love to hear how it works for your workflow — especially if you're on Windows or dealing with Node version chaos across multiple projects.</p>

