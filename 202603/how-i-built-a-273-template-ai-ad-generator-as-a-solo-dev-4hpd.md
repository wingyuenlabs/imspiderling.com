---
Title: How I Built a 273 Template AI Ad Generator as a Solo Dev
Description: 
Author: Aria Cole
Date: 2026-03-27T22:06:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>Two weeks ago I had an idea: what if you could drop any brand's URL and get professional ad creatives generated instantly?</p>

<p>Today, <a href="https://siloai.app" rel="noopener noreferrer">Silo</a> has 92 users, 273 ad templates, and people are actually using it to generate real ads for their brands.</p>

<p>Here's how I built it.</p>

<h2>
  
  
  The Stack
</h2>

<p>Nothing fancy:</p>

<ul>
<li>
<strong>Next.js 14</strong> (App Router)</li>
<li>
<strong>Prisma + SQLite</strong> on Fly.io (single machine, volume mounted)</li>
<li>
<strong>Gemini AI</strong> for both Brand DNA extraction and image generation</li>
<li>
<strong>Stripe</strong> for billing</li>
<li>
<strong>Google OAuth</strong> for sign in</li>
</ul>

<p>Total infrastructure cost: under $10/month. SQLite on a single Fly.io machine is honestly all you need until you hit thousands of users.</p>

<h2>
  
  
  How Brand DNA Works
</h2>

<p>When a user drops a URL, here's what happens:</p>

<ol>
<li>We scrape the page using Cheerio (server side, no headless browser needed)</li>
<li>Extract all visible text, meta tags, Open Graph data, and image URLs</li>
<li>Send everything to Gemini with a detailed prompt asking it to analyze: brand colors, typography style, voice/tone, target audience, key messaging</li>
<li>The AI returns structured JSON we call "Brand DNA"</li>
</ol>

<p>This Brand DNA becomes the foundation for every ad we generate. It's what makes the output actually look like the brand instead of generic AI slop.</p>

<h2>
  
  
  The Template System
</h2>

<p>This is where it gets interesting. Each of our 273 templates is essentially a mega detailed AI prompt with <code>[BRACKET]</code> placeholders.</p>

<p>For example, our "Steam Mirror Writing" template describes exactly: the background (steamy bathroom mirror), the text style (finger drawn on condensation), the product placement, the lighting, everything. When we generate, the AI fills in the brackets with brand specific info from the Brand DNA.</p>

<p>The key insight: <strong>the more detailed the prompt, the better the output</strong>. A vague prompt like "create an ad for this product" gives you garbage. A 200 word prompt describing exact layout, typography, colors, and composition gives you something that actually looks professional.</p>

<h2>
  
  
  The Pivot That Changed Everything
</h2>

<p>I originally built a Fabric.js canvas editor so users could manually adjust generated ads. Layers, text tools, drag and drop, the whole thing.</p>

<p>After building it 5 different ways, I scrapped everything and pivoted to an AI chat editor. Users just describe what they want changed ("make the text bigger", "change the background to blue") and the AI regenerates the image.</p>

<p>Way simpler. Way better results. Sometimes the best code is code you delete.</p>

<h2>
  
  
  What's Actually Working for Growth
</h2>

<p>Forget paid ads for now. Here's what's driving real signups:</p>

<ol>
<li>
<strong>Twitter/X</strong> (32 visitors in one day from organic posts)</li>
<li>
<strong>TAAFT directory listing</strong> (9 referrals, totally free)</li>
<li>
<strong>Organic word of mouth</strong>: A Polish AI educator found Silo, wrote a full tutorial about it, and drove 42 signups in a single day</li>
</ol>

<p>That last one blew my mind. I didn't reach out to him. He found the tool, liked it, and shared it with his audience. You can't buy that kind of marketing.</p>

<h2>
  
  
  The Numbers (Keeping It Real)
</h2>

<ul>
<li>92 total users</li>
<li>1 paying customer at $59/month</li>
<li>192 total ad generations</li>
<li>273 templates across 15+ categories</li>
<li>~$0.067 per generated image (Gemini AI)</li>
<li>Under $10/month infrastructure</li>
</ul>

<p>Am I getting rich? No. But the product works, people are using it, and organic growth is happening. That's enough signal to keep building.</p>

<h2>
  
  
  What's Next
</h2>

<ul>
<li>More SEO content (comparison pages are converting)</li>
<li>Template library keeps growing</li>
<li>Trying to crack the free to paid conversion</li>
<li>More direct outreach to DTC brands</li>
</ul>

<p>If you're building something similar or have questions about the stack, drop a comment. Happy to share more details.</p>

<p><strong>Try it free:</strong> <a href="https://siloai.app" rel="noopener noreferrer">siloai.app</a></p>

