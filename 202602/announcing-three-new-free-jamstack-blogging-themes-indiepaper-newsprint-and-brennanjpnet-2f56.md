---
Title: Announcing Three New Free JAMstack Blogging Themes: IndiePaper, Newsprint, and brennan.jp.net
Description: 
Author: Brennan K. Brown
Date: 2026-02-10T21:49:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ask anybody on the IndieWeb what they think about blogging, and you'll most likely get a rather passionate, long-winded answer. <a href="https://blog.brennanbrown.ca/the-dying-art-of-having-something-to-say-68f4e77d09fc0" rel="noopener noreferrer">I'm no different</a>. I started getting into web development over a decade ago with Jekyll and the announcement of Ghost when I was still in high school. </p>

<p>As the years have gone by, I've stayed in this cozy, somewhat obscure niche of <a href="https://jamstack.org/" rel="noopener noreferrer">JAMstack development</a>, making static site themes and projects that are open source and free for others to use however they like. This is exactly why I founded <a href="https://berryhouse.ca" rel="noopener noreferrer">🍓 Berry House</a> in the first place, because I believe having your own website is just getting more and important and useful in today's age of corpo-AI slopfests.</p>

<p>After I received a donation yesterday from the lovely <a href="https://binarydigit.dev/" rel="noopener noreferrer">BinaryDigit</a>, crediting me for the 11ty theme I made previously called <a href="https://github.com/brennanbrown/retroweird" rel="noopener noreferrer">RetroWeird</a>, I realized I needed to get back into making blog themes.</p>

<p>As fun as RetroWeird is visually, I can't imagine it would actually be easy to use as a blog theme, I would imagine there would be a lot of wrestling with it. (<a href="https://github.com/brennanbrown/hyperpop" rel="noopener noreferrer">Hyperpop</a> is also a fun theme I made as well, but it has similar problems.) And even though it's only been a few months since I've made these themes, I've already learned a lot since then. Sometimes, like actual property, it is easier to start from scratch than try to do renovations.</p>

<p>I needed to return to form. Start from scratch and take what I've learned over the past few months and make the best possible themes I could for people that want to get onto the IndieWeb.</p>

<p>With these themes, or any themes for Hugo, 11ty, Jekyll, Pelican, etc. All you need is a git repository, whether on GitLab or Codeberg or elsewhere, hook it up to a service like Netlify or Vercel, and you have your own blog for free. The only thing that'll cost is a domain name. No databases, no monthly fees, no limitations to customization. You write your blog posts in plain-text markdown files however you like, and that's it.</p>

<p>With all that out of the way, let's actually get to the themes!</p>

<h2>
  
  
  📰 Indiepaper
</h2>

<p><a href="https://indiepaper.netlify.app" rel="noopener noreferrer"><br>
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgitlab.com%2Fbrennankbrown%2Fbrennan.day%2F-%2Fraw%2Fmain%2Fsrc%2Fassets%2Fimages%2Fblog%2Findiepaper-screenshot.jpg" alt="Indiepaper theme screenshot showing a minimalist brutalist blog design" width="800" height="452"><br>
</a></p>
Indiepaper theme featuring a simple, brutalist design with smolweb compliance and microformats2 support



<p><strong>Platform:</strong> Hugo<br><br>
<strong>Philosophy:</strong> Smolweb compliance meets brutalist design<br><br>
<strong>Repository:</strong> <a href="https://github.com/brennanbrown/indiepaper" rel="noopener noreferrer">github.com/brennanbrown/indiepaper</a><br><br>
<strong>Demo:</strong> <a href="https://indiepaper.netlify.app" rel="noopener noreferrer">indiepaper.netlify.app</a></p>

<p>This is the first theme I started on, and originally was going to be the only theme I was going to be making before procrastination and scope creep got the better of me. Indiepaper takes inspiration from the brutalist web design movement with a greyscale aesthetic.</p>

<p>Indiepaper has microformats2 support built in, <code>h-card</code> for author identity, <code>h-entry</code> for blog posts, <code>h-feed</code> for listings, and <code>h-cite</code> for webmentions. </p>

