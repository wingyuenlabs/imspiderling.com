---
Title: How I Verify PC Workman Security Before Every exe Release
Description: 
Author: Marcin Firmuga
Date: 2026-01-21T21:00:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you want people to trust your software, you need to <strong>prove it</strong>.</p>

<p>Not say it. Prove it.</p>

<h2>
  
  
  The Problem
</h2>

<p>Here's the reality: <strong>5 out of 6 repositories on freelance platforms have malware.</strong></p>

<p>Most indie devs handle security like this:</p>

<ol>
<li>Upload .exe</li>
<li>Write "safe and tested" in description</li>
<li>Maybe add "open source"</li>
<li>Ship it</li>
</ol>

<p>I get it. Security verification takes time. Documentation takes even more. When you're building alone, time is the only thing you don't have.</p>

<p>But users are running your code on their machines. Giving you system access. <strong>That deserves more than a promise.</strong></p>

<p>This is Part 1 of my 3-part series on how I verify <a href="https://github.com/HuckleR2003/PC_Workman_HCK" rel="noopener noreferrer">PC_Workman</a> security before every release.</p>




<h2>
  
  
  What I Actually Do
</h2>

<p>Before every stable release, I run the same process. Every time. No shortcuts.</p>

<h3>
  
  
  1. GitHub Security Features (Always On)
</h3>

<p>The baseline. Not optional.</p>

<ul>
<li> <strong>Dependabot</strong> - Watches for vulnerable dependencies, auto-creates PRs</li>
<li> <strong>Secret Scanning</strong> - Prevents accidentally committing API keys</li>
<li> <strong>Security Advisories</strong> - Public disclosure channel if issues are found</li>
<li> <strong>Private Vulnerability Reporting</strong> - Secure channel for researchers</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh358uixvla8588vsmx5w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh358uixvla8588vsmx5w.png" alt=" " width="800" height="281"></a></p>

<p>These aren't "nice to have." They're <strong>mandatory</strong> for any project distributing executables.</p>

<h3>
  
  
  2. CodeQL Analysis (Every Commit)
</h3>

<p>GitHub runs <a href="https://codeql.github.com/" rel="noopener noreferrer">CodeQL</a> on every commit to PC_Workman.</p>

<p>What it catches:</p>

<ul>
<li>SQL injection patterns</li>
<li>Command injection vulnerabilities</li>
<li>Insecure data handling</li>
<li>Common security anti-patterns</li>
</ul>

<p>Not just at release. <strong>Every. Single. Commit.</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/codeql.yml</span>
<span class="na">name</span><span class="pi">:</span> <span class="s2">"</span><span class="s">CodeQL"</span>
<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span> <span class="nv">main</span> <span class="pi">]</span>
  <span class="na">pull_request</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span> <span class="nv">main</span> <span class="pi">]</span>
  <span class="na">schedule</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">cron</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0</span><span class="nv"> </span><span class="s">0</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">0'</span>  <span class="c1"># Weekly scan</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">analyze</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">permissions</span><span class="pi">:</span>
      <span class="na">security-events</span><span class="pi">:</span> <span class="s">write</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v3</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">github/codeql-action/init@v2</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">languages</span><span class="pi">:</span> <span class="s">python</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">github/codeql-action/analyze@v2</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk5honnxhy7aujuo7cd44.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk5honnxhy7aujuo7cd44.png" alt=" " width="800" height="326"></a></p>

<h3>
  
  
  3. VirusTotal Scan (Before Every Release)
</h3>

<p>I upload every <code>.exe</code> to <a href="https://www.virustotal.com" rel="noopener noreferrer">VirusTotal</a> before publishing.</p>

<p><strong>Not one antivirus engine. Seventy.</strong></p>

<p>For v1.6.4:</p>

<ul>
<li>
<strong>Result:</strong> 4/70 detections</li>
<li>
<strong>Why 4?</strong> False positives from lesser-known engines that flag PyInstaller by default</li>
<li>
<strong>Major engines (Defender, Kaspersky, Malwarebytes):</strong> All clean</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgeyfdc1zzaqglsbwnyvl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgeyfdc1zzaqglsbwnyvl.png" alt=" " width="685" height="257"></a></p>

<p>I document these results. Save screenshots. Link to scans in release notes.</p>

<p>You can verify any release yourself:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Download PC_Workman.exe</span>
<span class="c"># 2. Go to virustotal.com</span>
<span class="c"># 3. Upload file</span>
<span class="c"># Expected: 0-4 detections (PyInstaller false positives)</span>
</code></pre>

</div>






<h2>
  
  
  Why Documentation Matters
</h2>

<p>Running scans is step one. <strong>Documenting them is step two.</strong></p>

<p>I maintain a <a href="https://huckler2003.github.io/PC_Workman_HCK/SECURITY_report.html" rel="noopener noreferrer">Security Report page</a> showing:</p>

<ul>
<li>Which version was tested</li>
<li>When tests ran</li>
<li>What the results were</li>
</ul>

<p>When v1.6.5 releases, that page updates. New scan results. New dates. New proof.</p>

<p>This creates <strong>history</strong>. Users can verify that security isn't a one-time thing.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkx7hpbf91l8mm7xvitc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkx7hpbf91l8mm7xvitc.png" alt=" " width="800" height="632"></a></p>




<h2>
  
  
  The Pre-Release Checklist
</h2>

<p>Here's exactly what happens before a stable release:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>[ ] Final build compiled
[ ] VirusTotal scan complete (save results)
[ ] CodeQL shows 0 alerts
[ ] Security Report page update
[ ] All about POLICY created.
[ ] Release notes include verification links
[ ] Screenshots archived
</code></pre>

</div>



<p>Nothing ships without this completing. <strong>No exceptions.</strong></p>




