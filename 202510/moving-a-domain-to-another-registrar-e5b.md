---
Title: Moving a Domain to Another Registrar
Description: 
Author: Aidas Bendoraitis
Date: 2025-10-17T21:52:41.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Situation
</h2>

<p>The domain for my first SaaS project <a href="https://www.1st-things-1st.com" rel="noopener noreferrer">1st-things-1st.com</a> was registered with GoDaddy. Even though the whole project was already running under my company’s name, I never really bothered to move the domain to my company’s account at Namecheap.</p>

<p>Last week I noticed that the domain was about to expire, and I thought, alright, time to finally do it.</p>

<p>I had never transferred a domain before, so I wasn’t sure how it would go or whether I could pull it off without any downtime. Here’s how it went.</p>

<h2>
  
  
  The Story
</h2>

<p>Namecheap has this feature called “Transfer to Us.” You just follow a few simple steps: request a transfer for your domain, enter a one-time Auth code (also called as EPP - Extensible Provisioning Protocol - code) from another registrar to confirm you’re the owner, and pay for another year.</p>

<p>At GoDaddy’s side, there was a whole confirmation process to make sure I really wanted to transfer the domain and understood it couldn’t be undone. Once that was done, I got the code I needed for Namecheap.</p>

<p>Then came the waiting part. It took almost a week to get the confirmation that the transfer was successful, which was expected according to their help docs. As soon as I got the notification, I jumped into my Namecheap settings to check if everything looked right.</p>

<p>Unfortunately, the domain records weren’t migrated automatically. Only the ownership was transferred. The nameserver settings in Namecheap were still pointing to GoDaddy’s servers.</p>

<p>And since GoDaddy no longer listed the domain or its DNS records, I had to recover those values myself.</p>

<p>I needed to extract the following records for each subdomain:</p>

<ul>
<li>A – IPv4 Address</li>
<li>CNAME – Canonical Name</li>
<li>MX – Mail Exchange</li>
<li>TXT – Text</li>
</ul>

<p>I did it from the command line like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>dig 1st-things-1st.com ANY <span class="o">&gt;</span> 1st-things-1st.txt
<span class="nv">$ </span>dig www.1st-things-1st.com ANY <span class="o">&gt;&gt;</span> 1st-things-1st.txt
<span class="nv">$ </span>dig my.1st-things-1st.com ANY <span class="o">&gt;&gt;</span> 1st-things-1st.txt
<span class="nv">$ </span>dig our.1st-things-1st.com ANY <span class="o">&gt;&gt;</span> 1st-things-1st.txt
<span class="nv">$ </span>dig apps.1st-things-1st.com ANY <span class="o">&gt;&gt;</span> 1st-things-1st.txt
<span class="nv">$ </span>dig analytics.1st-things-1st.com ANY <span class="o">&gt;&gt;</span> 1st-things-1st.txt
</code></pre>

</div>



<p>Alternatively, you can look up WHOIS information on sites like <a href="https://who.is/dns/1st-things-1st.com" rel="noopener noreferrer">who.is</a>.</p>

<p>Once I had all the records, I switched to Namecheap’s nameservers (Namecheap BasicDNS) and added everything manually.</p>

<p>The website was down for about 30 to 60 minutes, depending on where in the world the visitor was. Some downtime was unavoidable, but overall, the transfer went through successfully.</p>

<h2>
  
  
  The Learnings
</h2>

<p>The whole thing could have been smoother if I had known what to expect. Here’s how I’d do it next time:</p>

<ol>
<li>Save all domain records to a local file before starting.</li>
<li>Get the Auth/EPP code from the old registrar.</li>
<li>Request the transfer at the new registrar.</li>
<li>Wait for the transfer to complete.</li>
<li>Pick the best time for downtime and let users know in advance.</li>
<li>Switch to the new nameservers and add all records manually.</li>
</ol>

