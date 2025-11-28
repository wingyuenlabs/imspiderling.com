---
Title: Building My Personal Website: From Idea to Automated Deployment (Part 1)
Description: 
Author: Danylo Mikula
Date: 2025-11-28T21:58:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>The idea of creating my own personal website—a place where I could share projects I'm working on and document my technical journey—has been on my mind for a long time. But as with many personal projects, it kept getting pushed aside. Finally, I found the time, and here it is: <a href="https://mikula.dev" rel="noopener noreferrer">mikula.dev</a>. In this post, I want to share how I built it, what tools I chose, and why.</p>

<h2>
  
  
  Choosing the Right Static Site Generator
</h2>

<p>When looking for a site generator, I had a few requirements in mind: it needed to be simple yet flexible, fast, and shouldn't require hours of configuration just to get started. After evaluating several options, I settled on <a href="https://gohugo.io/" rel="noopener noreferrer">Hugo</a>.</p>

<p>Hugo is one of the fastest static site generators out there. Written in Go, it can build thousands of pages in seconds. But speed isn't the only advantage—it generates pure static HTML files, which makes hosting incredibly straightforward. No databases, no server-side processing, no complex runtime dependencies. Just files that can be served by any web server.</p>

<p>The fact that Hugo outputs static files also brings security benefits—there's simply no dynamic attack surface. Combined with its extensive templating capabilities and active community, it was an easy choice.</p>

<h2>
  
  
  Finding the Perfect Theme
</h2>

<p>I didn't want to spend weeks developing my own theme from scratch. Instead, I looked for something that matched my aesthetic preferences and could be customized easily. I found <a href="https://github.com/panr/hugo-theme-terminal" rel="noopener noreferrer">Terminal</a> by Radek Kozieł, and it was exactly what I was looking for.</p>

<p>The theme has a clean, retro terminal-inspired look with beautiful syntax highlighting powered by Chroma. It uses <a href="https://github.com/tonsky/FiraCode" rel="noopener noreferrer">Fira Code</a> as the default monospace font, is fully responsive, and supports customizable color schemes. While it covered most of my needs out of the box, I did extend it with some additional functionality—like better post organization and a dedicated resume page.</p>

<h2>
  
  
  Where to Host?
</h2>

<p>Since Hugo generates static files, I had several hosting options to consider: GitHub Pages, AWS S3 with CloudFront, or a small cloud server. Each has its merits, but I went with a dedicated server on <a href="https://www.hetzner.com/cloud" rel="noopener noreferrer">Hetzner Cloud</a>.</p>

<p>Why? Flexibility. While GitHub Pages and S3 are excellent for simple static hosting, having my own server gives me complete control over the infrastructure. I can configure custom caching rules, set up rate limiting, add custom header and run additional services if needed. Plus, Hetzner offers excellent performance at very competitive prices.</p>

<h2>
  
  
  Caddy: The Modern Web Server
</h2>

<p>For the web server, I evaluated a few options—nginx, Apache, and Caddy. I chose <a href="https://caddyserver.com/" rel="noopener noreferrer">Caddy</a> for several compelling reasons.</p>

<p>First, automatic HTTPS. Caddy handles SSL certificate provisioning and renewal through Let's Encrypt completely automatically. No more manual certificate management, no cron jobs for renewal, no forgetting to renew and having your site go down. It just works.</p>

<p>Second, simplicity. Caddy's configuration format (the Caddyfile) is remarkably straightforward compared to nginx or Apache configurations. A basic site configuration can be just a few lines, yet it still offers powerful customization options when you need them.</p>

<p>I'm also using a custom Caddy build with additional plugins: <a href="https://github.com/caddy-dns/cloudflare" rel="noopener noreferrer">caddy-dns/cloudflare</a> for DNS-01 ACME challenges (so I can get certificates even before DNS propagation completes) and <a href="https://github.com/mholt/caddy-ratelimit" rel="noopener noreferrer">caddy-ratelimit</a> to protect against bots and abuse.</p>

<h2>
  
  
  DNS and Security with Cloudflare
</h2>

<p>For DNS management, I'm using <a href="https://www.cloudflare.com/" rel="noopener noreferrer">Cloudflare</a>. But it's not just about DNS—I have Cloudflare Proxy enabled, which means all traffic to my site goes through Cloudflare's network first. This provides several benefits: DDoS protection, CDN caching, and most importantly, it hides my server's real IP address from the public.</p>

