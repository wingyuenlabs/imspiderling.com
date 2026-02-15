---
Title: Termi-host : Web-Based Terminal Access from Any browser, Built with GitHub Copilot cli
Description: 
Author: vishal Prajapat
Date: 2026-02-15T21:33:55.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Description</strong>: A lightweight web terminal that lets you access your server from phones, tablets, TVs - anywhere with a browser. No SSH client needed!</p>

<h2>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdjabc7hlbgnfzij8a7hf.png" alt=" " width="800" height="407">
</h2>

<p><em>This is a submission for the <a href="https://dev.to/challenges/github-2026-01-21">GitHub Copilot CLI Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p><strong>termi-host</strong> is a lightweight web-based terminal that solves a real problem: What if you need to access your server but don't have an SSH client?</p>

<p>This happens more often than you'd think:</p>

<ul>
<li>📱 You're on your phone and need to check server logs</li>
<li>📺 You're using a smart TV browser</li>
<li>💻 You're on a public/work computer where you can't install SSH</li>
<li>🚨 Emergency access when you don't have your SSH keys</li>
<li>🎓 Teaching students without making them install software</li>
</ul>

<p><strong>termi-host</strong> gives you full terminal access through any web browser. Just install it on your VPS once, and access it from anywhere - phone, tablet, laptop, even your TV!</p>

<h3>
  
  
  Key Features:
</h3>

<ul>
<li>🌐 <strong>Works in any browser</strong> - Chrome, Firefox, Safari, Edge</li>
<li>🔒 <strong>Secure</strong> - Password protection, rate limiting, session timeouts</li>
<li>⚡ <strong>Super lightweight</strong> - Uses only ~16MB RAM per session</li>
<li>🎨 <strong>5 beautiful themes</strong> - Choose what looks best to you</li>
<li>📱 <strong>Mobile-friendly</strong> - Works great on phones and tablets</li>
<li>🚀 <strong>Easy setup</strong> - One command installation</li>
</ul>

<h3>
  
  
  GitHub:
</h3>

<p>🔗 <strong><a href="https://github.com/vpbgkt/termi-host" rel="noopener noreferrer">https://github.com/vpbgkt/termi-host</a></strong></p>

<h2>
  
  
  Demo
</h2>

<h3>
  
  
  Terminal Interface
</h3>

<p>![Terminal running commands](<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fugvfli5jzyry1bbmgwge.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fugvfli5jzyry1bbmgwge.png" alt=" " width="800" height="450"></a></p>

<p>)</p>
<h3>
  
  
  Login Screen
</h3>

<p>![Secure login page](<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvz1kc4sog177x1o33ldj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvz1kc4sog177x1o33ldj.png" alt=" " width="800" height="407"></a></p>

<p>)</p>
<h3>
  
  
  Mobile View
</h3>

<p>![Works perfectly on mobile](<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4rjh3cx4fdt4kpa7qmu3.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4rjh3cx4fdt4kpa7qmu3.jpeg" alt=" " width="720" height="1600"></a></p>

<p>)</p>
<h3>
  
  
  Quick Video Demo
</h3>

<p><a href="https://youtu.be/-UGABUjdxSY" rel="noopener noreferrer">https://youtu.be/-UGABUjdxSY</a><br>
<iframe width="710" height="399" src="https://www.youtube.com/embed/-UGABUjdxSY">
</iframe>
</p>

<h2>
  
  
  My Experience with GitHub Copilot CLI
</h2>

<p>GitHub Copilot CLI was like having an expert developer sitting next to me throughout this project. Here's how it helped:</p>

<h3>
  
  
  1. <strong>Quick Problem Solving</strong>
</h3>

<p>When I needed to set up WebSocket authentication (which is tricky!), I just asked:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gh copilot suggest <span class="s2">"how to authenticate websocket connections with express sessions"</span>
</code></pre>

</div>



<p>Copilot gave me the exact pattern I needed - sharing the session middleware between HTTP and WebSocket connections. This saved me hours of Stack Overflow searching!</p>

<h3>
  
  
  2. <strong>Security Features Made Easy</strong>
</h3>

<p>I wanted to add rate limiting to prevent brute force attacks but wasn't sure how. I asked:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gh copilot suggest <span class="s2">"add rate limiting to prevent login brute force attacks in express"</span>
</code></pre>

</div>



<p>Copilot suggested <code>express-rate-limit</code> and showed me exactly how to configure it (5 attempts per 15 minutes). Now the app is much more secure!</p>

<h3>
  
  
  3. <strong>Handling Edge Cases</strong>
</h3>

<p>One big problem: what happens if the user's internet drops? The terminal would crash and leave zombie processes. I asked:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gh copilot suggest <span class="s2">"cleanup terminal process when websocket disconnects in node-pty"</span>
</code></pre>

</div>



<p>Copilot showed me how to properly handle cleanup with event listeners. No more zombie processes!</p>

<h3>
  
  
  4. <strong>Production Deployment</strong>
</h3>

<p>I needed to run this as a system service that auto-restarts. Instead of googling systemd syntax:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gh copilot suggest <span class="s2">"create systemd service file for nodejs application with auto restart"</span>
</code></pre>

</div>



<p>Got a perfect systemd service file in seconds!</p>

<h3>
  
  
  5. <strong>Error Handling</strong>
</h3>

<p>Throughout the project, whenever I wrote code, I'd ask:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gh copilot suggest <span class="s2">"what error cases am I missing in this websocket handler"</span>
</code></pre>

