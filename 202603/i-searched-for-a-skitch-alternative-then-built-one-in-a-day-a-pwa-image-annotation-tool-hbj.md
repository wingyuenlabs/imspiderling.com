---
Title: I Searched for a Skitch Alternative, Then Built One in a Day — A PWA Image Annotation Tool
Description: 
Author: tommy
Date: 2026-03-06T22:01:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>I just wanted to draw one arrow on a screenshot and paste it into a chat.</p>

<p>That's it. Why does it take 30 seconds?</p>

<p>Since Skitch was effectively discontinued, every alternative I tried hit me with "Please sign in," "Choose a plan," or "Download required." No. That's not what I need.</p>




<p><strong>So I stopped searching and built one.</strong></p>

<blockquote>
<p><strong><a href="https://annotate.puremark.app" rel="noopener noreferrer">Pure Mark Annotate — Try it now (completely free)</a></strong></p>

<p>No install. No login. No account. Open the browser, drop an image, draw an arrow. Done.</p>
</blockquote>




<h2>
  
  
  What It Does (30-Second Overview)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Action</th>
<th>How</th>
</tr>
</thead>
<tbody>
<tr>
<td>Load an image</td>
<td>Drag &amp; drop / Ctrl+V paste / file picker</td>
</tr>
<tr>
<td>Annotate</td>
<td>Arrows, text, rectangles, highlights, mosaic, numbered circles</td>
</tr>
<tr>
<td>Blur faces</td>
<td>Auto-detect and one-click mosaic</td>
</tr>
<tr>
<td>Save</td>
<td>iPhone → Share Sheet / Desktop → PNG download</td>
</tr>
</tbody>
</table></div>

<p>Works on both desktop and mobile. English and Japanese UI.</p>

<p><strong>I obsessed over getting that "Skitch-style arrow" right — if you remember that feel, you'll know it when you see it.</strong></p>




<h2>
  
  
  Two Weeks of Going Nowhere
</h2>

<p>I decided to build this in early 2026.</p>

<p>The first plan was Flutter. Ship to both iOS and Android. Register on the stores, reach ex-Skitch users. I did market research, chose the brand name "Pure Mark Annotate," set up a Docker dev environment... and stalled.</p>

<p>"How do I test on a real device?" "I only have an iPhone, but should I build the Android version first?" "Google Play requires a public address, and the review process needs 12 testers..."</p>

<p>For about two weeks, I explored every direction. Flutter Web, cloud Macs, buying a cheap Android phone. Before I knew it, I'd spent all my time thinking about <em>which platform to build on</em> without writing a single line of the actual "draw an arrow" implementation.</p>

<p>The real problem wasn't the tool. <strong>It was friction.</strong></p>

<p>What I loved about Skitch was the zero friction to start using it. Open it when you want to, draw when you want to — that transparency. So the answer was obvious from the start. PWA.</p>

<p>One night, I dropped everything and decided: Flutter — done. App store submissions — figure it out later. Build something that works. Today.</p>




<h2>
  
  
  The Arrow Took 3 Hours
</h2>

<p>The core implementation came together in a few hours with Claude.</p>

<p>But I spent 3 hours on the arrow.</p>

<p>A simple triangle and a line should be fine — that's what I thought for the first 30 minutes, until something felt off. Skitch's arrow had a distinctive silhouette. The sharpness of the tip, the "pinch" of the shaft, the drop shadow that boosted visibility. It wasn't just a shape — it was a line with intention.</p>

<p>"Make the base a bit thinner." "The drop shadow should be subtle, but wide."</p>

<p>I iterated on the Canvas API paths in conversation with Claude. Bezier curves for the pinch, dialing in the shadow opacity and blur by the numbers, calculating the tip geometry so it wouldn't break at different angles. In a solo project, you can afford to spend 3 hours like this.</p>




<h2>
  
  
  The Day After Deploy, iPhone Returned a Blank Page
</h2>

<p>"Perfect." I deployed. The next day, I tested on iPhone.</p>

<p>The exported image was pure white. The annotations were gone.</p>

<p><strong>Root cause: iOS Safari's canvas pixel limit (~16.7 million pixels)</strong></p>

<p>A high-resolution photo (typical iPhone 4000x3000 = 12M px) exported at 3x scale easily exceeds the limit. No error. Just... white.</p>

