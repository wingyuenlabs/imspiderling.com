---
Title: SKALDA – privacy-first browser tools that run entirely in your browser
Description: 
Author: SKALDA
Date: 2025-11-26T21:08:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’ve started building <strong>SKALDA</strong>, a multi-brand suite of privacy-first browser tools.</p>

<p>Main site: <a href="https://skalda.io" rel="noopener noreferrer">https://skalda.io</a></p>

<p>Most online tools for “small” tasks — converting units, files, images, video, audio, working with PDFs, subtitles, data, and more — force you to upload files, create an account, and accept tracking. SKALDA does the opposite:</p>

<ul>
<li>tools run locally in the browser
</li>
<li>no logins or user accounts
</li>
<li>no file uploads to my servers
</li>
<li>no tracking scripts
</li>
<li>everything localized into 62+ languages, including RTL</li>
</ul>




<p>What exists today</p>

<p><strong>UNITS</strong> – <a href="https://units.skalda.io" rel="noopener noreferrer">https://units.skalda.io</a><br><br>
Scientific unit converter with:</p>

<ul>
<li>highly customizable Smart Tab on the homepage
</li>
<li>500+ units across many categories
</li>
<li>smart calculators and custom units
</li>
<li>“Units Museum” timeline + a small interactive game
</li>
<li>bulk converter that can handle millions of values in one input (transpose, export, etc.)</li>
</ul>

<p><strong>FLINT</strong> – <a href="https://flint.skalda.io" rel="noopener noreferrer">https://flint.skalda.io</a><br><br>
File tools with:</p>

<ul>
<li>file converter for 100+ formats across images, documents, audio, video, archives, etc.
</li>
<li>PDF organizer (merge / split / reorder / rotate / crop / watermark)
</li>
<li>metadata tools (inspect and clean EXIF/GPS/author and other fields)
</li>
<li>subtitle tools (SRT/VTT, timing fixes, batch workflows)</li>
</ul>

<p>Across SKALDA, settings and state are shared and stored locally: theme, language, ads toggle, and other preferences are synced via <code>localStorage</code> across all brands, with everything staying on the user’s device.</p>




<h2>
  
  
  Tech side
</h2>

<ul>
<li>React + TypeScript + Vite
</li>
<li>Cloudflare Pages for hosting
</li>
<li>Mix of open-source client-side / WASM libraries plus custom architecture and glue code
</li>
<li>Multi-brand setup (UNITS, FLINT, etc.) built from one codebase with brand-specific entry points, layouts, and translations
</li>
<li>i18n for 62+ languages, including RTL handling</li>
</ul>

<p>I launched UNITS about a month ago and FLINT a few days ago. With basically no marketing (just Product Hunt and AlternativeTo), SKALDA has done around 17k unique visitors in the first month across the domain, mostly organic.</p>




<h2>
  
  
  What’s next
</h2>

<p>I’m expanding SKALDA with more brands:</p>

<ul>
<li>
<strong>PIXEL</strong> – image tools
</li>
<li>
<strong>SCRIBE</strong> – writing / docs tools
</li>
<li>
<strong>DEV</strong> – developer utilities
</li>
<li>
<strong>SOLVEO</strong> or <strong>CLIP</strong> – calculators / solvers vs audio / video tools (still deciding the order)
</li>
<li>
<strong>SCOUT</strong> – search / analysis helpers
</li>
</ul>

<p>Right now I’m building out <strong>PIXEL</strong> as the next major brand, while improving UX and performance in UNITS and FLINT.</p>

