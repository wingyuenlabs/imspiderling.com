---
Title: My Cybersecurity Homelab: A Hands-On Journey into Defensive and Offensive Operations
Description: 
Author: shyn
Date: 2026-01-03T21:30:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>My Cybersecurity Homelab: A Hands-On Journey into Defensive and Offensive Operations<br>
As an aspiring cybersecurity professional, I believe that practical experience is just as crucial as theoretical knowledge. That's why I've dedicated time to building and continuously evolving my own homelab – a personal sandbox where I can experiment, learn, and sharpen my skills in both defensive and offensive security operations.</p>

<p>This post will walk you through my current homelab setup, highlighting the tools and technologies I'm using to simulate real-world scenarios, monitor my infrastructure, and hone my penetration testing abilities.</p>

<p><strong>The Foundation: Hardware and Network</strong><br>
My homelab is built upon a foundation of readily available hardware, proving that you don't need enterprise-grade equipment to gain valuable experience.</p>

<p><strong>Hardware Components:</strong></p>

<ul>
<li>Two Mini PCs: These serve as the workhorses of my lab, hosting various virtual machines and services.</li>
<li>Raspberry Pi: A versatile single-board computer, perfect for lightweight services.</li>
<li>Network Switch: The central nervous system, connecting all components and allowing for network segmentation experiments.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkddhfm6ge24wquql5mv0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkddhfm6ge24wquql5mv0.png" alt=" " width="800" height="594"></a></p>

<h2>
  
  
  Defensive Operations: Monitoring, SIEM, and Threat Detection
</h2>

<p>A significant part of my homelab focuses on defensive security. I want to understand how to monitor systems, detect anomalies, and respond to potential threats.</p>

<p><strong>Raspberry Pi: Jellyfin Media Server</strong><br>
While primarily for personal media consumption, running Jellyfin on my Raspberry Pi provides a practical environment for practicing service hardening, network segmentation, and monitoring a publicly accessible (within my home network) application. It's a great "live target" for practicing security controls.</p>

<p><strong>Mini PC 1: Wazuh for Endpoint and Container Monitoring</strong><br>
On my first Mini PC, I've deployed Wazuh, an open-source security platform that unifies XDR and SIEM capabilities.</p>

<ul>
<li>Endpoint Security: Wazuh agents are installed on my other lab machines and VMs, providing crucial security visibility.</li>
<li>Container Monitoring: I've configured Wazuh to monitor the status and security of my various Docker containers. This allows me to receive real-time alerts on container events, file integrity changes, and potential vulnerabilities. The custom dashboard provides a centralized view of my containerized environment's health and security posture.</li>
</ul>

<p>This setup is invaluable for learning about:</p>

<ul>
<li>Log Management: Collecting and analyzing logs from diverse sources.</li>
<li>File Integrity Monitoring (FIM): Detecting unauthorised changes to critical system files.</li>
<li>Vulnerability Detection: Identifying known vulnerabilities in my systems and applications.</li>
<li>Real-time Alerting: Setting up rules and alerts for suspicious activities.</li>
</ul>

<h2>
  
  
  Offensive Operations: Penetration Testing and Threat Simulation
</h2>

<p>On the other side of the coin, my homelab provides a safe and isolated environment to practice offensive security techniques. This helps me understand the attacker's mindset, which is crucial for building stronger defenses.</p>

<p><strong>Mini PC 2: Proxmox Virtualization Host</strong><br>
This is where the magic of virtualization happens. Proxmox VE (Virtual Environment) allows me to run multiple virtual machines, creating isolated environments for different offensive and defensive tools.</p>

<p><strong>Within Proxmox, I host:</strong></p>

<ul>
<li>Splunk (SIEM): Another powerful Security Information and Event Management (SIEM) solution. Learning both Wazuh and Splunk gives me a broader understanding of log aggregation, correlation, and threat hunting in different platforms. This VM is dedicated to ingesting logs from various sources within my lab for in-depth analysis.</li>
<li>Pi-hole: A network-wide ad blocker and DNS sinkhole. While seemingly simple, it's an excellent tool for understanding DNS traffic, network filtering, and can even be used to block malicious domains in a lab environment.</li>
<li>Local LLM for Red Team Testing: This is an exciting experimental area! I'm running a local Large Language Model (LLM) to explore its potential in red team operations. This includes generating phishing email drafts, crafting social engineering scenarios, or even assisting with code analysis for exploit development (all within the confines of my lab, of course!). This helps me understand the evolving landscape of AI in cybersecurity.</li>
<li>Kali Linux: The go-to distribution for penetration testing. This VM is my primary offensive workstation, equipped with a vast array of tools.</li>
<li>Metasploit: Integrated within Kali, Metasploit Framework is essential for exploit development, payload generation, and post-exploitation. I use it to test vulnerabilities against intentionally vulnerable target machines within my lab.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fba7lx8v7rhpzfyjp6wfy.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fba7lx8v7rhpzfyjp6wfy.jpg" alt=" " width="800" height="450"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9f938lxhxnt5of2hkc73.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9f938lxhxnt5of2hkc73.png" alt=" " width="800" height="999"></a></p>

<h2>
  
  
  What's Next?
</h2>

<p>My homelab is a living, breathing project that I continuously expand and refine. Future plans include:</p>

<ul>
<li>Network Segmentation: Implementing VLANs to create more isolated environments for different trust levels.</li>
<li>Active Directory Environment: Setting up a Windows Server with Active Directory to practice domain-based attacks and defenses.</li>
<li>Cloud Integration: Exploring hybrid cloud scenarios by integrating cloud-based services (e.g., AWS, Azure) into my lab.</li>
<li>Automation: Scripting common tasks and deployments using Ansible or Terraform.</li>
<li>Container Orchestration Security: Delving deeper into Kubernetes security.</li>
</ul>

<blockquote>
<p>This homelab is more than just a collection of hardware and software; it's a testament to my passion for cybersecurity and my commitment to continuous learning. I believe this practical experience will be invaluable as I transition into a professional cybersecurity role.</p>
</blockquote>

<p>Thanks for reading! Feel free to ask any questions or share your own homelab experiences in the comments below.</p>

