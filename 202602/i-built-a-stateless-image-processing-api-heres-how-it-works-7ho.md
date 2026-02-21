---
Title: I Built a Stateless Image Processing API — Here's How It Works
Description: 
Author: TheGlitch
Date: 2026-02-21T21:41:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every web project eventually runs into the same problem: you need to resize an image, convert a format, maybe add some effects, or strip a background. You can set up ImageMagick, wrestle with Sharp, or build custom pipelines — or you can make one HTTP call.</p>

<p>That's why I built <strong>TheGlitch</strong> — a stateless image processing API that handles everything in a single request.</p>

<h2>
  
  
  The Problem
</h2>

<p>Image processing infrastructure is surprisingly complex:</p>

<ul>
<li>Installing native dependencies on different OS targets</li>
<li>Managing temporary files and disk cleanup</li>
<li>Handling format quirks (AVIF support, GIF animation, color profiles)</li>
<li>GPU acceleration for AI operations like background removal</li>
<li>Scaling under load without leaking memory</li>
</ul>

<p>I wanted an API where you send an image in and get a processed image out. Nothing stored, nothing cached on disk, no state between requests.</p>

<h2>
  
  
  How It Works
</h2>

<p>TheGlitch processes images entirely in memory. The pipeline looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Input (URL/base64/binary/form) 
  → Decode &amp; Validate 
  → Resize &amp; Crop 
  → Apply Effects 
  → Format Convert 
  → Return Binary
</code></pre>

</div>



<p>A single GET request can do everything:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="s2">"https://theglitch.p.rapidapi.com/api/v1/process?</span><span class="se">\</span><span class="s2">
url=https://picsum.photos/1000&amp;</span><span class="se">\</span><span class="s2">
width=800&amp;</span><span class="se">\</span><span class="s2">
format=webp&amp;</span><span class="se">\</span><span class="s2">
brightness=10&amp;</span><span class="se">\</span><span class="s2">
contrast=15&amp;</span><span class="se">\</span><span class="s2">
sharpen=20"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-RapidAPI-Key: YOUR_KEY"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-RapidAPI-Host: theglitch.p.rapidapi.com"</span> <span class="se">\</span>
  <span class="nt">-o</span> result.webp
</code></pre>

</div>



<h2>
  
  
  Features
</h2>

<p><strong>Resize &amp; Crop</strong> — Four modes: fit (preserve aspect ratio), fill (crop to exact size), pad (add borders), stretch. Resolutions up to 8000x8000px.</p>

<p><strong>Format Conversion</strong> — Input/output: JPEG, PNG, WebP, GIF, BMP, TIFF. Quality control per format.</p>

<p><strong>7 Visual Effects</strong> — Brightness, contrast, saturation, blur, sharpen, grayscale, sepia. All combinable in one request.</p>

<p><strong>AI Background Removal</strong> — GPU-powered, takes about 3 seconds per image. Returns transparent PNG.</p>

<p><strong>14 Social Media Presets</strong> — Instagram square, Facebook cover, YouTube thumbnail, LinkedIn banner, and more. One parameter instead of remembering dimensions.</p>

<h2>
  
  
  Code Examples
</h2>

<p><strong>JavaScript:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span>
  <span class="dl">'</span><span class="s1">https://theglitch.p.rapidapi.com/api/v1/process?url=IMAGE_URL&amp;width=800&amp;format=webp</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">{</span> <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">X-RapidAPI-Key</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">YOUR_KEY</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">X-RapidAPI-Host</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">theglitch.p.rapidapi.com</span><span class="dl">'</span> <span class="p">}</span> <span class="p">}</span>
<span class="p">);</span>
<span class="kd">const</span> <span class="nx">blob</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">blob</span><span class="p">();</span>
</code></pre>

</div>



<p><strong>Python:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">https://theglitch.p.rapidapi.com/api/v1/process</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">params</span><span class="o">=</span><span class="p">{</span><span class="sh">'</span><span class="s">url</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">IMAGE_URL</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">width</span><span class="sh">'</span><span class="p">:</span> <span class="mi">800</span><span class="p">,</span> <span class="sh">'</span><span class="s">format</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">webp</span><span class="sh">'</span><span class="p">},</span>
    <span class="n">headers</span><span class="o">=</span><span class="p">{</span><span class="sh">'</span><span class="s">X-RapidAPI-Key</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">YOUR_KEY</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">X-RapidAPI-Host</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">theglitch.p.rapidapi.com</span><span class="sh">'</span><span class="p">}</span>
<span class="p">)</span>

<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">'</span><span class="s">result.webp</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">wb</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="n">f</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">content</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  Why Stateless?
</h2>

<p>Every image is processed in memory and discarded after the response is sent. This means:</p>

<ul>
<li>
<strong>GDPR compliant by design</strong> — no user data stored, ever</li>
<li>
<strong>No disk I/O bottleneck</strong> — everything happens in RAM</li>
<li>
<strong>Predictable scaling</strong> — each request is independent</li>
<li>
<strong>No cleanup jobs</strong> — nothing to garbage collect</li>
</ul>

<h2>
  
  
  Architecture Decisions
</h2>

<p>A few things I learned while building this:</p>

<ol>
<li><p><strong>SkiaSharp over ImageMagick</strong> — Native performance, cross-platform, no external dependencies. The tradeoff is less format support (no real AVIF encoding yet), but WebP covers most use cases.</p></li>
<li><p><strong>Replicate for GPU ops</strong> — Instead of running my own GPU server, I proxy AI operations through Replicate. Background removal costs about $0.0014 per image with BiRefNet. Cold starts are free for public models.</p></li>
<li><p><strong>Separate CPU and GPU rate limits</strong> — CPU operations (resize, effects, format) are cheap. GPU operations (background removal) are expensive. Different limits per plan make pricing fair.</p></li>
<li><p><strong>Single VPS deployment</strong> — Docker Compose with Caddy as reverse proxy, Cloudflare in front for CDN/DDoS/SSL. Total infrastructure cost: under $6/month.</p></li>
</ol>

<h2>
  
  
  API Endpoints
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Endpoint</th>
<th>What it does</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/api/v1/process</code></td>
<td>Full pipeline — resize + effects + format</td>
</tr>
<tr>
<td><code>/api/v1/resize</code></td>
<td>Resize only</td>
</tr>
<tr>
<td><code>/api/v1/convert</code></td>
<td>Format conversion</td>
</tr>
<tr>
<td><code>/api/v1/effects</code></td>
<td>Visual effects</td>
</tr>
<tr>
<td><code>/api/v1/remove-bg</code></td>
<td>AI background removal (GPU)</td>
</tr>
<tr>
<td><code>/api/v1/optimize</code></td>
<td>Auto-optimize for web (WebP)</td>
</tr>
<tr>
<td><code>/api/v1/preset/{name}</code></td>
<td>Social media presets</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Try It
</h2>

<p>The API is live with a free tier (500 requests/month). Check out the before/after examples on the website, or try it directly through RapidAPI:</p>

<ul>
<li>Website with live examples: <a href="https://theglitch.app" rel="noopener noreferrer">theglitch.app</a>
</li>
<li>API on RapidAPI: <a href="https://rapidapi.com/theglitchapp/api/theglitch-image-processing" rel="noopener noreferrer">TheGlitch Image Processing</a>
</li>
</ul>




<p>I'd love to hear what features you'd find useful. Background removal was the most requested during beta — what would you want next?</p>