<p>The purpose of Indiepaper is to not only be IndieWeb friendly, but also <a href="https://smolweb.org" rel="noopener noreferrer">smolweb</a> compliant. This means that it is, well, small! There's no embedded fonts or JavaScript or CSS frameworks. It's designed to be as minimal as possible, what you see is what you get. This means it is easy for people to customize it to their hearts content without breaking anything, because there's so little to break!</p>

<h3>
  
  
  Typography Without External Dependencies
</h3>

<p>I've been someone who relied on Google Fonts for a long while because I was only aware of a handful of universally-compatible fonts (none of which are rather attractive), and while I do love the service still, I found out about <a href="https://modernfontstacks.com/" rel="noopener noreferrer">Modern Font Stacks</a> which will give you an array of typefaces based on classification that are available for every widely-used system: Windows, MacOS, Ubuntu, iOS, and Android. </p>

<p>As somebody who also <em>loves</em> Garamond, I decided to use Old Style fonts for the body, and Geometric Humanist fonts for the headers. I believe this lends itself to both easy readability and a more modern look. The body text uses old-style serifs like Iowan Old Style and Palatino Linotype, while headers use geometric humanist sans-serifs like Avenir, Montserrat, and Corbel. Code blocks use modern monospace fonts including ui-monospace, Cascadia Code, and Source Code Pro.</p>

<h2>
  
  
  🗞️ Newsprint
</h2>

<p><a href="https://newsprint.netlify.app" rel="noopener noreferrer"><br>
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgitlab.com%2Fbrennankbrown%2Fbrennan.day%2F-%2Fraw%2Fmain%2Fsrc%2Fassets%2Fimages%2Fblog%2Fnewsprint-screenshot.jpg" alt="Newsprint theme screenshot showing a newspaper-style blog layout" width="800" height="638"><br>
</a></p>
Newsprint theme with traditional newspaper aesthetics, masthead, and multi-column layouts



<p><strong>Platform:</strong> Eleventy (11ty)<br><br>
<strong>Philosophy:</strong> Newsletter-first publication meets classic newspaper design<br><br>
<strong>Repository:</strong> <a href="https://github.com/brennanbrown/newsprint" rel="noopener noreferrer">github.com/brennanbrown/newsprint</a><br><br>
<strong>Demo:</strong> <a href="https://newsprint.netlify.app" rel="noopener noreferrer">newsprint.netlify.app</a></p>

<p>While making Indiepaper, I realized how cool it would be to have a blog that actually <em>looked</em> like a newspaper. I wanted to create something <a href="https://fitefuaite.com/journal/comhra/skeuomorphic-design/" rel="noopener noreferrer">§skeuomorphic</a>, omg.lol's <a href="https://omglol.news/" rel="noopener noreferrer">news and updates</a> actually already done this really well in a simple way! And I found this helpful <a href="https://codepen.io/silkine/pen/QWBxVX" rel="noopener noreferrer">codepen snippet</a> created by Silke V. which helped me with the initial design phase. </p>

<p>Not only did I want a newspaper look, but I figured the theme should also embody other newspaper ideals, such as having a thoughtfully designed RSS so it can be delivered as a newspaper as well, right? (And really good print styles to boot!)</p>

<p>I did this by having full-content feeds using email-safe HTML, plus category-specific feeds for News, Opinion, Features, Culture, and Business. Each article can be categorized, and readers can subscribe to specific topics they care about. The main feed is available at <code>/feed.xml</code>, with individual category feeds at <code>/feed/news.xml</code>, <code>/feed/opinion.xml</code>, and so on.</p>

<p>Newsprint has multi-column layouts, a traditional masthead, drop caps, pull quotes, ruled lines, and a sepia palette. The sidebar features follow links for configured social platforms, and there's built-in donation support through Ko-fi, Patreon, or any platform you prefer. The donation section can be easily hidden if you don't want to monetize.</p>

<p>Articles use a front matter system including title, subtitle (deck), publication date, author, category, featured status for homepage placement, excerpt for RSS and meta descriptions, and optional featured images with captions. The homepage features a dedicated featured article section and a grid of the latest nine articles, excluding the featured one.</p>

<h2>
  
  
  🎋 brennan.jp.net
</h2>