<p>The fix: calculate the device's limit before export and auto-scale down.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">getSafeExportScale</span><span class="p">(</span><span class="nx">w</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">h</span><span class="p">:</span> <span class="kr">number</span><span class="p">):</span> <span class="kr">number</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">isIOS</span> <span class="o">=</span> <span class="sr">/iPad|iPhone|iPod/</span><span class="p">.</span><span class="nf">test</span><span class="p">(</span><span class="nb">navigator</span><span class="p">.</span><span class="nx">userAgent</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">maxPixels</span> <span class="o">=</span> <span class="nx">isIOS</span> <span class="p">?</span> <span class="mi">16</span><span class="nx">_000_000</span> <span class="p">:</span> <span class="mi">100</span><span class="nx">_000_000</span><span class="p">;</span>
  <span class="kd">let</span> <span class="nx">scale</span> <span class="o">=</span> <span class="mi">3</span><span class="p">;</span>
  <span class="k">while </span><span class="p">(</span><span class="nx">w</span> <span class="o">*</span> <span class="nx">scale</span> <span class="o">*</span> <span class="nx">h</span> <span class="o">*</span> <span class="nx">scale</span> <span class="o">&gt;</span> <span class="nx">maxPixels</span> <span class="o">&amp;&amp;</span> <span class="nx">scale</span> <span class="o">&gt;</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">scale</span> <span class="o">-=</span> <span class="mf">0.5</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="nx">scale</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>On top of that, <code>canvas.toBlob()</code> is async and loses user activation, so <code>navigator.share()</code> won't work on iOS. Fixed it by using the synchronous <code>toDataURL()</code> and manually converting to a Blob.</p>

<p>After implementing this, I tested again. The annotated image exported correctly.</p>

<p>Skitch's arrow was back on iPhone.</p>




<blockquote>
<p><strong><a href="https://annotate.puremark.app" rel="noopener noreferrer">Pure Mark Annotate — Works on iPhone too</a></strong></p>
</blockquote>




<h2>
  
  
  Tech Stack (For Developers)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Tech</th>
<th>Why</th>
</tr>
</thead>
<tbody>
<tr>
<td>React 19 + TypeScript</td>
<td>Type safety, ecosystem breadth</td>
</tr>
<tr>
<td>Vite 6</td>
<td>Blazing fast HMR, great PWA plugin support</td>
</tr>
<tr>
<td>Tailwind CSS v4</td>
<td>Utility-first, perfect for dark UI</td>
</tr>
<tr>
<td>Zustand</td>
<td>Lightweight state management (no Redux needed)</td>
</tr>
<tr>
<td>vite-plugin-pwa</td>
<td>Auto SW generation, offline support</td>
</tr>
<tr>
<td>YuNet + ONNX Runtime Web</td>
<td>Face detection → auto mosaic (228KB, MIT license)</td>
</tr>
<tr>
<td>Cloudflare Pages</td>
<td>GitHub integration, auto deploy, free SSL</td>
</tr>
</tbody>
</table></div>

<p>Why React PWA over Flutter: Flutter's bundle is 2-5MB, <code>dart:io</code> doesn't work on web, and SEO is impossible. A React PWA loads in a few hundred KB and runs directly in Safari on iPhone for testing.</p>




<h2>
  
  
  Other Gotchas (Developer Notes)
</h2>

<h3>
  
  
  Mosaic Misalignment on Retina Displays
</h3>

<p>Retina displays have DPR 2-3x, so the canvas physical pixels and CSS coordinates diverge. Fixed by normalizing the mosaic source image to CSS pixel space.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Don't multiply by DPR — stay in CSS pixel space</span>
<span class="kd">const</span> <span class="nx">imgCanvas</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="dl">'</span><span class="s1">canvas</span><span class="dl">'</span><span class="p">);</span>
<span class="nx">imgCanvas</span><span class="p">.</span><span class="nx">width</span> <span class="o">=</span> <span class="nx">rect</span><span class="p">.</span><span class="nx">width</span><span class="p">;</span>
<span class="nx">imgCanvas</span><span class="p">.</span><span class="nx">height</span> <span class="o">=</span> <span class="nx">rect</span><span class="p">.</span><span class="nx">height</span><span class="p">;</span>
<span class="nx">imgCtx</span><span class="p">.</span><span class="nf">drawImage</span><span class="p">(</span><span class="nx">image</span><span class="p">,</span> <span class="nx">dx</span><span class="p">,</span> <span class="nx">dy</span><span class="p">,</span> <span class="nx">dw</span><span class="p">,</span> <span class="nx">dh</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Text Becomes Tiny on Export
</h3>

<p>Coordinates and <code>strokeWidth</code> were being scaled, but <code>fontSize: 24</code> and the numbered circle radius <code>18</code> were hardcoded. Fixed by passing a <code>scale</code> argument through all drawing parameters.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">function</span> <span class="nf">renderAnnotations</span><span class="p">(</span><span class="nx">ctx</span><span class="p">,</span> <span class="nx">annotations</span><span class="p">,</span> <span class="nx">imageCanvas</span><span class="p">,</span> <span class="nx">scale</span> <span class="o">=</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">fontSize</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="mi">24</span> <span class="o">*</span> <span class="nx">scale</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">r</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="mi">18</span> <span class="o">*</span> <span class="nx">scale</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">pixelSize</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="mi">10</span> <span class="o">*</span> <span class="nx">scale</span><span class="p">);</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  I Changed the Face Detection Model 4 Times
</h3>

