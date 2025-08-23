---
Title: Why Do AI Crawlers Keep Hitting robots.txt Instead of My Content?
Description: 
Author: Dom Sipowicz
Date: 2025-08-23T21:58:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>Over the past weeks I’ve been monitoring traffic from AI crawlers like OpenAI’s GPTBot, oai-searchbot and ClaudeBot. The data (see screenshots below) raises some interesting questions:</p>

<h1>
  
  
  Questions I Wanted to Answer
</h1>

<ul>
<li><p>Why does GPTBot visit <code>robots.txt</code> so many times, sometimes multiple times per day?</p></li>
<li><p>Why does <code>GPTBot</code> prefer <code>robots.txt</code> over <code>sitemap.xml</code>?</p></li>
<li><p>Why do I see AI bot traffic but no crawling of fresh content? Just repeated hits to old resources.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftxg3vvljjrfy1e1dtwtx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftxg3vvljjrfy1e1dtwtx.png" alt="Vercel Observability Query Builder: Bot traffic" width="800" height="718"></a></p>

<blockquote>
<p>(Screenshot 1: Vercel Observability Query Builder: Bot traffic)</p>
</blockquote>




<h2>
  
  
  1. <code>robots.txt</code> Obsession
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv9p8yesl7s4res9cyzs6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv9p8yesl7s4res9cyzs6.png" alt="OpenAI GPTBot hits robots multiple times per day" width="735" height="370"></a></p>

<blockquote>
<p>(Screenshot 2: OpenAI GPTBot robots.txt traffic)</p>
</blockquote>

<p>The charts clearly show GPTBot hammering robots.txt across multiple IPs, sometimes 7 times in 2 days from the same subnet. Unlike Googlebot, which fetches <code>robots.txt</code> a few times per day and caches the rules, <strong>GPTBot</strong> seems to re-check every time it rotates IPs or restarts.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foitkxfd1tvvselnveqb9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foitkxfd1tvvselnveqb9.png" alt="OpenAI AI crawlers traffic pattern" width="800" height="719"></a></p>

<blockquote>
<p>(Screenshot 3: OpenAI AI crawlers traffic pattern)</p>
</blockquote>

<p>That means there’s no centralised “consent” store for the crawler. Every new instance behaves like a fresh bot, wasting its crawl budget on permission checks.</p>




<h2>
  
  
  2. <code>sitemap.xml</code> Inconsistencies
</h2>

<p>I’ve tracked two different projects, and the behaviour is inconsistent. On one site, GPTBot fetched the sitemap exactly <strong>once in a month</strong>. On another, it skipped the sitemap entirely but went straight for content. Meanwhile, Anthropic’s ClaudeBot actually hit the sitemap multiple times.</p>

<p>The missing piece here is a smart algorithm that keeps score over time for each website. Google solved this years ago: it doesn’t blindly trust every <code>lastmod</code> tag, but instead builds a trust score for each domain based on history, accuracy, and freshness signals. That’s how it decides whether to treat a sitemap update seriously or to ignore it.</p>

<p>AI crawlers aren’t doing this yet. They either underuse sitemaps or waste fetches on them without consistency. To improve, AI labs need to adopt a similar scoring system. Or, as I strongly suspect from patterns I’ve seen, they may simply partner with Google Search and tap into its index instead of reinventing crawling from scratch.</p>

<blockquote>
<p>Side note: I’ve even seen OpenAI API results that looked suspiciously close to Google Search outputs ...</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F26ismcf6vt1dnc8yt72u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F26ismcf6vt1dnc8yt72u.png" alt="AI crawlers traffic pattern - Vercel Observability Query builder" width="800" height="600"></a></p>

<blockquote>
<p>(Screenshot 4: AI crawlers traffic pattern - Vercel Observability Query builder)</p>
</blockquote>




<h2>
  
  
  3. Crawling Old Content Repeatedly (and the Budget Problem)
</h2>

