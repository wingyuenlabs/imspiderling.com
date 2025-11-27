---
Title: Getting Started With Nmap: A Beginner-Friendly Guide
Description: 
Author: Brian Shisia
Date: 2025-11-27T21:41:22.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’re getting into cybersecurity, ethical hacking, or network engineering, you’ll quickly hear the name <strong>Nmap</strong>. When I started learning Nmap, it felt overwhelming - so many flags, so many scan types, so many outputs.</p>

<p>At the same time, I’m building a small tool in <strong>Go</strong> that works like Nmap, which forced me to understand what Nmap actually does behind the scenes.</p>

<p>This article is a simple introduction to Nmap for beginners — no assumptions, no jargon overload.</p>




<h2>
  
  
  What is Nmap?
</h2>

<p><strong>Nmap (Network Mapper)</strong> is a free and open-source tool used to discover hosts, open ports, running services, OS information, and much more.</p>

<p>Think of Nmap as a "network Sherlock Holmes" — it asks questions like:</p>

<ul>
<li>Who is online?</li>
<li>Which ports are open?</li>
<li>What services are running?</li>
<li>What OS might this machine be using?</li>
</ul>




<h2>
  
  
  Why Nmap Is Important (Even for Developers)
</h2>

<p>Even if you’re not a pentester, Nmap helps you:</p>

<ul>
<li>Test your own servers for exposed ports</li>
<li>Learn how attackers might discover your infrastructure</li>
<li>Troubleshoot network issues</li>
<li>Understand how protocols behave</li>
<li>Build your own network tools (like I’m doing in Go)</li>
</ul>




<h2>
  
  
  Installing Nmap
</h2>

<h3>
  
  
  Linux (Debian/Ubuntu/Kali)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt <span class="nb">install </span>nmap
</code></pre>

</div>



<h3>
  
  
  Arch
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>pacman <span class="nt">-S</span> nmap
</code></pre>

</div>



<h3>
  
  
  Mac
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>brew <span class="nb">install </span>nmap
</code></pre>

</div>



<h3>
  
  
  Windows
</h3>

<p>Download the installer from: <a href="https://nmap.org/download.html" rel="noopener noreferrer">https://nmap.org/download.html</a></p>




<h2>
  
  
  The Most Important Nmap Commands for Beginners
</h2>

<p>Let’s go through practical commands you’ll actually use.</p>




<h3>
  
  
  1. Basic Host Scan
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nmap 192.168.1.10
</code></pre>

</div>



<p>Scans common ports and shows open ones.</p>




<h3>
  
  
  2. Ping Scan (Check if a host is online)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nmap <span class="nt">-sn</span> 192.168.1.0/24
</code></pre>

</div>



<p>Good for discovering devices on a network.</p>




<h3>
  
  
  3. Scan all ports (1–65535)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nmap <span class="nt">-p-</span> 192.168.1.10
</code></pre>

</div>



<p>If you want <em>everything</em>, this is the scan.</p>




<h3>
  
  
  4. Fast Scan (Top 100 ports)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nmap <span class="nt">-F</span> 192.168.1.10
</code></pre>

</div>



<p>Quick way to get useful info.</p>




<h3>
  
  
  5. Stealth Scan (SYN Scan)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nmap <span class="nt">-sS</span> 192.168.1.10
</code></pre>

</div>



<p>Very common in pentesting because it’s fast and less detectable.</p>




<h3>
  
  
  6. Get Service Info
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nmap <span class="nt">-sV</span> 192.168.1.10
</code></pre>

</div>



<p>This tries to detect what service is running on each open port<br>
(e.g., Apache, SSH version, MySQL version).</p>


<h3>
  
  
  7. OS Detection
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nmap <span class="nt">-O</span> 192.168.1.10
</code></pre>

</div>


<p>Tries to guess the operating system.</p>


<h3>
  
  
  8. Scan Without Ping (Useful for firewalled hosts)
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nmap <span class="nt">-Pn</span> 192.168.1.10
</code></pre>

</div>



<h2>
  
  
  The Most Common Nmap Flag (For Beginners): <code>-sS</code>
</h2>

<p>If you're wondering "which Nmap flag is the most used?", it’s <strong><code>-sS</code></strong> — the SYN (stealth) scan.<br>
It balances speed, accuracy, and low detection.</p>


<h1>
  
  
  How Nmap Works (Simple Explanation)
</h1>

<p>When you scan a port, you’re basically performing a tiny conversation:</p>

<ul>
<li>Send a TCP SYN packet → “Are you accepting connections?”</li>
<li>
<p>Get a reply:</p>

<ul>
<li>
<strong>SYN/ACK</strong> → Port is open</li>
<li>
<strong>RST</strong> → Port is closed</li>
<li>
<strong>No response / filtered</strong> → Firewall blocked it</li>
</ul>
</li>
</ul>

<p>This is the exact logic I'm trying to replicate while building my Go tool.</p>


<h1>
  
  
  Building My Own Nmap-like Tool in Go
</h1>

<p>I wanted to understand scanning deeper, so I started building a simple port scanner in Go.</p>

<p>Here’s a simplified version of checking if a port is open:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"fmt"</span>
    <span class="s">"net"</span>
    <span class="s">"time"</span>
<span class="p">)</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">host</span> <span class="o">:=</span> <span class="s">"192.168.1.10"</span>
    <span class="k">for</span> <span class="n">port</span> <span class="o">:=</span> <span class="m">1</span><span class="p">;</span> <span class="n">port</span> <span class="o">&lt;=</span> <span class="m">1024</span><span class="p">;</span> <span class="n">port</span><span class="o">++</span> <span class="p">{</span>
        <span class="n">address</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"%s:%d"</span><span class="p">,</span> <span class="n">host</span><span class="p">,</span> <span class="n">port</span><span class="p">)</span>
        <span class="n">conn</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">net</span><span class="o">.</span><span class="n">DialTimeout</span><span class="p">(</span><span class="s">"tcp"</span><span class="p">,</span> <span class="n">address</span><span class="p">,</span> <span class="n">time</span><span class="o">.</span><span class="n">Millisecond</span><span class="o">*</span><span class="m">200</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">==</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="n">fmt</span><span class="o">.</span><span class="n">Printf</span><span class="p">(</span><span class="s">"Port %d is open</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">port</span><span class="p">)</span>
            <span class="n">conn</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This is the <em>basic idea</em> behind an Nmap-like tool — check if a port accepts connections.</p>




<h1>
  
  
  Tips for Learning Nmap Effectively
</h1>

<ul>
<li>Start with one command per day</li>
<li>Always scan your own machines (legally safe)</li>
<li>Try different scan types and compare results</li>
<li>Read the built-in help (<code>nmap --help</code>)</li>
<li>Build your own small port scanner to understand the basics</li>
</ul>




<h1>
  
  
  Final Thoughts
</h1>

<p>Nmap may look intimidating, but once you understand the basics, it becomes one of the most powerful and fun tools in cybersecurity.</p>