<p><a href="https://brennan.jp.net" rel="noopener noreferrer"><br>
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgitlab.com%2Fbrennankbrown%2Fbrennan.day%2F-%2Fraw%2Fmain%2Fsrc%2Fassets%2Fimages%2Fblog%2Fbrennanjpnet-screenshot.jpg" alt="brennan.jp.net theme screenshot showing colorful Japanese web design aesthetic" width="800" height="442"><br>
</a></p>
brennan.jp.net theme featuring Japanese web design with vibrant colors and dense information layout



<p><strong>Platform:</strong> Hugo<br><br>
<strong>Philosophy:</strong> Japanese web design with vibrant colours and dense information layout while remaining accessible and modern<br><br>
<strong>Repository:</strong> <a href="https://github.com/brennanbrown/brennan.jp.net" rel="noopener noreferrer">github.com/brennanbrown/brennan.jp.net</a><br><br>
<strong>Demo:</strong> <a href="https://brennan.jp.net" rel="noopener noreferrer">brennan.jp.net</a></p>

<p>Finally, I want to talk about the silly theme I decided to create. When I got started on <a href="https://blog.brennanbrown.ca/omg-lol-is-the-internet-we-need-right-now-3538199d5dea" rel="noopener noreferrer">omg.lol</a> I realized I wanted a fun, short domain. I decided to go looking for my first name with whatever TLD was available. And oh my god, there are <a href="https://porkbun.com/awesome" rel="noopener noreferrer">SO many ways</a> to end a website now! <code>.taxi</code>, <code>.pizza</code>, <code>.skins</code>, there are so many alternatives to the boring <code>.com</code> or <code>.org</code> we've come accustomed to. </p>

