---
Title: I Built a Collaborative Book Platform Where the Community Decides What Gets Published - No Admins, No Gatekeepers
Description: 
Author: Art
Date: 2026-03-21T21:48:09.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxmjb2398yoem3zwd7wg8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxmjb2398yoem3zwd7wg8.png" alt=" " width="800" height="675"></a></p>

<p><em>Written in Go. Runs on SQLite. Authors own their files. Forever.</em></p>




<p>Forty-two years ago, I started with an Amiga 500 and no manual.</p>

<p>Last week, I shipped ForgeCrowdBook — a platform where books live on Codeberg, GitHub, or IPFS, and the community pins what gets featured. No database server. No password hell. No platform lock-in.</p>

<p>Here's the idea — and here's where I need your help.</p>




<h2>
  
  
  The Problem with Publishing Platforms
</h2>

<p>Every writing platform eventually becomes a gatekeeper.</p>

<p>Either an algorithm decides what gets seen, or an admin team moderates what exists, or a VC-backed company decides the rules change next quarter. Authors pour their work into systems they don't own.</p>

<p>Take WordPress. It powers 43% of the web and it's genuinely impressive — but running it means: a PHP server, a MySQL database, a caching layer (because otherwise it's slow), a plugin for security, a plugin for backups, a plugin for SEO, and a hosting bill that grows with traffic. Your content lives in a database you probably don't back up often enough. When the host goes down, your book goes down.</p>

<p>Or you go with Medium, Substack, or Ghost — until the terms change, the algorithm buries you, or the company pivots. Your audience was never really yours.</p>

<p>I've watched it happen. I've experienced it. I wanted something different.</p>




<h2>
  
  
  How ForgeCrowdBook Works
</h2>

<p>You write first. That's it. No repo setup, no IPFS pinning, no technical hurdles before you've written a single chapter.</p>

<p>Here's the actual flow:</p>

<ol>
<li>
<strong>Sign up via Magic Link</strong> — no password, no form hell.</li>
<li>
<strong>Write your book</strong> directly on the platform. Chapters in Markdown.</li>
<li>
<strong>Your book appears in "New Releases"</strong> — visible, readable, pinnable.</li>
<li>
<strong>The community pins it.</strong> Once it hits the threshold (<code>min_pins</code>, default: 3), you get an email: <em>"Your book made it — here's your ZIP, here's a guide to set up a repo."</em>
</li>
<li>
<strong>You create a repo on Codeberg, GitHub, or IPFS</strong> and register the URL.</li>
<li>
<strong>Your book goes live on the main page.</strong> From that point on, we only store the link — never the file.</li>
</ol>

<p>The technical bar comes <em>after</em> the creative win. You only need a repo because your book earned it.</p>

<p>Two flavours of books:</p>

<ul>
<li>
<strong>Closed books</strong> — only the author adds chapters. The community pins the book.</li>
<li>
<strong>Open books</strong> — any registered user can add chapters. Once a chapter reaches <code>min_pins</code> likes, the book gets featured.</li>
</ul>

<p>No admin. No algorithm. The community decides.</p>




<h2>
  
  
  The Tech Stack (Intentionally Boring)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Language:  Go 1.23+
Database:  SQLite (embedded via modernc.org/sqlite)
Auth:      Magic Links only — no passwords
Config:    SML (our own simple markup language)
</code></pre>

</div>



<p>Why Go + SQLite? Because boring infrastructure means I can think about the actual problem.</p>

<p>Single binary. Single file database. Runs on a €5 VPS. No Docker required unless you want it.</p>

<p>Auth is Magic Link only — no password resets, no "forgot password" flows, no bcrypt debates. Just click the link in your email. Done.</p>




<h2>
  
  
  Performance: Everything is Static
</h2>

<p>Here's what makes ForgeCrowdBook genuinely fast: <strong>there is no dynamic page rendering</strong>.</p>

<p>The reading experience is pure static output. Markdown files served from Git forges or IPFS — no PHP, no template engine crunching on every request, no database query per page load. A reader opening a book chapter gets a file. That's it.</p>

<p>Compare that to a typical WordPress setup: a request hits PHP, PHP queries MySQL, MySQL returns rows, PHP renders HTML, a caching plugin maybe saves it for next time. Four moving parts for one page view. Under load, that stack buckles.</p>

<p>ForgeCrowdBook's architecture under load: serve a file. That's it. Static hosting scales to zero cost and near-infinite traffic. IPFS makes it even more resilient — the more people read, the more the content propagates.</p>

<p>The Go backend handles auth, pins, and discovery. Everything else is static. That's not a limitation — it's the design.</p>

<p>Here's what a book page template looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>{{ define "content" }}
  <span class="nt">&lt;h1&gt;</span>{{ .Book.Title }}<span class="nt">&lt;/h1&gt;</span>
  {{ if .Book.AuthorName }}
    <span class="nt">&lt;p&gt;</span>{{ t "label_by" }} {{ .Book.AuthorName }}<span class="nt">&lt;/p&gt;</span>
  {{ end }}
  {{ if .Book.Prologue }}
    {{ markdown .Book.Prologue }}
  {{ else }}
    <span class="nt">&lt;p&gt;</span>{{ .Book.Description }}<span class="nt">&lt;/p&gt;</span>
  {{ end }}
{{ end }}
</code></pre>

