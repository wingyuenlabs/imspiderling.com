---
Title: 1,500 Baby Names and Counting: What I Learned Building a Multilingual Name Database
Description: 
Author: Yunhan
Date: 2026-03-18T22:14:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>We just crossed 1,500 baby names on <a href="https://babynamepick.com" rel="noopener noreferrer">BabyNamePick.com</a> — covering 30+ origins from Irish to Japanese to Polynesian. Here's what I learned building a multilingual name database from scratch.</p>

<h2>
  
  
  The Challenge
</h2>

<p>Most baby name sites focus on English names. But parents today come from everywhere, and they want names that reflect their heritage. We set out to build a database that covers:</p>

<ul>
<li>
<strong>Western European</strong>: Irish, British, French, German, Italian, Spanish, Portuguese</li>
<li>
<strong>Nordic &amp; Celtic</strong>: Nordic, Scottish, Welsh, Cornish</li>
<li>
<strong>Eastern European</strong>: Russian, Slavic, Polish, Czech</li>
<li>
<strong>Middle Eastern</strong>: Arabic, Hebrew, Persian, Turkish</li>
<li>
<strong>South Asian</strong>: Indian, Sanskrit</li>
<li>
<strong>East Asian</strong>: Chinese, Japanese, Korean, Vietnamese, Thai</li>
<li>
<strong>African</strong>: Pan-African names from multiple traditions</li>
<li>
<strong>Pacific</strong>: Hawaiian, Polynesian, Filipino</li>
<li>
<strong>Americas</strong>: American, Latin American</li>
</ul>

<h2>
  
  
  Data Structure
</h2>

<p>Each name entry looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Akira"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"meaning"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Bright and clear"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"origin"</span><span class="p">:</span><span class="w"> </span><span class="s2">"japanese"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"gender"</span><span class="p">:</span><span class="w"> </span><span class="s2">"unisex"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"styles"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"classic"</span><span class="p">,</span><span class="w"> </span><span class="s2">"strong"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The <code>styles</code> array is key for discovery. Parents don't always search by origin — they search by <em>vibe</em>: "nature names," "strong names," "vintage names." Our style tags include:</p>

<ul>
<li>classic, modern, vintage, elegant</li>
<li>nature, strong, short, whimsical</li>
<li>biblical, mythological, royal, literary</li>
<li>musical, spiritual, space</li>
</ul>

<h2>
  
  
  Lessons Learned
</h2>

<h3>
  
  
  1. Romanization is tricky
</h3>

<p>Chinese names like "Qian" or Japanese names like "Koharu" need careful romanization. We standardized on the most common English transliteration for each name.</p>

<h3>
  
  
  2. Gender varies by culture
</h3>

<p>A name that's strictly male in one culture might be unisex in another. "Kai" is male in Japanese but unisex in Hawaiian. We went with the most internationally recognized gender assignment.</p>

<h3>
  
  
  3. Meanings are layered
</h3>

<p>Many names have multiple meanings across languages. "Tao" means "the way" in Chinese philosophy but also "peach." We include the most culturally significant meaning first.</p>

<h3>
  
  
  4. US Top 100 names drive traffic
</h3>

<p>No matter how interesting your rare names are, the bulk of search traffic comes from names like Liam, Olivia, Emma, and Noah. You need both — popular names for traffic, unique names for differentiation.</p>

<h3>
  
  
  5. Static generation scales beautifully
</h3>

<p>With Next.js SSG, we generate 3,500+ pages at build time. Each name gets its own page, each origin gets a category page, and each style gets a browsing page. Build time is under 2 minutes.</p>

<h2>
  
  
  What's Next
</h2>

<ul>
<li>
<strong>2,000 names</strong> by end of March</li>
<li>
<strong>FAQ schema</strong> on every category page for richer search results</li>
<li>
<strong>Name comparison</strong> feature for parents deciding between two names</li>
<li>
<strong>Pronunciation guides</strong> with audio for non-English names</li>
</ul>

<p>Check it out at <a href="https://babynamepick.com" rel="noopener noreferrer">babynamepick.com</a> and let me know what names we're missing!</p>




<p><em>Building in public. Follow along for more updates on SEO, static sites, and indie projects.</em></p>

