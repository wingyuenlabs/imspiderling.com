---
Title: How to Achieve 100/100 on PageSpeed Insights
Description: 
Author: Daniel Tofan
Date: 2025-11-22T21:55:52.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Reading time:</strong> 9 minutes</p>




<h2>
  
  
  Why write about PageSpeed optimization in 2025?
</h2>

<p>PageSpeed optimization isn't new. Google released Lighthouse in 2016, and performance best practices have been documented for years. You might expect that by 2025, most professional websites would score well.</p>

<p>The opposite is true.</p>

<p>Modern websites are often slower than sites built five years ago. Despite faster networks and more powerful devices, the average website in 2025 is heavier, more complex, and performs worse than its predecessors. Why?</p>

<p><strong>The complexity creep:</strong></p>

<ul>
<li>Modern frameworks ship more JavaScript by default</li>
<li>Analytics, marketing pixels, and chat widgets accumulate over time</li>
<li>High-resolution images and videos are now standard</li>
<li>Third-party integrations (payment processors, CRMs, booking systems) each add overhead</li>
<li>"It works on my machine" testing misses real-world mobile performance</li>
</ul>

<p>I regularly encounter professionally built websites from 2024-2025 scoring 40-70/100. These aren't old legacy sites—they're new builds using modern tools, costing thousands of dollars, from established agencies.</p>

<h2>
  
  
  Why does achieving 100/100 matter?
</h2>

<p>You might ask: "Isn't 85/100 good enough? Does the last 15 points really matter?"</p>

<p>The answer depends on what you're optimizing for.</p>

<p><strong>The business case:</strong></p>

<p>Google's research shows that 53% of mobile users abandon sites taking longer than 3 seconds to load. Each additional second of load time correlates with approximately 7% reduction in conversions. These aren't small numbers—they directly affect revenue.</p>

<p>For a business with 10,000 monthly visitors and a 3% conversion rate:</p>

<ul>
<li>A fast site (2 seconds, 100/100 score): 300 conversions/month</li>
<li>A slow site (8 seconds, 60/100 score): Approximately 210 conversions/month</li>
</ul>

<p>That's 90 lost conversions monthly. At $100 average transaction value, that's $108,000/year in lost revenue.</p>

<p><strong>The SEO case:</strong></p>

<p>Core Web Vitals became Google ranking factors in 2021. Two identical sites with identical content will rank differently based on performance. The faster site gets more organic traffic. More traffic means more conversions. The performance advantage compounds over time.</p>

<p><strong>The user experience case:</strong></p>

<p>Beyond metrics, there's a qualitative difference between a site that scores 85/100 and one that scores 100/100. The 100/100 site feels instant. Content appears immediately. Nothing jumps around. Users trust it more, engage more, and return more often.</p>

<p><strong>The competitive advantage:</strong></p>

<p>If your competitors score 60-70/100 and you score 100/100, you've created a measurable advantage in user experience, search rankings, and conversion rates. In competitive markets, these margins matter.</p>

<p>So yes, the last 15 points matter—not for the score itself, but for the business outcomes those points represent.</p>




<h2>
  
  
  What does a perfect PageSpeed score require?
</h2>

<p>Most developers know the basics—optimize images, minify CSS, reduce JavaScript. But knowing the basics and achieving 100/100 are different things. The gap between 85/100 and 100/100 isn't about doing more of the same. It requires understanding which techniques have the most impact and implementing them correctly.</p>

<p>I've built multiple sites scoring 100/100/100/100 across all four metrics (Performance, Accessibility, Best Practices, SEO). In this guide, I'll explain the specific techniques that matter most, why they work, and what to watch out for.</p>

<p><strong>You'll learn:</strong></p>

<ul>
<li>How critical CSS inlining affects render times</li>
<li>Why image format choice matters more than compression level</li>
<li>How to eliminate render-blocking resources</li>
<li>What causes layout shift and how to prevent it</li>
<li>How to test performance under real-world conditions</li>
</ul>

<p>Before we start, a reality check: getting to 100/100 takes time the first time through—typically 4-6 hours for a complete site. Once you understand the patterns, subsequent optimizations go faster. But there's no shortcut around learning what works.</p>




<h2>
  
  
  Why does critical CSS inlining improve scores?
</h2>

<p><strong>The problem:</strong> When browsers load a page, external CSS files block rendering. The browser downloads your HTML, encounters <code>&lt;link rel="stylesheet"&gt;</code>, pauses rendering, downloads the CSS, parses it, then finally renders the page. This delay affects your Largest Contentful Paint (LCP) score significantly.</p>