</div>



<p>That's it. No code generator. No build pipeline. No hydration step. Go renders this template once — you get HTML. Fast, cacheable, deployable anywhere. The <code>markdown</code> helper converts Markdown to HTML at render time. The <code>t</code> helper handles i18n. Everything else is just data from SQLite.</p>

<p>This is what performance looks like when you don't add complexity you don't need.</p>

<h2>
  
  
  What's Alive Right Now
</h2>

<p>The platform is running. Two books are already on it:</p>

<p><strong>"The Third Attempt"</strong> — a closed book. My own story: 42 years, 3 attempts, an Amiga 500, a Forge 4D framework, and an AI team. For junior developers and everyone still waiting for Godot. Free to read. Free to share.</p>

<p><strong>"2026 – The Rewrite Begins"</strong> — an open book. A community rewrite of Orwell's 1984, set in 2026. Because some dystopias need new endings, written together.</p>

<p>Both are pinnable. Both are community-owned. Neither requires an account to read.</p>




<h2>
  
  
  What I'm Looking For
</h2>

<p>This is where you come in.</p>

<p>ForgeCrowdBook is real and running, but it's early. I'm an author and a developer, not a team. I want to build this <em>with</em> the community, not just <em>for</em> it.</p>

<h3>
  
  
  I want your feature requests.
</h3>

<p>What's missing? What would make you actually use this as a reader, an author, or a contributor?</p>

<p>Some things I'm already thinking about:</p>

<ul>
<li>Book covers ofc</li>
<li>Nice theme(s)</li>
<li>Convert to EPUP (<a href="https://kdp.amazon.com" rel="noopener noreferrer">kindle direct publishing</a>)</li>
<li>Convert to PDF (print on demand like <a href="https://www.epubli.com/" rel="noopener noreferrer">Epubli</a>)</li>
<li>RSS feeds per book and per author</li>
<li>IPFS pinning support (add your chapter directly from IPFS)</li>
<li>Better discovery (tags, genres, languages)</li>
<li>CLI tool for authors to push chapters without touching the web UI</li>
<li>Federation — imagine following a book across instances</li>
</ul>

<p>But I might be thinking about the wrong things entirely. Tell me what you'd build first.</p>

<h3>
  
  
  I want beta authors.
</h3>

<p>If you have something to write — a technical series, a fiction project, an open manifesto — I want to host it. Reach out or drop a comment. Magic Link signup is live.</p>

<h3>
  
  
  I want Go eyes on the code.
</h3>

<p>The codebase is ~2,000 lines of clean Go. If you enjoy reading well-structured Go and have thoughts on architecture, routing patterns, or test coverage — I'd love a code review. Open to brutal honesty.</p>




<h2>
  
  
  Why There's No Spam Filter
</h2>

<p>There isn't one. And it's not an oversight.</p>

<p>Most platforms fight spam with CAPTCHAs, moderation queues, IP blocklists, or ML classifiers. All of that is expensive to build, expensive to maintain, and never quite works.</p>

<p>ForgeCrowdBook uses <strong>natural friction at the right moment</strong> instead.</p>

<p><strong>Layer 1 — Writing</strong>: Anyone can create a book. Zero barrier. Spammers can post — but it only lands in "New Releases". No damage done.</p>

<p><strong>Layer 2 — Community pins</strong>: Spam doesn't get 3 pins. Real books do. The community <em>is</em> the filter — no algorithm needed.</p>

<p><strong>Layer 3 — The repo</strong>: Only authors who receive the "your book made it" email have a reason to set up a repo. That's real work. Spammers don't bother — there's no reward for them at this stage.</p>

<p>The result: IPFS stays clean. The main page stays clean. No moderator ever had to make a decision.</p>

<p>Anti-spam through system design, not through policing. That's the only kind that scales.</p>




<h2>
  
  
  The Philosophy Behind It
</h2>

<p>No passwords. No admin moderation. No content lock-in. No third parties. No database server.</p>

<p>Every one of those "nos" is a deliberate choice. Complexity is a liability. Every dependency is a future migration. Every admin is a future bottleneck.</p>

<p>The license is <strong>libera</strong> — a libre license in Esperanto. Because the ideas here belong to everyone.</p>




<h2>
  
  
  Try It / Follow It / Build It
</h2>

<ul>
<li>
<strong>Platform</strong>: <a href="https://codeberg.org/crowdware/forgecrowdbook" rel="noopener noreferrer">forgecrowdbook on Codeberg</a>
</li>
<li>
<strong>Read "The Third Attempt"</strong>: running live at localhost... soon public</li>
<li>
<strong>Feature requests</strong>: Drop them in the comments here or open an issue on Codeberg</li>
</ul>

<p>If this resonates — follow me here on dev.to. I'll be writing about the architecture decisions, the SML config language, the IPFS integration road ahead, and what it's like building publishing infrastructure as a solo dev in 2026.</p>




<p><em>Built by Art (Adam Art Ananda) — developer, author, Rainbow Warrior. Based in Wittenberg, Germany..</em></p>