<h2>
  
  
  What This Doesn't Cover
</h2>

<p>File scanning catches:</p>

<ul>
<li> Known malware signatures</li>
<li> Common vulnerability patterns</li>
</ul>

<p>File scanning <strong>doesn't</strong> prove:</p>

<ul>
<li> Code does what it claims</li>
<li> Who actually published the release</li>
<li> Binary hasn't been tampered with</li>
</ul>

<p>Those are separate problems requiring separate solutions.</p>

<p><strong>Coming up:</strong></p>

<ul>
<li>
<strong>Part 2:</strong> Sigstore (cryptographic signatures proving origin)</li>
<li>
<strong>Part 3:</strong> OpenSSF Best Practices Badge (60+ security requirements)</li>
</ul>




<h2>
  
  
  The Point
</h2>

<p>Security isn't a marketing checkbox. <strong>It's a process.</strong></p>

<p>I do this because users deserve to know exactly what they're downloading.</p>

<p>Every test. Every version. Every time.</p>




<h2>
  
  
  Try PC_Workman
</h2>

<p><strong>Current version:</strong> v1.6.4<br><br>
<strong>GitHub:</strong> <a href="https://github.com/HuckleR2003/PC_Workman_HCK" rel="noopener noreferrer">HuckleR2003/PC_Workman_HCK</a><br><br>
<strong>Security Report:</strong> <a href="https://huckler2003.github.io/PC_Workman_HCK/security" rel="noopener noreferrer">View verification results</a></p>

<h3>
  
  
  What is PC_Workman?
</h3>

<p>AI-powered PC monitoring tool. Built solo on dying hardware during warehouse shifts in the Netherlands.</p>

<p><strong>Features:</strong></p>

<ul>
<li>Real-time CPU/GPU/RAM monitoring</li>
<li>AI diagnostics (HCK_GPT)</li>
<li>Custom fan control with profiles</li>
<li>Time-travel debugging (see what was running hours ago)</li>
<li>100% open source</li>
</ul>

<p><strong>Tech stack:</strong></p>

<ul>
<li>Python + PyQt5</li>
<li>PyInstaller for .exe builds</li>
<li>Sigstore for signing (Part 2 topic)</li>
<li>CodeQL for security scanning</li>
</ul>




<h2>
  
  
  Series Navigation
</h2>

<p>This is <strong>Part 1 of 3</strong> in the PC_Workman Security Series:</p>

<ol>
<li>
<strong>File Scanning &amp; Repository Security</strong> (you are here)</li>
<li>Sigstore and Cryptographic Verification (coming soon)</li>
<li>Working Toward OpenSSF Best Practices Badge (coming soon)</li>
</ol>




<h2>
  
  
  About Me
</h2>

<p>I'm Marcin Firmuga. Solo dev at <a href="https://github.com/HuckleR2003" rel="noopener noreferrer">HCK_Labs</a>.</p>

<p>Built PC_Workman from scratch:</p>

<ul>
<li>680+ hours of code</li>
<li>4 complete UI rebuilds</li>
<li>16,000 lines deleted</li>
<li>Coded after 10-hour warehouse shifts</li>
<li>On a laptop hitting 94°C</li>
</ul>

<p>Before this: game translations, PC technician internships, and countless projects I never finished.<br>
But this one stuck.</p>

<p><strong>Find me:</strong></p>

<ul>
<li>GitHub: <a href="https://github.com/HuckleR2003" rel="noopener noreferrer">@HuckleR2003</a>
</li>
<li>X/Twitter: <a href="https://twitter.com/hck_lab" rel="noopener noreferrer">@hck_lab</a>
</li>
<li>LinkedIn: <a href="https://linkedin.com/in/marcinfirmuga" rel="noopener noreferrer">/marcinfirmuga</a>
</li>
</ul>

<p>--</p>

<h2>
  
  
  Discussion
</h2>

<p><strong>Questions for the community:</strong></p>

<ol>
<li>How do you handle security verification for your projects?</li>
<li>Ever had a false positive that scared users away?</li>
<li>What security tools do you swear by?</li>
</ol>

<p>Drop your experiences in the comments. Let's learn from each other.<br>
FEEDBACK IS GOLD!</p>

<p>--</p>

<p><em>Building in public. Securing in public. One release at a time.</em></p>

<h1>
  
  
  BuildInPublic #Security #OpenSource #Python #IndieHacker
</h1>

<h2>
  
  
  About the Author
</h2>

<p>I’m Marcin Firmuga. Solo developer and founder of HCK_Labs.</p>

<p>I created PC Workman , an open-source, AI-powered<br>
PC resource monitor<br>
built entirely from scratch on dying hardware during warehouse<br>
shifts in the Netherlands.</p>

<p>This is the first time I’ve given one of my projects a real, dedicated home.</p>

<p><strong>Before this</strong>: </p>

<blockquote>
<p>game translations, PC technician internships, warehouse operations in multiple countries, and countless failed projects I never finished.</p>
</blockquote>

<p><strong>But this one? This one stuck.</strong><br>
700+ hours of code. 4 complete UI rebuilds. 16,000 lines deleted.<br>
3 AM all-nighters. Energy drinks and toast.</p>

<p><strong>And finally,</strong> an app I wouldn’t close in 5 seconds.<br>
That’s the difference between <strong>building **and **shipping</strong>.</p>

<p>PC_Workman is the result.</p>

<h1>
  
  
  BuildInPublic #IndieDev #OpenSource #Python #World
</h1>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6fpp1beil2hzcwyiwxu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6fpp1beil2hzcwyiwxu.png" alt=" " width="720" height="539"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." alt="Uploading image" width="800" height="400"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." alt="Uploading image" width="800" height="400"></a></p>