<p><strong>The solution:</strong> Inline your critical CSS directly in the <code>&lt;head&gt;</code> tag. The browser can render immediately without waiting for external files.</p>

<p><strong>A real example:</strong> A professional services site I optimized scored 94/100 before CSS inlining. After moving critical styles inline, it scored 100/100. The only change was moving approximately 3KB of above-the-fold CSS into the HTML head.</p>

<p>Here's what that structure looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;head&gt;</span>
  <span class="nt">&lt;style&gt;</span>
    <span class="c">/* Critical CSS - inline everything needed for initial render */</span>
    <span class="nt">body</span> <span class="p">{</span> <span class="nl">margin</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span> <span class="nl">font-family</span><span class="p">:</span> <span class="n">system-ui</span><span class="p">,</span> <span class="nb">sans-serif</span><span class="p">;</span> <span class="p">}</span>
    <span class="nt">header</span> <span class="p">{</span> <span class="nl">background</span><span class="p">:</span> <span class="m">#1a1a1a</span><span class="p">;</span> <span class="nl">color</span><span class="p">:</span> <span class="no">white</span><span class="p">;</span> <span class="nl">padding</span><span class="p">:</span> <span class="m">1rem</span><span class="p">;</span> <span class="p">}</span>
    <span class="nc">.hero</span> <span class="p">{</span> <span class="nl">min-height</span><span class="p">:</span> <span class="m">400px</span><span class="p">;</span> <span class="nl">background</span><span class="p">:</span> <span class="n">linear-gradient</span><span class="p">(...);</span> <span class="p">}</span>
  <span class="nt">&lt;/style&gt;</span>

  <span class="c">&lt;!-- Load non-critical CSS asynchronously --&gt;</span>
  <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"/full-styles.css"</span> <span class="na">media=</span><span class="s">"print"</span> <span class="na">onload=</span><span class="s">"this.media='all'"</span><span class="nt">&gt;</span>
<span class="nt">&lt;/head&gt;</span>
</code></pre>

</div>



<p><strong>What to watch for:</strong> Keep inline CSS under 8KB. Beyond this size, you're delaying HTML download time, which can actually hurt your First Contentful Paint score instead of helping it. Extract only the styles needed for above-the-fold content.</p>

<p><strong>Framework considerations:</strong> If you're using Nuxt, Next.js, or similar frameworks, look for build-time CSS extraction features. Nuxt's <code>experimental.inlineSSRStyles</code> handles this automatically during static generation.</p>




<h2>
  
  
  How do modern image formats reduce file size?
</h2>

<p><strong>The problem:</strong> Images typically account for 60-80% of page weight. Unoptimized images directly affect load times, especially on mobile networks.</p>

<p><strong>The solution:</strong> Use AVIF format where supported, with WebP fallback, and serve appropriately sized images for different viewports.</p>

<p><strong>A real example:</strong> I built a healthcare website (<a href="https://medconnect.codecrank.ai" rel="noopener noreferrer">medconnect.codecrank.ai</a>) with professional medical imagery and team photos. Initial image exports totaled approximately 2.5MB per page. After optimization:</p>

<ul>
<li>
<strong>Format conversion:</strong> Changed from PNG/JPG to AVIF (50-80% smaller at equivalent quality)</li>
<li>
<strong>Responsive sizing:</strong> Served 400px images for mobile, 1200px for desktop (not full 4K resolution for all devices)</li>
<li>
<strong>Result:</strong> Lightweight page weight with excellent visual quality</li>
<li>
<strong>Performance score:</strong> 100/100/100/100 (perfect scores across all metrics)</li>
</ul>

<p><strong>The implementation:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;picture&gt;</span>
  <span class="nt">&lt;source</span> <span class="na">srcset=</span><span class="s">"hero-400.avif 400w, hero-800.avif 800w, hero-1200.avif 1200w"</span>
          <span class="na">type=</span><span class="s">"image/avif"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;source</span> <span class="na">srcset=</span><span class="s">"hero-400.webp 400w, hero-800.webp 800w, hero-1200.webp 1200w"</span>
          <span class="na">type=</span><span class="s">"image/webp"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;img</span> <span class="na">src=</span><span class="s">"hero-800.jpg"</span>
       <span class="na">alt=</span><span class="s">"Hero image"</span>
       <span class="na">width=</span><span class="s">"1200"</span>
       <span class="na">height=</span><span class="s">"800"</span>
       <span class="na">loading=</span><span class="s">"lazy"</span><span class="nt">&gt;</span>
<span class="nt">&lt;/picture&gt;</span>
</code></pre>

</div>



<p><strong>How browsers handle this:</strong> Modern browsers automatically select the best format and size they support. Chrome uses AVIF, Safari uses WebP (AVIF support pending), and older browsers fall back to JPG. Mobile devices get 400px versions, desktop gets 1200px. You write it once, browsers handle the rest.</p>

<p><strong>Tools worth knowing:</strong> Squoosh (squoosh.app) for manual conversion with quality preview, or Sharp (Node.js library) for batch processing. Both give you control over quality settings per image.</p>




<h2>
  
  
  What makes resources render-blocking?
</h2>

<p><strong>The problem:</strong> External JavaScript files block the browser's main thread during download and execution. Even optimized scripts add 200-500ms of blocking time, which directly affects your Time to Interactive and Total Blocking Time scores.</p>

<p><strong>The solution:</strong> Defer JavaScript execution until after initial page render. Load scripts after the page displays content to users.</p>

<p><strong>The Google Analytics consideration:</strong> Standard GA4 implementation is the most common performance issue I encounter. The default tracking code blocks rendering and adds approximately 500ms to LCP.</p>

<p><strong>Standard implementation (blocks rendering):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;script </span><span class="na">async</span> <span class="na">src=</span><span class="s">"https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"</span><span class="nt">&gt;&lt;/script&gt;</span>
</code></pre>

</div>



<p><strong>Performance-optimized implementation:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;script&gt;</span>
  <span class="nb">window</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">load</span><span class="dl">'</span><span class="p">,</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
    <span class="c1">// Load GA4 after page fully renders</span>
    <span class="kd">var</span> <span class="nx">script</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="dl">'</span><span class="s1">script</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">script</span><span class="p">.</span><span class="nx">src</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX</span><span class="dl">'</span><span class="p">;</span>
    <span class="nb">document</span><span class="p">.</span><span class="nx">head</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nx">script</span><span class="p">);</span>
  <span class="p">});</span>