<p>The mosaic feature's auto face detection needed on-device inference that runs in the browser. This was the hardest part. Long story short, I swapped models four times.</p>

<p><strong>1. MediaPipe BlazeFace (first choice)</strong><br>
Google-made, lightweight, decent accuracy. But WebAssembly compatibility issues on iOS Safari meant it didn't work on some devices. A PWA that doesn't work everywhere isn't a PWA. Dropped early.</p>

<p><strong>2. @vladmandic/face-api (SSD MobileNet V1)</strong><br>
TensorFlow.js-based, better iOS stability. But the accuracy wasn't production-ready. Missed faces in group photos, weak on side profiles, too many false positives. Six commits of threshold tuning and multi-scale scanning couldn't save it.</p>

<p><strong>3. SCRFD-2.5GF (ONNX Runtime Web)</strong><br>
InsightFace's high-accuracy model. WIDERFace Easy 93.8% — leagues ahead. Group photos and profile shots handled nearly perfectly. Dropped the TensorFlow.js dependency too. But then I discovered the <strong>license is non-commercial</strong> (commercial use requires separate permission). With future monetization in mind, I had to swap it out.</p>

<p><strong>4. YuNet 2023mar (current)</strong><br>
From OpenCV Zoo, MIT licensed, no commercial restrictions. Model size 228KB — <strong>93% smaller</strong> than SCRFD's 3.2MB. WIDERFace accuracy drops about 5%, but for PureMark's use case (mosaic on front-facing to slightly angled faces), it's more than enough.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th></th>
<th>BlazeFace</th>
<th>face-api.js</th>
<th>SCRFD-2.5GF</th>
<th>YuNet</th>
</tr>
</thead>
<tbody>
<tr>
<td>Model size</td>
<td>~400KB</td>
<td>~5.4MB</td>
<td>3.2MB</td>
<td><strong>228KB</strong></td>
</tr>
<tr>
<td>iOS stability</td>
<td>No</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Group photo accuracy</td>
<td>Fair</td>
<td>Poor</td>
<td>Excellent</td>
<td>Good</td>
</tr>
<tr>
<td>Commercial license</td>
<td>Yes</td>
<td>Yes</td>
<td>No (needs permission)</td>
<td>Yes (MIT)</td>
</tr>
<tr>
<td>Runtime</td>
<td>MediaPipe</td>
<td>TF.js</td>
<td>ONNX Runtime</td>
<td>ONNX Runtime</td>
</tr>
</tbody>
</table></div>

<p><strong>Lesson: If you pick a model based on accuracy alone, licensing will bite you later.</strong> Open source doesn't mean commercially free. Academic models are especially risky.</p>

<p>The PWA Service Worker uses a <code>CacheFirst</code> strategy, so face detection works offline from the second visit onward.</p>


<h2>
  
  
  Deploying to Cloudflare Pages
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># public/_redirects (SPA routing)
/* /index.html 200
</code></pre>

</div>


<p>Subdomain design anticipates future multi-app expansion:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>puremark.app               → Landing page (future)
annotate.puremark.app      → Image annotation (this project)
</code></pre>

</div>



<p>Domain, DNS, and hosting are all on Cloudflare — CNAME setup and SSL issuance are fully automatic.</p>




<h2>
  
  
  Summary
</h2>

<ul>
<li>
<strong>Development time</strong>: Effectively 1 day</li>
<li>
<strong>Cost</strong>: Domain only (Cloudflare Pages and CDN are free)</li>
<li>
<strong>Code</strong>: ~2,000 lines of TypeScript</li>
</ul>

<p>I didn't "have AI build it for me." I used AI as a booster to bring my own obsessions to life. The arrow's pinch, the fight with iOS Safari — every decision was mine. AI was the tool to execute them.</p>

<p>The two weeks I spent going nowhere with Flutter happened because I'd shifted from "eliminate friction" to "which platform should I ship on." That detour is what led me to the right answer: PWA.</p>




<p>If you're the kind of person who gets frustrated by how long it takes to draw one arrow on a screenshot — give it a try.</p>

<p><strong><a href="https://annotate.puremark.app" rel="noopener noreferrer">Pure Mark Annotate — Use it now in your browser</a></strong></p>