</div>



<p>Copilot helped me think about edge cases I would have missed - like handling authentication failures, connection timeouts, and invalid sessions.</p>

<h3>
  
  
  The Impact:
</h3>

<p><strong>Before Copilot CLI:</strong></p>

<ul>
<li>Spend 30 minutes googling</li>
<li>Read 5-10 Stack Overflow threads</li>
<li>Try different solutions</li>
<li>Debug when they don't work</li>
<li>Finally get it working</li>
</ul>

<p><strong>With Copilot CLI:</strong></p>

<ul>
<li>Ask specific question</li>
<li>Get working solution in seconds</li>
<li>Understand why it works</li>
<li>Move on to next feature</li>
</ul>

<p><strong>Time saved: Probably 60-70% of development time!</strong></p>

<p>The best part? Copilot didn't just give me code - it explained <em>why</em> things work. I learned better patterns for WebSocket security, proper cleanup handlers, and production-ready error handling.</p>

<h3>
  
  
  Real Example:
</h3>

<p>Here's actual code that Copilot helped me write:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Problem: WebSocket needs to check if user is authenticated</span>
<span class="c1">// Copilot suggested this clean solution:</span>

<span class="nx">server</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">upgrade</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">request</span><span class="p">,</span> <span class="nx">socket</span><span class="p">,</span> <span class="nx">head</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">sessionMiddleware</span><span class="p">(</span><span class="nx">request</span><span class="p">,</span> <span class="p">{},</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">AUTH_ENABLED</span> <span class="o">&amp;&amp;</span> <span class="p">(</span><span class="o">!</span><span class="nx">request</span><span class="p">.</span><span class="nx">session</span> <span class="o">||</span> <span class="o">!</span><span class="nx">request</span><span class="p">.</span><span class="nx">session</span><span class="p">.</span><span class="nx">authenticated</span><span class="p">))</span> <span class="p">{</span>
      <span class="nx">socket</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="dl">'</span><span class="s1">HTTP/1.1 401 Unauthorized</span><span class="se">\r\n\r\n</span><span class="dl">'</span><span class="p">);</span>
      <span class="nx">socket</span><span class="p">.</span><span class="nf">destroy</span><span class="p">();</span>
      <span class="k">return</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="nx">wss</span><span class="p">.</span><span class="nf">handleUpgrade</span><span class="p">(</span><span class="nx">request</span><span class="p">,</span> <span class="nx">socket</span><span class="p">,</span> <span class="nx">head</span><span class="p">,</span> <span class="p">(</span><span class="nx">ws</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">wss</span><span class="p">.</span><span class="nf">emit</span><span class="p">(</span><span class="dl">'</span><span class="s1">connection</span><span class="dl">'</span><span class="p">,</span> <span class="nx">ws</span><span class="p">,</span> <span class="nx">request</span><span class="p">);</span>
    <span class="p">});</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Without Copilot, I would've struggled for hours trying to figure out session middleware sharing between HTTP and WebSocket!</p>

<h2>
  
  
  Why This Project Matters
</h2>

<p>In 2026, we should be able to manage our servers from anywhere. But traditional SSH has barriers:</p>

<ul>
<li>Requires client installation</li>
<li>Needs SSH keys (risky on shared devices)</li>
<li>Often blocked by firewalls</li>
<li>Impossible on some devices (smart TVs, gaming consoles)</li>
</ul>

<p><strong>termi-host removes all these barriers.</strong> If you have a browser, you have terminal access.</p>

<h3>
  
  
  Tech Stack:
</h3>

<ul>
<li>
<strong>Backend:</strong> Node.js, Express, WebSocket (ws), node-pty</li>
<li>
<strong>Frontend:</strong> xterm.js, vanilla JavaScript</li>
<li>
<strong>Security:</strong> express-session, express-rate-limit, security headers</li>
<li>
<strong>Deployment:</strong> Systemd, nginx-ready</li>
</ul>

<h3>
  
  
  Quick Start:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/vpbgkt/termi-host.git
<span class="nb">cd </span>termi-host
bash install.sh
npm start
</code></pre>

</div>



<p>That's it! Access at <code>http://localhost:3000</code></p>

<h3>
  
  
  Performance:
</h3>

<ul>
<li>✅ 16MB RAM per session</li>
<li>✅ &lt;50ms latency</li>
<li>✅ &lt;1 second startup</li>
<li>✅ Handles 20+ concurrent users</li>
</ul>

<h2>
  
  
  What's Next
</h2>

<p>I'm planning to add:</p>

<ul>
<li>Multi-session support (tabs)</li>
<li>File upload/download</li>
<li>Terminal recording/playback</li>
<li>Docker image for easier deployment</li>
<li>Mobile app wrapper</li>
</ul>

<h2>
  
  
  Try It Yourself!
</h2>

<p>🔗 <strong>Live Demo:</strong> <a href="http://159.65.151.238:3000" rel="noopener noreferrer">http://159.65.151.238:3000</a> (admin/admin)<br>
🔗 <strong>GitHub:</strong> <a href="https://github.com/vpbgkt/termi-host" rel="noopener noreferrer">https://github.com/vpbgkt/termi-host</a><br>
📖 <strong>Docs:</strong> Full installation guide in README</p>




<p><strong>License:</strong> MIT - Free and open source forever! ❤️</p>

<p>If you find this useful, please ⭐ star the repo and let me know in the comments! Questions? I'm happy to help! 🚀</p>




