---
Title: How to Add OG Images to Your Website (Manual vs. Automated)
Description: 
Author: Narender singh
Date: 2026-03-24T21:49:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every time you share a link on Twitter, Slack, Discord, or LinkedIn, the platform fetches your page's meta tags to build a preview card. The og:image tag controls what image shows up. If you don't set one, you get either nothing or whatever the crawler decides to grab from your page. Usually it picks your logo at 50x50 pixels. Looks awful.</p>

<p>I've spent way too long manually creating OG images in Figma, so here's what I've learned about the different ways to handle this.</p>

<h2>
  
  
  What og:image actually does
</h2>

<p>It's a meta tag in your HTML head:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;meta</span> <span class="na">property=</span><span class="s">"og:image"</span> <span class="na">content=</span><span class="s">"https://yoursite.com/og-image.png"</span> <span class="nt">/&gt;</span>
</code></pre>

</div>



<p>Social platforms read this when someone pastes your URL. Facebook, Twitter, LinkedIn, Discord, Slack — they all use it. The recommended size is 1200x630 pixels. Some platforms crop differently, but 1200x630 is the safe bet that works everywhere.</p>

<p>You also want og:title and og:description, but the image is what people actually notice. A link with a good preview image gets clicked. A link without one gets scrolled past.</p>

<h2>
  
  
  The manual way
</h2>

<p>This is how most people start:</p>

<ol>
<li>Open Figma (or Canva, or whatever)</li>
<li>Create a 1200x630 canvas</li>
<li>Design something with your title, maybe a logo, some background</li>
<li>Export as PNG</li>
<li>Upload to your public assets</li>
<li>Add the meta tag pointing to that file</li>
</ol>

<p>It works fine if you have 5 pages. It falls apart when you have 50 blog posts and each one needs a unique image with the post title on it. I did this for a while. Every time I published something new, I'd spend 10 minutes in Figma tweaking text alignment. Not the worst use of time, but definitely repetitive.</p>

<h2>
  
  
  The automated way
</h2>

<p>The idea is simple: instead of a static image URL, you use an API endpoint that generates the image on the fly from URL parameters.</p>

<p>Your meta tag ends up looking like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;meta</span> <span class="na">property=</span><span class="s">"og:image"</span> <span class="na">content=</span><span class="s">"https://ogpix-pi.vercel.app/api/og?title=My+Blog+Post&amp;theme=dark"</span> <span class="nt">/&gt;</span>
</code></pre>

</div>



<p>When a social platform crawls your page, it hits that URL, the API renders an image with your title text, and returns it. No Figma. No uploading files. The image is generated from the URL itself.</p>

<p>I built <a href="https://ogpix-pi.vercel.app/playground" rel="noopener noreferrer">OGPix</a> to do exactly this. You can pick from different themes, set your title and description as URL params, and get back a 1200x630 PNG. But the concept works with any OG image API — the integration pattern is the same regardless.</p>

<h2>
  
  
  Code examples
</h2>

<h3>
  
  
  Plain HTML
</h3>

<p>The simplest case. Just hardcode the meta tags:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;head&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">property=</span><span class="s">"og:title"</span> <span class="na">content=</span><span class="s">"My Page Title"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">property=</span><span class="s">"og:description"</span> <span class="na">content=</span><span class="s">"A short description of the page"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">property=</span><span class="s">"og:image"</span> <span class="na">content=</span><span class="s">"https://ogpix-pi.vercel.app/api/og?title=My+Page+Title&amp;theme=gradient"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">property=</span><span class="s">"og:image:width"</span> <span class="na">content=</span><span class="s">"1200"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">property=</span><span class="s">"og:image:height"</span> <span class="na">content=</span><span class="s">"630"</span> <span class="nt">/&gt;</span>
<span class="nt">&lt;/head&gt;</span>
</code></pre>

</div>



<p>Include the width and height tags. Without them, some platforms delay rendering the preview because they have to fetch the image first to figure out dimensions.</p>

<h3>
  
  
  Next.js (App Router)
</h3>

<p>In Next.js 13+ with the app router, you export a metadata object:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// app/blog/[slug]/page.tsx</span>
<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">generateMetadata</span><span class="p">({</span> <span class="nx">params</span> <span class="p">})</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">post</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">getPost</span><span class="p">(</span><span class="nx">params</span><span class="p">.</span><span class="nx">slug</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">ogImage</span> <span class="o">=</span> <span class="err">\</span><span class="s2">`https://ogpix-pi.vercel.app/api/og?title=</span><span class="se">\$</span><span class="s2">{encodeURIComponent(post.title)}&amp;theme=minimal</span><span class="se">\`</span><span class="s2">;

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [{ url: ogImage, width: 1200, height: 630 }],
    },
    twitter: {
      card: "summary_large_image",
      images: [ogImage],
    },
  };
}
</span></code></pre>

</div>



<p>The nice thing here is that each blog post automatically gets its own OG image with the correct title. No manual work per post.</p>

<h3>
  
  
  Astro
</h3>

<p>Astro handles this in the layout's head:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>---
// src/layouts/BlogPost.astro
const { title, description } = Astro.props;
const ogImage = \`https://ogpix-pi.vercel.app/api/og?title=\${encodeURIComponent(title)}&amp;theme=ocean\`;
---
&lt;html&gt;
  &lt;head&gt;
    &lt;meta property="og:title" content={title} /&gt;
    &lt;meta property="og:description" content={description} /&gt;
    &lt;meta property="og:image" content={ogImage} /&gt;
    &lt;meta property="og:image:width" content="1200" /&gt;
    &lt;meta property="og:image:height" content="630" /&gt;
    &lt;meta name="twitter:card" content="summary_large_image" /&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;slot /&gt;
  &lt;/body&gt;
&lt;/html&gt;
</code></pre>

</div>



<p>Same pattern. Build the URL from your page data, drop it in the meta tag.</p>

<h2>
  
  
  Testing your OG tags
</h2>

<p>After adding the tags, you need to actually verify they work. Social platforms cache aggressively, and it's easy to have a typo in your URL that you won't notice until someone shares your link and it looks broken.</p>

<p>I built a free <a href="https://ogpix-pi.vercel.app/checker" rel="noopener noreferrer">OG Tag Checker</a> that shows you exactly how your link will appear on Twitter, Facebook, and LinkedIn. It also flags missing tags. I use it every time before I publish something.</p>

<p>Twitter also has its own <a href="https://cards-dev.twitter.com/validator" rel="noopener noreferrer">Card Validator</a>, and Facebook has the <a href="https://developers.facebook.com/tools/debug/" rel="noopener noreferrer">Sharing Debugger</a>. Both are useful for clearing cached previews after you make changes.</p>

<h2>
  
  
  Quick checklist
</h2>

<ul>
<li>Set og:image to a 1200x630 image URL</li>
<li>Include og:image:width and og:image:height</li>
<li>Set twitter:card to "summary_large_image" if you want the big preview</li>
<li>Make sure the image URL is absolute (starts with https://)</li>
<li>Test with a checker tool before publishing</li>
<li>Remember that platforms cache previews, so use their debug tools to refresh</li>
</ul>

<p>If you want to play around with different OG image styles before integrating, the <a href="https://ogpix-pi.vercel.app/playground" rel="noopener noreferrer">OGPix playground</a> lets you preview themes and copy the URL. It's free, no account needed.</p>

<p>Got questions about OG images or a different setup you'd like to see covered? Drop a comment.</p>

