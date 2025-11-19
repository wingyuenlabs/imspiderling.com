---
Title: Cloudflare went down yesterday. My monitoring lied. So I built this.
Description: 
Author: Yibnum
Date: 2025-11-19T21:44:34.000Z
Robots: noindex,nofollow
Template: index
---
<p>Yesterday’s Cloudflare outage wasted a few hours of my time — not because the outage was confusing, but because my monitoring stack gave me zero context about what was actually failing.</p>

<p>Everything lit up red. Every alert fired.<br>
But nothing told me whether the problem was:</p>

<p>my origin</p>

<p>Cloudflare’s edge</p>

<p>DNS</p>

<p>SSL</p>

<p>routing</p>

<p>My servers were completely fine the whole time.</p>

<p>The real issue ended up being Cloudflare’s Bot Management system (a feature file doubled in size and tripped them up).</p>

<p>The bigger discovery:<br>
most monitoring tools cannot tell the difference between an origin outage and a CDN/edge outage.</p>

<p>So I built a simple tool today to diagnose exactly that:</p>

<p>👉 <a href="https://stayup.dev" rel="noopener noreferrer">https://stayup.dev</a></p>

<p>Paste a URL and it checks:</p>

<p>origin health</p>

<p>Cloudflare/Vercel/AWS edge</p>

<p>DNS</p>

<p>SSL expiry</p>

<p>CDN failure patterns</p>

<p>I built this out of frustration, but if you’re interested I’d love to hear how your monitoring handled the outage yesterday.</p>

