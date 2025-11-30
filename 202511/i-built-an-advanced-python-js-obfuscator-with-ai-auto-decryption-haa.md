---
Title: I built an Advanced Python & JS Obfuscator with AI Auto-Decryption 🛡️
Description: 
Author: Hassan Magdy
Date: 2025-11-30T21:35:12.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey Developers! 👋</p>

<p>I'm <strong>EuroMoscow</strong>, a Full Stack Developer. Recently, I noticed that many online code obfuscators are either paid, outdated, or break the code after encryption (especially with Python versions mismatch).</p>

<p>So, I decided to build my own solution: <strong>EuroMoscow Shield</strong> 🛡️.</p>

<p>It is a free, mobile-responsive, and powerful tool to protect your Intellectual Property (IP) for both <strong>Python</strong> and <strong>JavaScript</strong> projects.</p>

<h2>
  
  
  🔗 Try it Live: <a href="https://euro-moscow-shield.vercel.app/" rel="noopener noreferrer">EuroMoscow Shield</a>
</h2>

<h2>
  
  
  💻 Source Code: <a href="https://github.com/Shadow132245/EuroMoscow-Shield" rel="noopener noreferrer">GitHub Repository</a>
</h2>




<h2>
  
  
  🔥 Key Features
</h2>

<p>I wanted to create something robust, so I implemented multiple layers of protection:</p>

<h3>
  
  
  🐍 For Python Developers
</h3>

<ul>
<li>
<strong>Safe Renaming:</strong> Smart AST parsing that renames variables/functions without breaking your logic.</li>
<li>
<strong>Portable Blob:</strong> A custom technique I developed to compress code into a zlib integer array (Runs on any Python 3.x version, solving the <code>Marshal</code> version error).</li>
<li>
<strong>EuroMoscow XOR:</strong> Custom encryption with dynamic keys.</li>
<li>
<strong>Import Guard:</strong> Automatically detects libraries (<code>sys</code>, <code>os</code>, etc.) and protects them from being renamed.</li>
</ul>

<h3>
  
  
  📜 For JavaScript Developers
</h3>

<ul>
<li>
<strong>Hex Encoding:</strong> Converts logic to hexadecimal strings (<code>\x68\x65...</code>).</li>
<li>
<strong>Char Code Array:</strong> Transforms code into number arrays.</li>
<li>
<strong>URL &amp; Base64 Encoding:</strong> Standard web obfuscation layers.</li>
</ul>

<h3>
  
  
  🤖 The "AI" Auto-Decryptor
</h3>

<p>This is my favorite feature. In the <strong>Decryptor</strong> section, you don't need to select which algorithm was used.<br>
The engine automatically scans the code using RegEx patterns and heuristically detects the layers (Base64 -&gt; Zlib -&gt; XOR -&gt; Blob) to recover the source code instantly!</p>




<h2>
  
  
  📱 Mobile Responsive &amp; Cyberpunk UI
</h2>

<p>I spent a lot of time polishing the UI. It features a dark "Cyberpunk" theme with:</p>

<ul>
<li>
<strong>CodeMirror</strong> integration for syntax highlighting.</li>
<li>
<strong>Client-Side Processing:</strong> Files are downloaded directly from the browser (No server size limits!).</li>
<li>
<strong>Fully Responsive:</strong> Works perfectly on mobile devices with a custom sidebar layout.</li>
</ul>




<h2>
  
  
  🛠️ Tech Stack
</h2>

<ul>
<li>
<strong>Backend:</strong> Python (Flask)</li>
<li>
<strong>Frontend:</strong> HTML5, CSS3, JavaScript</li>
<li>
<strong>Deployment:</strong> Vercel</li>
<li>
<strong>Libraries:</strong> <code>ast</code>, <code>zlib</code>, <code>base64</code>, <code>codemirror</code>
</li>
</ul>




<h2>
  
  
  🚀 API Access
</h2>

<p>I also built a public API for developers who want to integrate this protection into their CI/CD pipelines.</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
python
import requests

url = "[https://euro-moscow-shield.vercel.app/process](https://euro-moscow-shield.vercel.app/process)"

payload = {
    "code": "print('Hello Dev.to')",
    "action": "encrypt",
    "lang": "python",
    "options": ["rename", "marshal", "xor"]
}

response = requests.post(url, json=payload)
print(response.json()['result'])

🙌 Feedback
I built this project to help the community protect their open-source tools and bots. I would love to hear your feedback, suggestions, or bug reports!

If you find it useful, please give it a Star ⭐ on GitHub!

Happy Coding! 💻
</code></pre>

</div>

