---
Title: Technical Deep Dive: How we Created a Security-hardened 1-Click Deploy OpenClaw
Description: 
Author: Jess Lulka
Date: 2026-02-24T22:11:29.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This article was originally written by Freddie Rice (Staff Product Security Engineer)</em></p>

<p><a href="https://www.digitalocean.com/resources/articles/what-is-moltbot" rel="noopener noreferrer">OpenClaw, an open source AI assistant</a> (<a href="https://x.com/openclaw/status/2017103710959075434" rel="noopener noreferrer">recently renamed from Moltbot</a>, and earlier Clawdbot), has exploded in popularity over the last few days, and at DigitalOcean we immediately wondered “how can we enable more people to try this new technology safely and easily?” We noticed that there was a lot of interest by folks looking to use this software, but also that there was concern around the security of the open source software, especially when connecting it directly to users’ own machines. We dug in to find a way to deliver this software to our customers as fast as possible, as easily as possible and as safe as possible.</p>

<p>At DigitalOcean, our <a href="https://marketplace.digitalocean.com/apps/moltbot" rel="noopener noreferrer">1-Click Deploy OpenClaw</a> (formerly 1-Click Deploy Moltbot) through our Marketplace enables us to package the latest and greatest software configured for our Droplet® server, and make it easily available to customers. Creating our 1-Click Deploy OpenClaw was the natural next step in getting this to our customers.</p>

<p>Toying around with OpenClaw on a local machine is fun, but it could severely impact the ability to deploy and use the software for longer term use and may not meet the safe environment that you need. Some of the benefits to deploying on DigitalOcean include:</p>

<ul>
<li>Always available – the service is available to customers via the web</li>
<li>Easy to connect to it – Droplets have a static ip address</li>
<li>Vertical scalability – scale up CPUs, memory, and disk storage with higher workloads</li>
<li>Cognitive overload – start with basic configs, tweak the ones that matter to you</li>
</ul>

<p>We made a lot of changes as we built the 1-Click Deploy OpenClaw, but the main elements we focused on were</p>

<ul>
<li>How do we communicate with the service safely?</li>
<li>How do we keep the agentic code isolated from the rest of the system?</li>
<li>How do we prevent attacks from the wider internet?</li>
</ul>

<p>All of that while providing a straightforward deployment UX to our customers! Let’s dig in…</p>

<h3>
  
  
  Delivering an Image with Safe Defaults
</h3>

<p>Our priority in creating a 1-Click Deploy OpenClaw on our Droplet was twofold: First, speed, as we wanted to get something out quickly to our users. Second was providing a solution that provided additional security benefits. These are the actions we took to meet those goals:</p>

<h3>
  
  
  Keeping deployments consistent (DevOps)
</h3>

<p>We saw that there are multiple ways to deploy the software – we chose the most consistent path, which was picking a stable release from the Git repository on GitHub, pulling it and building from there.</p>

<p>Why not pull the latest and greatest from main? Changes are happening at a rapid pace, which is awesome for feature development but can come at the expense of stability. Depending on the minute we build our 1-click image, we could get a working version or a broken version.</p>

<p>So we make sure that we can deliver the latest stable version.</p>

<h3>
  
  
  TLS (Keep communications safe and auditable)
</h3>

<p>Once we had a Packer image that we could iterate on, we applied our security best practices for the 1-clicks to set up TLS. This is a crucial step to make sure that our customers can communicate with the bot in a safe way that doesn’t allow eavesdropping.</p>

<p>Our best practices consist of using Caddy as a reverse proxy with a TLS certificate issued by LetsEncrypt. Caddy ensures that the service is deployed externally is the service we want to publish and provides a safe channel with which to serve it. Furthermore, Caddy outputs logs to a location that can be audited after the fact, allowing the end user to see how their service is actually being used.</p>

<p>A new UX improvement we added to this image is seamless TLS configuration with LetsEncrypt via IP addresses without requiring a domain name! While OpenClaw spins up, Caddy is requesting a new certificate on your behalf, no configuration required.</p>

<h3>
  
  
  Authz (Gateway Key + Pairing)
</h3>

<p>How do we know that the requests are coming from you? We have a OpenClaw gateway key in place to make sure that the user who is supposed to use the platform is the correct one.</p>

<p>Next, we leaned into a feature that OpenClaw provides called “Paring” – this exists to make sure that the devices that are going to communicate with the main server are the trusted ones.</p>

<h3>
  
  
  Sandboxing (keep safe from Agents)
</h3>

<p>Part of the configuration is an Anthropic / OpenAI / Model key – these are sensitive pieces of material that are required in order to allow the software to function! So how do we let agents that can run arbitrary code on the machine, not read and abuse these tokens?</p>

<p>Furthermore, how do we stop the agents from potentially destroying the machine itself?</p>

<p>Luckily, there is a configuration available that puts the agent deployments into their own containers. If an agent blows up, it will destroy its own ephemeral docker container, but the host filesystem will still be safe from incorrect agentic modifications.</p>

<h3>
  
  
  Safe Defaults
</h3>

<p>These boxes are taking the best configurations that we implement for all of our 1-clicks, including but not limited to:</p>

<p><strong>Fail2ban</strong> – Make sure that the background noise of the internet doesn’t cause disruptions to your droplet. It does this by monitoring the logs of failed requests to the system and dynamically updating firewall rules to block known bad patterns on the internet.</p>

<p><strong>Unattended upgrades</strong> – We want to make sure that your Droplet is always up to date. We have Ubuntu configured with unattended upgrades that periodically will check for vulnerable packages and automatically patch them.</p>

<h2>
  
  
  Deployment Constraints and Upcoming Features
</h2>

<p>To ensure a stable and repeatable installation, we utilize Packer for our image provisioning; however, during testing, we found that smaller Droplet configurations consistently encountered out-of-memory errors during the snapshot creation process. While this currently necessitates a minimum $24/month Droplet size to match the snapshot’s disk and memory requirements, we chose to prioritize getting this tool into your hands today rather than delaying for further optimization. We are already iterating on the image to reduce its footprint and support lower-cost tiers, and in the spirit of transparency, we have <a href="https://github.com/digitalocean/droplet-1-clicks" rel="noopener noreferrer">made our Packer scripts public</a> so you can audit the provisioning process and gain confidence in the one-click experience. We are also working to quickly add support for all DigitalOcean Gradient AI models, including OpenAI, add auto provisioning of Gradient AI API Key and injecting for the user, and more updates as OpenClaw evolves over time.</p>

<h3>
  
  
  After deploy (make it yours!)
</h3>

<p>1-Click Deploy OpenClaw is a great launch point, but OpenClaw is infinitely customizable once up and running in the Droplet. Choose which messaging platforms are the best fit for your workflows, and get chatting.</p>

<h2>
  
  
  Get started with the 1-Click Deploy OpenClaw
</h2>

<p>Get started with the <a href="https://marketplace.digitalocean.com/apps/moltbot" rel="noopener noreferrer">1-Click Deploy OpenClaw by visiting the Marketplace</a>, and <a href="https://www.digitalocean.com/community/tutorials/how-to-run-moltbot" rel="noopener noreferrer">follow this tutorial</a> for step by step instructions on how to get started.</p>

