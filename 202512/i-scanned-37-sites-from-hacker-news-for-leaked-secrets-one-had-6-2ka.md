---
Title: I scanned 37 sites from Hacker News for leaked secrets. One had 6.
Description: 
Author: Bren Builds
Date: 2025-12-16T21:29:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most security tools scan your GitHub repo looking for accidentally committed secrets. That's useful. But it misses something...</p>

<p>What about the secrets that are deployed?</p>

<p>I built a scanner that checks your live site. It downloads your JavaScript bundles and runs regex patterns against them. The same thing an attacker would do if they opened DevTools and clicked View Source.</p>

<p>I pointed it at 37 sites from the Hacker News front page over two days. Mostly indie projects and startups.</p>

<p><strong>The results</strong></p>

<p>One site had 6 real secrets in their production JavaScript:</p>

<ul>
<li>1 AWS credential</li>
<li>5 API keys (Stripe, SendGrid, and others)</li>
</ul>

<p>The site launched recently. The founder probably has no idea.</p>

<p><strong>How secrets end up in production JavaScript</strong></p>

<p>This happens more than people think. Here's a common example.</p>

<p>You have an environment variable:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>NEXT_PUBLIC_STRIPE_KEY=sk_live_abc123...
</code></pre>

</div>



<p>Or maybe you're using Vite:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>VITE_OPENAI_KEY=sk-abc123...
</code></pre>

</div>



<p>These frameworks inline environment variables that match certain prefixes into your client bundle at build time. It's documented behavior. But developers forget which keys are public and which are secret.</p>

<p>After the build, your <code>main.js</code> or <code>app.bundle.js</code> contains something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiKey</span><span class="p">:</span> <span class="dl">"</span><span class="s2">sk_live_abc123realkey</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">region</span><span class="p">:</span> <span class="dl">"</span><span class="s2">us-east-1</span><span class="dl">"</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Anyone can find it. Open DevTools, go to Sources, search for "sk_live" or "api_key". Done.</p>

<p><strong>The fix isn't in your repo anymore</strong></p>

<p>Here's the problem with repo scanners. You committed a secret six months ago. You realized the mistake and removed it. You rotated the key. Great.</p>

<p>But did you redeploy every environment? Is that old bundle still sitting in a CDN cache somewhere? Is your staging site still running the old build?</p>

<p>Repo scanners check your current code. They don't check what's actually running.</p>

<p><strong>Not every detection is a real problem</strong></p>

<p>I want to be clear about this. The scanner uses pattern matching. It flags things that look like API keys.</p>

<p>Some of those are fine. Firebase client configurations are meant to be public. Stripe publishable keys (starting with <code>pk_</code>) are designed for frontend use. Google Maps API keys are often restricted by domain.</p>

<p>The tool flags patterns worth checking. You decide what's actually a problem.</p>

<p><strong>Try it yourself</strong></p>

<p>The scanner is free. No signup. One click.</p>

<p><a href="https://domainoptic.com/audit" rel="noopener noreferrer">https://domainoptic.com/audit</a></p>

<p>If you want to see what a detection looks like first, scan <code>wirier.com</code>. I built it as a demo target with fake secrets planted in the code.</p>

<p>If you find something on your own site, congrats!</p>