<p>This is where the inefficiency really shows. Bots keep returning to old content instead of discovering what’s new. Even when they’ve seen the sitemap, they often ignore it and waste their crawl budget revisiting stale pages.</p>

<p>There should be a smarter way to surface new material—and honestly, respecting <code>lastmod</code> in <code>sitemap.xml</code> would solve a lot of this. I really hope someone on the search teams at OpenAI and Anthropic is reading this.</p>

<p>From what I see:</p>

<ul>
<li><p>Crawling budgets are tiny. Sometimes a bot “spends” its limited fetches just on <code>robots.txt</code> and pages it has already crawled.</p></li>
<li><p>No centralised rule cache. Each IP acts independently, re-checking permissions and burning requests on duplicate work.</p></li>
<li><p>Unstable sessions. The pattern of repeated restarts suggests crawler instances spin up and down often, leading to wasted quota.</p></li>
</ul>

<p>And that’s why your fresh blog post doesn’t get fetched, while your <code>robots.txt</code> enjoys multiple visits per day.</p>




<h2>
  
  
  4. The Static Assets Surprise (a.k.a. Bots Running Headless Browsers)
</h2>

<p>Now here’s the real surprise: OpenAI’s crawler sometimes downloads static assets. Next.js chunks, CSS, polyfills. That almost certainly means it’s firing up a headless browser and <strong>actually rendering the page</strong>. Rendering at scale is expensive, so seeing this in the logs is like catching the bot red-handed burning VC money on your webpack bundles.</p>

<p><strong>Developers, let’s be honest: we shouldn’t force AI labs to reinvent Google Search’s rendering farm from scratch.</strong> The sane thing is still <em>to serve content via SSR/ISR so crawlers don’t have to play Chromium roulette just to see your page</em>. Otherwise you’re basically making Sam Altman pay more to crawl your vibe-coded portfolio site.</p>

<p>The funny bit? This is great news for vibe coders. All those sites built with pure CSR - the “AI slop” nobody thought would ever be indexable, might now actually get pulled into GPTBot’s memory. Your prayers have been heard... at least until the crawl budget runs out.</p>

<p><strong>Fun fact:</strong> some vibe coding tools default to CSR, which is basically SEO assisted suicide. If you care about visibility, whether in Google or in AI engines, please stop.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc98od3sc7cy4gnrxhlnr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc98od3sc7cy4gnrxhlnr.png" alt="Vercel Observability: AI crawlers render the page" width="800" height="733"></a></p>

<blockquote>
<p>(Screenshot 5: GPTBot download static assets (however hallucinated 404s)</p>
</blockquote>




<h2>
  
  
  5. What This Means for AI SEO
</h2>

<p><strong>The good news:</strong><br>
OpenAI and Anthropic at least play by the rules. They ask permission before scraping, unlike the swarm of shady scrapers hitting your site daily.</p>

<p><strong>The bad news:</strong></p>

<ul>
<li>Crawl budgets are tiny and often wasted.</li>
<li>Fresh content gets ignored.</li>
<li>Sitemaps and lastmod aren’t respected.</li>
<li>JS rendering happens only occasionally, so CSR-only sites are still at risk of being invisible.</li>
</ul>

<h2>
  
  
  Closing Thought
</h2>

<p>Google has a 25-year head start in crawling, indexing, and ranking. AI crawlers are still in year one of that journey. They’re not true search engines yet, but the scaffolding is going up fast.</p>

<blockquote>
<p>If anyone from OpenAI, Anthropic, or xAI is reading this: please, implement smarter crawl budgets and start respecting sitemap freshness. Otherwise, all we’ll get is bots lovingly revisiting robots.txt while the real content sits untouched.</p>
</blockquote>




<p>Godspeed</p>

<p><a href="https://x.com/dom_sipowicz" rel="noopener noreferrer">https://x.com/dom_sipowicz</a><br>
<a href="https://www.linkedin.com/in/dominiksipowicz/" rel="noopener noreferrer">https://www.linkedin.com/in/dominiksipowicz/</a></p>