<span class="nt">&lt;/script&gt;</span>
</code></pre>

</div>



<p><strong>The trade-off:</strong> This approach won't track users who leave within the first 2 seconds of page load. In practice, this represents less than 1% of traffic for most sites and is worth the performance improvement.</p>

<p><strong>What to check:</strong> Review your <code>&lt;head&gt;</code> section. Any <code>&lt;script&gt;</code> tag without <code>defer</code> or <code>async</code> attributes is blocking. Move it to the page bottom or defer its execution.</p>




<h2>
  
  
  How do you prevent cumulative layout shift?
</h2>

<p><strong>The problem:</strong> Content shifting position while the page loads creates a poor user experience and hurts your CLS (Cumulative Layout Shift) score. Common causes include images loading without reserved space, web fonts swapping in, or ads inserting dynamically.</p>

<p><strong>The solution:</strong> Reserve space for all content before it loads.</p>

<p><strong>A real example:</strong> A professional services site I built maintains a CLS score of 0 (zero layout shift). Here's the approach:</p>

<p><strong>1. Set explicit image dimensions:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;img</span> <span class="na">src=</span><span class="s">"hero.jpg"</span> <span class="na">width=</span><span class="s">"1200"</span> <span class="na">height=</span><span class="s">"800"</span> <span class="na">alt=</span><span class="s">"Hero"</span><span class="nt">&gt;</span>
<span class="c">&lt;!-- Browser reserves 1200x800 space before image loads --&gt;</span>
</code></pre>

</div>



