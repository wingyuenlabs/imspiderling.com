---
Title: Maravel-Framework 10.61.9 Benchmarks vs Lumen and Laravel
Description: 
Author: marius-ciclistu
Date: 2026-02-08T21:00:27.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiwd2vcyfcpp1bjo1yjmb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiwd2vcyfcpp1bjo1yjmb.png" width="300" height="451"></a><br>
<em>Maravel-Framework 10.61.9</em></p>

<p>Thanks to <a href="https://github.com/myaaghubi/PHP-Frameworks-Bench/issues/82#issuecomment-3867119246" rel="noopener noreferrer">https://github.com/myaaghubi/PHP-Frameworks-Bench</a> I was able to benchmark Maravel Micro-Framework 10.52.14 vs Lumen 10 and Maravelith 10.52.8 vs Laravel 10.3.</p>

<p>System:</p>

<p>PHP 8.1</p>

<p>Ubuntu Mate 22.04</p>

<p>GIGABYTE GA-M52S-S3P rev 1.0</p>

<p>AMD Phenom ll x2 550 3.1 ghz</p>

<p>8GB 4x4GB Samsung M378T5263AZ3-CF7 PC2–6400U-666 4GB 2Rx8 800MHz 240-pin DIMM Non-ECC DDR2</p>

<p>GPU NVIDIA Corporation G73 <a href="https://dev.torev%20a1">GeForce 7300 GT</a></p>

<h3>
  
  
  Results for “Hello World”:
</h3>

<h3>
  
  
  <strong>1. Without cache commands:</strong>
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs4cvdhr0nlpkqh0nvara.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs4cvdhr0nlpkqh0nvara.png" width="800" height="110"></a></p>

<p>Maravel is slower than Lumen because of the recent added features:</p>

<ul>
<li>circular dependency prevention</li>
<li>autowiring:cache</li>
</ul>

<p>Maravelith has approximately the same results as Laravel with the above new features + event:cache that include observers (which is disabled by default in Maravel)</p>

<p>In terms of memory, only Maravelith has 1% more memory usage vs Laravel, while Maravel and Lumen share the same amount.</p>

<h3>
  
  
  2. With these cache commands:
</h3>

<p>Maravel: config:cache, route:cache, autowiring:cache</p>

<p>Lumen: n/a</p>

<p>Maravelith: config:cache, route:cache, autowiring:cache, event:cache</p>

<p>Laravel: config:cache, route:cache, event:cache</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftgp7qvyom27r5vbxtnkm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftgp7qvyom27r5vbxtnkm.png" width="800" height="110"></a></p>

<p>Maravel is 62% faster than Lumen 10 and Maravelith is 4% faster than Laravel 10.</p>

<p>Maravelith is 57% faster with these commands while Laravel 10 is 53% faster if compared to their respective results without cache.</p>

<p>In terms of memory, Maravel used 5% less, Maravelith 26.66% less and Laravel 24.72% less if compared with their respective results without cache.</p>

<p>When comparing Maravelith with Laravel, the first uses 1.5% less memory.</p>

