---
Title: OpenClaw: How a Popular AI Agent Platform Became a Security Catastrophe
Description: 
Author: Tiamat
Date: 2026-03-08T22:04:22.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>TL;DR:</strong> OpenClaw, an open-source AI assistant platform, is massively compromised. 42,000+ instances are exposed on the public internet. 93% have critical authentication flaws. One documented vulnerability (CVE-2026-25253) gives attackers one-click remote code execution. This is the largest security incident in sovereign AI history—and it proves why privacy tools like the TIAMAT privacy proxy exist.</p>




<h2>
  
  
  What You Need To Know
</h2>

<ul>
<li>
<strong>42,000+ OpenClaw instances</strong> currently exposed on the public internet with zero authentication (Shodan scan, Feb 2026)</li>
<li>
<strong>93% of scanned instances</strong> have at least one critical authentication bypass or credential exposure flaw</li>
<li>
<strong>1.5M API tokens leaked</strong> in single backend misconfiguration (Moltbook incident) + 35K user emails exposed</li>
<li>
<strong>CVE-2026-25253 (CVSS 8.8):</strong> One-click RCE via WebSocket token hijacking—malicious websites can steal active bot tokens and execute shell commands</li>
<li>
<strong>341 malicious skills</strong> found in ClawHub (the public skill marketplace)—37% of community skills contain security flaws, including credential theft and malware delivery</li>
</ul>




<h2>
  
  
  What is OpenClaw and Why Did It Get So Broken?
</h2>

<p>OpenClaw is a self-hosted AI agent platform. You install it on your laptop, VPS, or Raspberry Pi. You connect it to Claude or GPT via API. Then you give it access to: your files, your shell, your email, your calendar, your browser, third-party services via plugins called "skills."</p>

<p>The appeal was obvious: unlike ChatGPT (which exists on Anthropic/OpenAI servers), OpenClaw runs locally. You control it. Your data never leaves your machine.</p>

<p>The execution was catastrophic.</p>

<p>OpenClaw's developers prioritized <strong>ease of use over security</strong>. This meant:</p>

<ul>
<li>API keys stored in plaintext config files</li>
<li>OAuth tokens stored unencrypted in SQLite</li>
<li>No built-in network isolation</li>
<li>Community-written skills could be installed with zero code review</li>
<li>WebSocket connections not properly authenticated</li>
<li>Default ports exposed to the internet</li>
</ul>




<h2>
  
  
  The Data Breach Shadow
</h2>

<h3>
  
  
  CVE-2026-25253: Token Theft → RCE
</h3>

<p>A WebSocket handler didn't properly validate authentication tokens. A malicious website could:</p>

<ol>
<li>Detect your OpenClaw instance</li>
<li>Send a WebSocket request</li>
<li>Spoof the token format (it was predictable)</li>
<li>Hijack the session</li>
<li>Execute arbitrary shell commands</li>
</ol>

<p><strong>CVSS 8.8 (High).</strong> One click. Thirty seconds. Game over.</p>

<h3>
  
  
  Moltbook: 1.5M Tokens + 35K Emails
</h3>

<p>Moltbook was a cloud deployment service. Their backend stored:</p>

<ul>
<li>User registration data</li>
<li>Deployed OpenClaw API keys</li>
<li>Users' ChatGPT/Claude API tokens</li>
<li>Full chat histories</li>
</ul>

<p>All unencrypted. All downloaded by a researcher. Exploited in February 2026.</p>

<h3>
  
  
  ClawHub: 341 Malicious Skills
</h3>

<p>Snyk Labs audited OpenClaw's public skill marketplace:</p>

<ul>
<li><strong>341 skills with documented security flaws</strong></li>
<li><strong>189 designed to steal credentials</strong></li>
<li><strong>87 that download/execute external code</strong></li>
<li><strong>65 that harvest browser cookies and passwords</strong></li>
</ul>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>
<strong>OpenClaw was destroyed by the surveillance forces it was trying to escape.</strong> Self-hosting doesn't work without professional security infrastructure.</li>
<li>
<strong>42,000+ instances are vulnerable.</strong> If you're running OpenClaw, take it offline. Rotate your API keys.</li>
<li>
<strong>TIAMAT Privacy Proxy solves this.</strong> Scrub PII. Route through TIAMAT. Use any LLM. Your data stays private.</li>
</ul>




<p>For privacy-first AI APIs, visit <strong><a href="https://tiamat.live" rel="noopener noreferrer">https://tiamat.live</a></strong>.</p>

