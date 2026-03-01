---
Title: 📨 Pop-out Messaging Extension: Dev Whisper
Description: 
Author: Anna Villarreal
Date: 2026-03-01T21:39:45.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/weekend-2026-02-28">DEV Weekend Challenge: Community</a></em></p>

<p>This is the fastest I've ever made something functional.</p>

<p>- b r e a t h e s -</p>

<h2>
  
  
  The Community
</h2>

<p>For the past couple years, I have actively used dev as a resource. Learning, discovering, and trying things way out of my league. Because of this, I have learned more than I would have on my own with no other support. When you enter the realm of web development and you don't really have anyone close to you doing it, it's the wild west. Dev has provided a place for all walks of life to learn and push forward to discover new things. For this reason, I chose Dev as my community!</p>

<p>The one thing I noticed about dev is there there was no p2p chat. I thought it would be cool if users could send simple messages together about cool projects, ect. If the chat is implemented by way of an optional extension, then we are never forcing people to deal with the chat. It's truly a personal decision. </p>

<p>Thinking from a community standpoint - I wanted to include all the meme monday memes inside the gif picker in the chat. YES you can probably find your meme you posted 2 months ago. This way, I'm using community data, for the community, to continue enhancing the experience. </p>



<h2>
  
  
  What I Built
</h2>

<p>A free open source messaging app for dev members to communicate with each other without having to share personal information. It's shipped as an extension but has a pop-out option for ease of use. </p>



<h2>
  
  
  Demo
</h2>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/4RGimHksJgY">
</iframe>
</p>



<h2>
  
  
  Code
</h2>


<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/AnnaVi11arrea1" rel="noopener noreferrer">
        AnnaVi11arrea1
      </a> / <a href="https://github.com/AnnaVi11arrea1/dev_messages" rel="noopener noreferrer">
        dev_messages
      </a>
    </h2>
    <h3>
      3rd party messaging app that works with Dev.to
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">Dev.to Messages (Unofficial 3rd Party Extension)</h1>
</div>
<p>A Chrome extension that adds private direct messaging to <a href="https://dev.to" rel="nofollow">Dev.to</a>, with built-in link safety, spam reporting, and a first-contact approval system.</p>

<div class="markdown-heading">
<h2 class="heading-element">File Structure</h2>
</div>
<div class="snippet-clipboard-content notranslate position-relative overflow-auto"><pre class="notranslate"><code>dev_messages/
├── manifest.json          MV3 manifest
├── background.js          Service worker – updates unread badge
├── content.js             Injected into dev.to – detects user, adds Message buttons
├── popup.html             All UI views &amp; modals
├── popup.css              Dev.to-inspired styles (380px popup)
├── popup.js               Full SPA app logic
├── js/
│   ├── storage.js         chrome.storage.local wrapper
│   ├── eligibility.js     Dev.to API eligibility check
│   └── linkSafety.js      URL spoofing detection + safe renderer
└── icons/                 16 / 48 / 128 px PNG icons
</code></pre></div>

<div class="markdown-heading">
<h2 class="heading-element">Features</h2>
</div>
<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Details</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Account eligibility</strong></td>
<td>Checks <code>joined_at</code> &amp; <code>articles_count</code> via the Dev.to public API. Account must be at least <strong>30 days old</strong> and have <strong>at least 1 published post</strong> before sending or receiving messages.</td>
</tr>
<tr>
<td><strong>First-contact approval</strong></td>
<td>When someone messages you for the first time,</td>
</tr>
</tbody>
</table></div>…</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/AnnaVi11arrea1/dev_messages" rel="noopener noreferrer">View on GitHub</a></div>
</div>


<h3>
  
  
  Meme Search Implementation
</h3>

<p>Arguably, the most important feature. XD</p>

<p>I wanted not to just pull in the memes, but to allow a user to search them. The best way I figured was to use the alt tags as search parameters. Many of us are nice enough to put alt tags on our stuff, and so I was able to leverage that to get a sort of lazy search going. Hey, it works. I thought that was a little cool and different so that is the snippet I'll be sharing below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/* gifPicker.js – Fetches meme images from Ben Halpern's Meme Monday posts,
   including every image posted in the comments (community memes). */

