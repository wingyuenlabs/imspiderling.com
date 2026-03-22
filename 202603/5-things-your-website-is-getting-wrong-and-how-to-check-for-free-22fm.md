---
Title: 5 things your website is getting wrong (and how to check for free)
Description: 
Author: Ben Greenberg
Date: 2026-03-22T21:51:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most websites fail basic technical hygiene checks. Not because developers don't care, but because these things are easy to miss when you're focused on shipping features. Here are five common issues worth fixing today.</p>

<h2>
  
  
  1. Missing or Wrong Security Headers
</h2>

<p>Headers like <code>Content-Security-Policy</code>, <code>X-Frame-Options</code>, and <code>Strict-Transport-Security</code> (HSTS) protect your users from clickjacking, XSS attacks, and protocol downgrade attacks. Skipping them leaves real attack surface open. Browsers and security scanners will flag these absences, and some enterprise clients actively check before integrating with your API.</p>

<p><strong>How to check:</strong> Run <code>curl -I https://yourdomain.com</code> and scan the response headers. Or paste your URL into <a href="https://securityheaders.com" rel="noopener noreferrer">securityheaders.com</a> for a free graded report.</p>

<h2>
  
  
  2. Open Graph Tags That Break Link Previews
</h2>

<p>When someone shares your link on Slack, LinkedIn, or Twitter, the platform reads your Open Graph meta tags to build the preview card. If <code>og:title</code>, <code>og:image</code>, or <code>og:description</code> are missing or misconfigured, the preview looks broken or empty. This tanks click-through rates on content you spent real time creating.</p>

<p><strong>How to check:</strong> Use the <a href="https://www.opengraph.xyz" rel="noopener noreferrer">OpenGraph.xyz</a> previewer. Paste your URL and see exactly what Slack or LinkedIn will render. Fix any missing tags in your <code>&lt;head&gt;</code>.</p>

<h2>
  
  
  3. No Canonical URL
</h2>

<p>If your page is reachable at both <code>https://example.com/page</code> and <code>https://example.com/page?ref=newsletter</code>, search engines may treat these as separate pages competing against each other. Over time this splits your ranking signals and can suppress both versions. A canonical tag tells crawlers which URL is the one that counts.</p>

<p><strong>How to check:</strong> Open DevTools in Chrome, go to Elements, and search for <code>canonical</code> in the <code>&lt;head&gt;</code>. You should find something like <code>&lt;link rel="canonical" href="https://example.com/page" /&gt;</code>. If it is missing or pointing to the wrong URL, fix it in your template.</p>

<h2>
  
  
  4. Images Without Alt Text
</h2>

<p>Alt text is not optional. Screen readers rely on it for users with visual impairments, and search engine crawlers use it to understand image content. A page full of images with empty or missing <code>alt</code> attributes is both an accessibility failure and a missed SEO opportunity.</p>

<p><strong>How to check:</strong> Run <code>document.querySelectorAll('img:not([alt])')</code> in the browser console. Any results mean you have untagged images. Axe DevTools (free browser extension) will also flag these automatically.</p>

<h2>
  
  
  5. Missing Viewport Meta Tag
</h2>

<p>Without <code>&lt;meta name="viewport" content="width=device-width, initial-scale=1"&gt;</code> in your <code>&lt;head&gt;</code>, mobile browsers will render your page at desktop width and then scale it down. The result is a tiny, unreadable layout that frustrates users and tanks your Core Web Vitals scores.</p>

<p><strong>How to check:</strong> Open DevTools, toggle the device toolbar (Ctrl+Shift+M), and load your page. If it renders like a shrunken desktop site, the viewport tag is probably missing. Check the Elements panel to confirm.</p>




<p>These five checks take maybe 15 minutes to run manually. If you want to do all of them in one go, I built a free tool that runs 5 key checks instantly, no signup, no waiting: <a href="https://audit.hummusonrails.com/free" rel="noopener noreferrer">https://audit.hummusonrails.com/free</a></p>

