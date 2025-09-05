---
Title: I built the ultimate Pokédex and would love you to try it!
Description: 
Author: Juanda Martínez
Date: 2025-09-05T21:10:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>Four years ago, while playing Pokémon, I realized that I could memorize a lot of stuff like my SSN or my EIN... but not the <strong>Pokémon type chart</strong>. Every match, I had to pause, open a wiki, search weaknesses, check evolution chains, among all crappy useless information that only made me <strong>waste more time scrolling down</strong> to the exact info I wanted.</p>

<p>So as a web dev, I decided that I could build my own tools for my personal use, so why not create a web app that solve this issue for me? having this kind of information I cared about in a single place, without wasting too much effort on <code>Googling a Pokémon → Open a wiki link → Scroll up/down many times → Find the info I wanted → Click more links bc I needed more info → Repeat</code></p>

<p>And that's how <strong>Pokémon Stats</strong> was born: a small MVP using <strong>PokéAPI</strong> + <strong>Bootstrap</strong>, quickly fetching Pokémon's evolution chains, evolution details, a type chart, and a basic search bar to enter the Pokémon name. Nothing fancy if you ask me, and made for <em>casual players</em> (just like me).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo44vp1ghqzu5wmok88am.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo44vp1ghqzu5wmok88am.webp" alt="First version of Pokémon Stats" width="800" height="669"></a></p>

<p>So I bought a domain and hosted the site directly with <em>Vercel</em>, because I wanted to feel this like a sophisticated web app, and from that I just've been pushing small updates with stuff that I wanted to improve in the user experience while playing Pokémon games.</p>

<p><strong>Fast-forward to today</strong>: I’ve completely redesigned the project from the ground up. Same idea, but <em>cleaner</em>, <em>faster</em>, and built with the stack I actually enjoy using: <strong>Next.js, React, Tailwind, shadcn/ui)</strong>.</p>

<h2>
  
  
  What changed?
</h2>

<ul>
<li>A redesigned <strong>type effectiveness chart</strong> (interactive, mobile-friendly).</li>
<li>A <strong>search system with suggestions</strong> and <strong>keyboard navigation</strong>.</li>
<li>
<strong>Improved evolution chains</strong>, including special cases (stones, time of day, trades, etc.).</li>
<li>Support for <strong>regional forms and variants</strong> (Alola, Galar, etc.).</li>
<li>High-quality official artwork with fallback sprites.</li>
<li>A roadmap and contributors page, because now it’s <strong>open source</strong>!</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4z2ju9ru64zu89ib1wmm.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4z2ju9ru64zu89ib1wmm.webp" alt="Pokémon Stats redesigned" width="800" height="843"></a></p>

<h2>
  
  
  Why it matters to me
</h2>

<p>This project isn’t the most complex I’ve ever built, but it’s the one I’ve loved the most. It started as a personal tool, grew through trial and error, and turned into something I still actively use while playing. I’ve put a lot of effort (and coffee) into making it not just functional but actually enjoyable to look at and use.</p>

<p>It’s free, it’s open, and if you’re a Pokémon fan (or just like checking out passion projects), I’d love for you to try it: <a href="https://pokemonstats.com" rel="noopener noreferrer">pokemonstats.com</a></p>

<p>You can also peek at the source, file an issue, or even contribute: <a href="https://github.com/juandadev/pokemonstats" rel="noopener noreferrer">GitHub - juandadev/pokemonstats</a></p>

