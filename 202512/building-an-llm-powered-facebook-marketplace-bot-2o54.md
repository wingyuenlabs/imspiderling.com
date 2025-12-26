---
Title: Building an LLM-powered Facebook Marketplace Bot
Description: 
Author: Isaac Addis
Date: 2025-12-26T21:23:50.000Z
Robots: noindex,nofollow
Template: index
---
<p>I was interested in trying out bot development in Ethiopia’s emerging tech industry by creating a bot that would monitor product listings on Facebook Marketplace. I succeeded in my attempts to pass through bot detection using a simple heuristic: mimic a real human. As I eventually discarded this bot (Facebook Marketplace already has a “Notify Me” feature), I am open to sharing how I developed this.</p>

<p>Please note that this does violate Facebook ToS.</p>

<h2>
  
  
  System Overview
</h2>

<p>Here’s a summary of our tech stack:</p>

<ul>
<li>Digital Ocean VPS running Ubuntu</li>
<li>Better Stack for logs (we had to use raw HTTPS requests with helper functions – this wasn’t working with the <code>pino</code> library)</li>
<li>
<code>pm2</code> for process management (this makes it easy to run a script as an always-on background task)</li>
<li>Slack for alerts</li>
<li>
<code>better-sqllite3</code> to track alerted listings (to avoid alerting about the same product twice)</li>
<li>OpenAI /chat/completions API with gpt-4o-mini for filtering product listings</li>
<li>TypeScript + OOP (I’ve found that bot development works really well with OOP)</li>
</ul>

<p>Something interesting we did was use LLMs to filter out listings unrelated to our product query. For example, when the bot searches for "iPhone 15"s, the LLM filters out listings for "iPhone 15 Pro"s. We used gpt-4o-mini for this because of its high speed and low cost and had perfect results.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgwkf6iindi5r1jyg8coz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgwkf6iindi5r1jyg8coz.png" alt="Bot alert showing an iPhone 13 Pro Max deal" width="800" height="448"></a></p>

<h2>
  
  
  Technical Challenges + Solutions
</h2>

<h3>
  
  
  Captcha Harvesting
</h3>

<p>Note: being signed-in is not required for monitoring Facebook Marketplace</p>

<p>Captchas prevent signing in directly through the bot. To get past this, I created a captcha harvester script that opened Puppeteer on my local computer and saved subsequent authentication cookies onto my local machine. With the <code>scp</code> command, I moved the cookies onto my VPS. This works for getting past login walls on Facebook.</p>

<h3>
  
  
  Proxy Rotation
</h3>

<p>I’ve noticed that I needed to rotate proxies from our ProxyManager class when: <br>
No amount of captchas would get me signed-in<br>
I would suddenly get a login wall when trying to monitor products</p>

<h3>
  
  
  Residential Proxies
</h3>

<p>I used residential proxies to make it seem like traffic was coming from realistic IPs.</p>

<h3>
  
  
  Puppeteer Stealth Mode
</h3>

<p>LLMs suggest this. Something crucial from this plugin is the obscuring of <code>navigator.webdriver</code>, something that identifies Puppeteer/Playwright usage.</p>

<h3>
  
  
  User Agent
</h3>

<p>I configured the browser user agent to Windows: <code>await page.setUserAgent(Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36);</code> despite running on a Linux VPS. </p>

<h2>
  
  
  Tips
</h2>

<ul>
<li><p>Create scripts with LLMs to debug things. For example, I needed to create a script to check if Facebook recognized my authentication cookies with the Puppeteer settings we use in the bot<br>
Avoid alerting on obvious scams by filtering out prices that seem too low</p></li>
<li><p>Approximating what a human user would do proved to be a good tactic for getting past bot protection on Facebook Marketplace.</p></li>
</ul>

<p>Techniques used here to get around bot protection should be applicable to other services.</p>