const GifPicker = {
  _cache: null,

  async fetchImages() {
    if (this._cache) return this._cache;

    /* ── Step 1: collect all Meme Monday articles ─────────────────────────── */
    const articles = [];
    for (let page = 1; page &lt;= 6; page++) {
      try {
        const res = await fetch(
          `https://dev.to/api/articles?username=ben&amp;per_page=100&amp;page=${page}`
        );
        if (!res.ok) break;
        const data = await res.json();
        if (!data.length) break;
        for (const a of data) {
          if (/meme\s+monday/i.test(a.title)) articles.push(a);
        }
        if (data.length &lt; 100) break;
      } catch { break; }
    }

    /* ── Step 2: collect images with dedup ────────────────────────────────── */
    const seen   = new Set();
    const images = [];
    const add    = (url, title) =&gt; {
      if (!url || seen.has(url)) return;
      seen.add(url);
      images.push({ url, title });
    };

    /* Cover images first so they appear at the top of the grid */
    for (const a of articles) {
      if (a.cover_image) add(a.cover_image, a.title);
    }

    /* ── Step 3: fetch all comment threads in parallel ────────────────────── */
    const commentThreads = await Promise.all(
      articles.map(a =&gt;
        fetch(`https://dev.to/api/comments?a_id=${a.id}`)
          .then(r =&gt; r.ok ? r.json() : [])
          .catch(() =&gt; [])
      )
    );

    for (let i = 0; i &lt; articles.length; i++) {
      this._extractImages(commentThreads[i], articles[i].title, add);
    }

    this._cache = images;
    return images;
  },

  /* Recursively walk comment tree and pull every &lt;img src="…"&gt; with its own alt */
  _extractImages(comments, articleTitle, add) {
    for (const comment of comments) {
      const imgTags = (comment.body_html || '').matchAll(/&lt;img([^&gt;]+)&gt;/gi);
      for (const m of imgTags) {
        const attrs  = m[1];
        const srcM   = attrs.match(/src="([^"]+)"/i);
        const altM   = attrs.match(/alt="([^"]*)"/i);
        if (srcM) {
          const alt = altM?.[1]?.trim() || articleTitle;
          add(srcM[1], alt);
        }
      }
      if (comment.children?.length) {
        this._extractImages(comment.children, articleTitle, add);
      }
    }
  },
};

</code></pre>

</div>





<h2>
  
  
  How I Built It
</h2>

<p>I very quickly realized that I was going to need a secure database so users could keep their chat history. I'm using Neon's free tier for this project. Give it a whirl! XD. Requests from the extension go to Vercel over HTTPS. Vercel connects to Neon with an encrypted connection. The database is never exposed to the internet, and it is only queried by Vercel's serverless functions. </p>

<p>I used copilot CLI to quickly build up a working template - I would not be able to build something so quickly without it. This is not my first chrome extension, but it is the 'fanciest' one I have made to date. </p>

<p>I tested with my unpacked files in the google chrome extensions. I was lucky enough to test functionality with another dev user. That was amazingly helpful. A big thank you to <a class="mentioned-user" href="https://dev.to/trickell">@trickell</a> for being awesome.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flqc60t60kjf7xwrl2dxb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flqc60t60kjf7xwrl2dxb.png" alt="coolest chat ever" width="800" height="618"></a></p>

<p>I had fun making a settings and options area for users which includes colors and text sizes for increased accessibility.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl5cpnl1xpa6cb127u3q1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl5cpnl1xpa6cb127u3q1.png" alt="customize settings" width="800" height="615"></a></p>



<h3>
  
  
  Features:
</h3>

<p><small><em>Created with the mindset of reducing spam and annoyances</em></small><br>
<br></p>

<ul>
<li><p>Account eligibility - Checks <code>joined_at</code> &amp; <code>articles_count</code> via the Dev.to public API. Account must be at least <strong>30 days old</strong> and have <strong>at least 1 published post</strong> before sending or receiving messages. </p></li>
<li><p>First-contact approval -  When someone messages you for the first time, you see a banner with their opening message and must hit <strong>Approve</strong> or <strong>Deny</strong> before the conversation unlocks. </p></li>
<li><p>Flag / report spam - Every incoming message has a 🚩 button. Clicking it opens a reason picker (spam, phishing, harassment, inappropriate, other) and marks the message as reported. </p></li>
<li><p>Link spoofing detection - Before any link opens, <code>LinkSafety.isSpoofed()</code> compares the visible URL text against the actual <code>href</code> hostname. A red alert is shown if they differ.</p></li>
<li><p>Link safety warning - Every link in every message routes through a warning modal that displays the full destination URL before opening it in a new tab. </p></li>
<li><p>Local storage persistence - All conversations, approvals, flags, and user data are stored in <code>chrome.storage.local</code>. The extension must be installed and active to read messages. </p></li>
<li><p>Unread badge - The extension icon shows a live unread count, updated by the background service worker whenever storage changes</p></li>
</ul>

<p>My extension is getting reviewed by google currently... whish me luck! haha</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F38xqrnzmaumz8zdrvvz7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F38xqrnzmaumz8zdrvvz7.png" alt="pending review" width="632" height="120"></a></p>

<p>A user must have the extension to test it. Guess I've made myself a guinea pig haven't I. (hides, 😅)</p>