<p><strong>2. Use CSS aspect-ratio for responsive images:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="nt">img</span> <span class="p">{</span>
  <span class="nl">width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
  <span class="nl">height</span><span class="p">:</span> <span class="nb">auto</span><span class="p">;</span>
  <span class="py">aspect-ratio</span><span class="p">:</span> <span class="m">3</span><span class="p">/</span><span class="m">2</span><span class="p">;</span> <span class="c">/* Maintains space even as viewport changes */</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>3. Configure fonts to display fallbacks immediately:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="k">@font-face</span> <span class="p">{</span>
  <span class="nl">font-family</span><span class="p">:</span> <span class="s2">'CustomFont'</span><span class="p">;</span>
  <span class="nl">src</span><span class="p">:</span> <span class="sx">url('/fonts/custom.woff2')</span><span class="p">;</span>
  <span class="py">font-display</span><span class="p">:</span> <span class="n">swap</span><span class="p">;</span> <span class="c">/* Show system font immediately, swap when custom font loads */</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>The result:</strong> Content positions remain stable throughout the load process. The page feels responsive and professional.</p>

<p><strong>Debugging layout shift:</strong> Run PageSpeed Insights and review the filmstrip view. If you see elements jumping position, add explicit dimensions or aspect-ratios to the shifting elements.</p>




<h2>
  
  
  What common issues occur during optimization?
</h2>

<p>Even when following established practices, you might encounter these issues:</p>

<h3>
  
  
  "I inlined my CSS but my score dropped"
</h3>

<p>This happens when inline CSS exceeds 8-10KB. The browser must download the entire HTML file before rendering anything, which delays First Contentful Paint.</p>

<p><strong>Solution:</strong> Extract only above-the-fold styles. Identify which CSS is needed for initial viewport rendering, inline only that portion, and load the rest asynchronously:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"/full-styles.css"</span> <span class="na">media=</span><span class="s">"print"</span> <span class="na">onload=</span><span class="s">"this.media='all'"</span><span class="nt">&gt;</span>
</code></pre>

</div>



<h3>
  
  
  "My AVIF images appear blurry or pixelated"
</h3>

<p>Default AVIF quality settings are often too aggressive. A quality setting of 50 works for photographs but degrades images containing text or graphics.</p>

<p><strong>Solution:</strong> Increase quality to 75-85 for images with text or fine details. Use image conversion tools that show quality previews before batch processing.</p>

<h3>
  
  
  "CLS score remains poor despite setting dimensions"
</h3>

<p>Common culprits beyond images: web fonts loading (text reflows when custom font loads), ads inserting dynamically, or content above images pushing them down during load.</p>

<p><strong>Solutions:</strong></p>

<ul>
<li>Fonts: Use <code>font-display: swap</code> and preload critical font files</li>
<li>Images: Always set explicit <code>width</code> and <code>height</code> attributes or use CSS <code>aspect-ratio</code>
</li>
<li>Dynamic content: Set minimum heights on containers before content populates</li>
</ul>

<h3>
  
  
  "Performance is great on desktop but needs work on mobile"
</h3>

<p>Mobile devices have slower processors and network connections. What renders quickly on a development machine often struggles on mid-range Android phones over 4G networks. If you're only testing on desktop, you're missing how most users experience your site.</p>

<p><strong>Solution:</strong> Always test mobile performance with Chrome DevTools throttling enabled (4x CPU slowdown, Fast 3G network). This simulates realistic mobile conditions and reveals actual user experience. Aim for 90+ on mobile, and you will likely get 100 on desktop.</p>




<h2>
  
  
  How should you test performance?
</h2>

<p><strong>The challenge:</strong> Your site might perform well on your development machine but struggle in real-world conditions. Mobile users on 4G with mid-range phones experience performance very differently than you do on a MacBook Pro with fiber internet.</p>

<p><strong>The testing process:</strong></p>

<ol>
<li>
<strong>Run PageSpeed Insights</strong> (pagespeed.web.dev) on mobile configuration first</li>
<li>
<strong>Check Core Web Vitals against targets:</strong>

<ul>
<li>LCP (Largest Contentful Paint): under 2.5 seconds</li>
<li>TBT (Total Blocking Time): under 200ms</li>
<li>CLS (Cumulative Layout Shift): under 0.1</li>
</ul>
</li>
<li>
<strong>Review the filmstrip:</strong> Look for empty frames where nothing renders</li>
<li>
<strong>Address the largest issue first</strong> (typically render-blocking CSS or oversized images)</li>
<li>
<strong>Re-test and iterate</strong> until reaching 90+/100</li>
</ol>

<p><strong>A perspective on perfection:</strong> Don't necessarily chase 100/100 if you're already at 95+. The final 5 points often represent diminishing returns. Consider whether that time is better spent on content, user experience, or other priorities.</p>

<p><strong>Real device testing:</strong> Test on actual mobile devices when possible, not just Chrome DevTools simulation. Real hardware reveals issues that simulators miss.</p>




<h2>
  
  
  What do these techniques look like in practice?
</h2>

<p><strong>Examples you can test right now:</strong></p>

<p>I built these sites to demonstrate these techniques in production:</p>

<p><strong>Professional Services Example</strong> - <a href="https://zenaith.codecrank.ai" rel="noopener noreferrer">zenaith.codecrank.ai</a></p>

<ul>
<li>
<strong>Scores:</strong> 100/100/100/100 (all four metrics)</li>
<li>
<strong>LCP:</strong> 1.8 seconds</li>
<li>
<strong>Techniques used:</strong> Inline critical CSS, AVIF images, zero render-blocking scripts, CLS: 0</li>
<li>
<strong>Verification:</strong> Click "⚡ PageSpeed Me" in the footer to test live</li>
</ul>

<p><strong>MedConnect Pro (Healthcare Site)</strong> - <a href="https://medconnect.codecrank.ai" rel="noopener noreferrer">medconnect.codecrank.ai</a></p>

<ul>
<li>
<strong>Scores:</strong> 100/100/100/100 (perfect scores across all metrics)</li>
<li>
<strong>Techniques used:</strong> AVIF conversion, responsive images, semantic HTML, accessibility-first design</li>
<li>
<strong>Verification:</strong> Click "⚡ PageSpeed Me" in the footer to test live</li>
</ul>

<p><strong>Mixology (Cocktail Recipes)</strong> - <a href="https://mixology.codecrank.ai" rel="noopener noreferrer">mixology.codecrank.ai</a></p>

<ul>
<li>
<strong>Score:</strong> 96/100 mobile, 100/100 desktop</li>
<li>
<strong>Techniques used:</strong> Static generation, optimized images, minimal JavaScript</li>
</ul>

<p>You can test any of these sites yourself. Newer sites even include a 'PageSpeed Me' button linking directly to Lighthouse testing. I have nothing to hide—the scores are verifiable.</p>




<h2>
  
  
  The Honest Take
</h2>

<p>Most developers ship sites that "work" and move on. Getting to 100/100 takes additional time and attention to detail that many choose not to invest.</p>

<p>I put PageSpeed buttons on every site I build because the work should speak for itself. If I claim 100/100, you can verify it immediately. If I don't achieve it, I'll explain exactly why (client-requested features, necessary third-party integrations, etc.).</p>

<p>Fair warning: PageSpeed scores can fluctuate by a few points depending on network conditions, server load, and time of day. A site scoring 100/100 might test at 98/100 an hour later. What matters is consistent high performance (90-100 range), not chasing a perfect score every single test.</p>

<p>This transparency is uncommon in web development. Many agencies don't want you testing their work. I built these techniques into my process specifically so performance isn't an afterthought—it's built in from the start.</p>

<p>The results are measurable, verifiable, and reproducible. Some clients care deeply about performance. Some don't. I serve those who do.</p>




<h2>
  
  
  What's the realistic time investment?
</h2>

<p>Optimizing to 100/100 isn't quick the first time through. For a typical site, expect:</p>

<ul>
<li>1-2 hours: Image optimization (format conversion, resizing, quality testing)</li>
<li>1-2 hours: CSS optimization (critical extraction, inline implementation)</li>
<li>1 hour: Layout shift fixes (dimensions, aspect-ratios, font configuration)</li>
<li>1 hour: Testing and iteration</li>
</ul>

<p><strong>Total:</strong> 4-6 hours for first-time implementation</p>

<p><strong>However:</strong> Once you've completed this process for one site, you understand the patterns. Your second site takes approximately 2 hours. Your tenth site takes 30 minutes because you've built the tooling and established the workflow.</p>

<p>Most developers never invest this time because "good enough" ships. But if you care about user experience, SEO performance, and conversion rates, it's worth learning these techniques.</p>




<h2>
  
  
  What comes next?
</h2>

<p>You now understand how to achieve 100/100 PageSpeed scores. You know the techniques, the trade-offs, and the testing approach.</p>

<p>In my next article, I'll examine why performance optimization often gets overlooked in professional web development. I'll share a real case study—a $5,000 professional website scoring 40/100—and explain what affects the cost and quality of web development.</p>

<p><strong>Want to verify these techniques work?</strong> Visit any of the sites mentioned above and click "⚡ PageSpeed Me" to test them live. Then consider: what would perfect performance scores mean for your business?</p>

<p><strong>Need help with performance optimization?</strong> Visit <a href="https://codecrank.ai" rel="noopener noreferrer">codecrank.ai</a> to learn about our approach to web development. We build performance optimization into every project from day one.</p>




<p><em>All performance metrics verified with Google Lighthouse. Sites tested on mobile with 4G throttling and mid-tier device simulation.</em></p>