<p>I ended up settling for <code>brennan.day</code>, but I also bought <code>brennan.page</code> and <code>brennan.cafe</code>, since they were both cheap first-year purchases and short, and I'm currently using the other two for homeservers I've been neglecting. (It's so fun to procrastinate side projects with other side projects!)</p>

<p>When I was searching, though, one that caught my eye was <code>brennan.jp.net</code>, not only was this really inexpensive (less than $8 for registration <em>and</em> future yearly renewals) but it was also short and simple.</p>

<p>And then I had another thought, *Japanese web design sure is interesting, huh?</p>

<p>I wanted to recreate the compact, text-heavy, colourful aesthetic of traditional Japanese web design from the 1990s through 2010s, while maintaining modern accessibility and performance standards.</p>

<h3>
  
  
  A Brief History of Japanese Web Design
</h3>

<p>Many have already written about this topic with much more knowledge and research than I have, but digging into it, I found the phenomenon fascinating. As <a href="https://medium.com/@mirijam.missbichler/why-japanese-websites-look-so-different-2c7273e8be1e" rel="noopener noreferrer">one Reddit user succinctly put it</a>: "Japan, living in the year 2000 since 1985." The essence is that a lot of Japanese sites still retain a classic web 1.0 aesthetic that isn't <em>exactly</em> what the web 1.0 aesthetic looked like here in the West. While there is a simplicity, there is also a density, a wide array of different rows and columns displaying diverse information to the user all at once.</p>

<p>Before the iPhone changed mobile web design globally, Japan had<a href="https://web-japan.org/trends/11_culture/pop111124.html" rel="noopener noreferrer"><em>keitai culture</em></a>. As early as 1999, <a href="https://www.hongkiat.com/blog/japanese-web-design/" rel="noopener noreferrer">NTT DoCoMo launched i-mode</a>, bringing email and web browsing to compact mobile phones years before the rest of the world caught up. By 2000, Japanese phones had cameras, and by 2001, they had 3G. <a href="https://medium.com/@stevenmanangu360/why-japanese-websites-are-weirdly-designed-b2fdf0639f14" rel="noopener noreferrer">The J-SH04</a> was photo messaging before most people had even heard of a smartphone.</p>

<p>When the Western world started simplifying web design for the iPhone around 2007, <a href="https://digialps.com/why-japanese-websites-feel-stuck-in-the-90s/" rel="noopener noreferrer">Japanese designers didn't feel the same pressure</a> since they'd already optimized for mobile screens a decade earlier. Those text-heavy, densely packed layouts were <em>designed</em> to be viewed on tiny keitai screens. What looks overwhelming on a desktop was crafted for one-handed phone navigation on crowded Tokyo trains.</p>

<p>There's also the cultural expectation of <em>passivity</em> in information presentation, <a href="https://www.hongkiat.com/blog/japanese-web-design/" rel="noopener noreferrer">Japanese UX architects have noted</a> users expect information to be presented to them comprehensively, like a detailed brochure, rather than having to dig through minimalist menus to find what they need.</p>

<p><a href="https://www.ultimatewb.com/blog/5180/why-do-many-japanese-websites-maintain-a-design-aesthetic-that-appears-90s-or-early-2000s-to-western-eyes-while-western-websites-often-embrace-minimalist-design-trends/" rel="noopener noreferrer">Detail and thoroughness are valued</a> over the stark simplicity of minimalism. The bright, clashing colors within the neon-lit streets of Tokyo's shopping districts, where <a href="https://digialps.com/why-japanese-websites-feel-stuck-in-the-90s/" rel="noopener noreferrer">every available space is used efficiently</a>.</p>

<p><a href="https://sabrinas.space" rel="noopener noreferrer">Japanese language also has thousands of CJK characters</a>, which means far fewer web font options compared to Latin alphabets. This is why <a href="https://randomwire.com/why-japanese-web-design-is-so-different/" rel="noopener noreferrer">so many Japanese sites use text embedded in images</a>, giving designers typographic freedom which web fonts don't have.</p>

<p><a href="https://sabrinas.space" rel="noopener noreferrer">A fascinating quantitative study by Sabrina's Space</a> ran over 2,600 images of popular websites through clustering and found that Japanese sites distinctly avoid dark, minimalist designs, clustering instead around lighter colors and higher visual density. This pattern doesn't appear in other countries, even neighbouring nations with similar writing systems.</p>

<h3>
  
  
  ...Back to the Theme
</h3>

<p>There's a nostalgic charm with dense, newspaper-style layouts packed with information, bright and vibrant colors with thick borders, and classic web elements like webrings, 88x31 badges, and visitor counters (using local storage for fun, not actual analytics). Despite its retro appearance, it maintains modern accessibility with proper semantic HTML, keyboard navigation, high contrast, and screen reader compatibility.</p>

<p>The theme is designed to be accessible to non-technical users. You can customize colors through simple configuration in <code>hugo.toml</code>, change the primary and secondary color schemes with hex codes, toggle the sidebar on or off, and add custom CSS without touching the core theme files.</p>

<h2>
  
  
  Getting Started with Any Theme
</h2>

<p>All three themes are designed for accessible deployment. You can host them for free on <a href="https://netlify.com" rel="noopener noreferrer">Netlify</a>, <a href="https://pages.github.com" rel="noopener noreferrer">GitHub Pages</a>, <a href="https://vercel.com" rel="noopener noreferrer">Vercel</a>, or <a href="https://pages.cloudflare.com" rel="noopener noreferrer">Cloudflare Pages</a>. The only cost is a domain name (which can be as cheap as $5/year on <a href="https://porkbun.com" rel="noopener noreferrer">Porkbun</a>).</p>

<p>Like all my work, they're released under the AGPL* license, meaning you can use them however you like, modify them, and even use them for commercial projects.</p>

<p><em>Note:</em> The themes also include proper SEO configuration with <code>noindex</code> options for demo sites (remember to turn this off when you launch!), social media meta tags, and optimized feeds for discoverability.</p>

<h3>
  
  
  Why I Made Them
</h3>

<p>I'd like to think these themes represent three different approaches to personal publishing:</p>

<ul>
<li>
<strong>Indiepaper</strong> for those who value minimalism, speed, and accessibility above all else</li>
<li>
<strong>Newsprint</strong> for those who want to run a serious publication with professional aesthetics</li>
<li>
<strong>brennan.jp.net</strong> for those who miss the creative, playful web of the past</li>
</ul>

<p>I care about content ownership, web standards, and the IndieWeb principles of controlling your own digital identity. These themes are fast, accessible, and designed to last. No framework churn, no dependencies, just HTML, CSS, and markdown files.</p>

<p>Whether you're a writer, journalist, developer, or just someone who wants a corner of the internet to call your own, these themes offer a foundation for building something meaningful. Just you, your words, and your website.</p>

<p>Happy blogging!</p>