<p>To take security a step further, I configured firewall rules directly in Hetzner Cloud to only allow incoming HTTP/HTTPS traffic from Cloudflare's IP ranges. This means even if someone discovers my server's actual IP address, they can't connect to the web server directly—all requests must go through Cloudflare. This setup effectively creates an additional security layer and ensures that all traffic benefits from Cloudflare's protection.</p>

<p>Cloudflare publishes their IP ranges publicly, so keeping the firewall rules updated is straightforward. Combined with Caddy's rate limiting, this gives me a solid defense-in-depth approach without adding complexity to the daily operations.</p>

<h2>
  
  
  Infrastructure as Code with Terraform
</h2>

<p>As someone who believes in automating everything, I needed proper Infrastructure as Code for my cloud setup. I looked for existing Terraform modules for Hetzner Cloud but didn't find anything that met my standards for flexibility and maintainability. So I built my own.</p>

<p>I created a set of reusable Terraform modules that cover the essential Hetzner Cloud resources:</p>

<ul>
<li>
<a href="https://github.com/danylomikula/terraform-hcloud-server" rel="noopener noreferrer">terraform-hcloud-server</a> — Multi-server management with support for private networks and firewalls.</li>
<li>
<a href="https://github.com/danylomikula/terraform-hcloud-network" rel="noopener noreferrer">terraform-hcloud-network</a> — VPC and subnet management</li>
<li>
<a href="https://github.com/danylomikula/terraform-hcloud-firewall" rel="noopener noreferrer">terraform-hcloud-firewall</a> — Firewall rules configuration</li>
<li>
<a href="https://github.com/danylomikula/terraform-hcloud-ssh-key" rel="noopener noreferrer">terraform-hcloud-ssh-key</a> — SSH key management with support for key generation</li>
</ul>

<p>These modules are designed to work together seamlessly while remaining flexible enough for various use cases. They're all open source and available on both GitHub and the Terraform Registry.</p>

<h2>
  
  
  Configuration Management with Ansible
</h2>

<p>With infrastructure sorted out, I needed a way to automate the actual server configuration and site deployment. For this, I created an Ansible collection: <a href="https://github.com/danylomikula/ansible-hugo-deploy" rel="noopener noreferrer">ansible-hugo-deploy</a>.</p>

<p>This collection handles the complete deployment pipeline:</p>

<ul>
<li>Installing and configuring Caddy with custom builds (including the Cloudflare DNS and rate limiting plugins)</li>
<li>Generating SSH deploy keys for secure repository access</li>
<li>Cloning the Hugo site from GitHub</li>
<li>Building the site with Hugo</li>
<li>Obtaining and managing SSL certificates</li>
<li>Configuring rate limiting and security headers</li>
<li>Setting up automated content updates via systemd timer</li>
</ul>

<p>The systemd timer runs daily, pulling the latest changes from the repository and rebuilding the site. This means I can just push a new post to GitHub, and within a day (or I can trigger it manually), the site updates automatically. No SSH-ing into the server, no manual deployments.</p>

<h2>
  
  
  The Complete Picture
</h2>

<p>Here's how everything fits together:</p>

<ol>
<li>
<strong>Terraform</strong> provisions the infrastructure on Hetzner Cloud—server, network, firewall rules (allowing only Cloudflare IPs), and SSH keys</li>
<li>
<strong>Ansible</strong> configures the server—installs Caddy, Hugo, sets up the deployment pipeline</li>
<li>
<strong>Hugo</strong> generates the static site from Markdown content</li>
<li>
<strong>Caddy</strong> serves the site with automatic HTTPS, compression, and rate limiting</li>
<li>
<strong>Cloudflare</strong> handles DNS, proxies all traffic, provides DDoS protection and CDN caching</li>
<li>
<strong>systemd timer</strong> keeps the content automatically updated</li>
</ol>

<p>The entire setup—from bare server to fully functional website—takes about 15 minutes. And once it's running, I never need to touch the server for content updates. Write a post in Markdown, push to GitHub, and the site updates itself.</p>

<h2>
  
  
  What's Next?
</h2>

<p>In the second part of this series, I'll dive deeper into the technical details of both the Terraform modules and the Ansible collection. I'll walk through the code, explain the design decisions, and show how you can use these tools for your own projects.</p>

<p>All the code is open source and available on my GitHub:</p>

<ul>
<li><a href="https://github.com/danylomikula?tab=repositories&amp;q=terraform-hcloud" rel="noopener noreferrer">Terraform Hetzner Cloud Modules</a></li>
<li><a href="https://github.com/danylomikula/ansible-hugo-deploy" rel="noopener noreferrer">Ansible Hugo Deploy Collection</a></li>
</ul>

<p>Feel free to use them, contribute, or just take inspiration for your own automation journey!</p>

