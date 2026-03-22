---
Title: I Built a Free Open Source Tool That Saved Me From Deleting My Production Database (Again)
Description: 
Author: Subham 
Date: 2026-03-22T21:31:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Okay so real talk. I've done it twice.</p>

<p>The first time was during college. I was trying to drop a test table, typed the wrong database name, and wiped out actual user data from a project I was working on with friends. The second time was worse. I was 11pm deep into shipping a feature, ran a migration script, and it quietly nuked 3 months of production records.</p>

<p>Both times I just sat there staring at my screen like 👁️👄👁️</p>

<p>And both times, I had no backup.</p>

<p>So I built <strong><a href="https://github.com/Subham-Maity/chronobase" rel="noopener noreferrer">ChronoBase</a></strong>  a free, open source, local-first database backup manager for PostgreSQL and MongoDB. It runs on your laptop, has a proper web UI, schedules automatic backups, and lets you restore to any database with one click.</p>

<p>No monthly fees. No cloud account. No random company holding your data hostage.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx4rjl2zuzgeb8bz9r3j0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx4rjl2zuzgeb8bz9r3j0.png" alt="chronobase - About&lt;br&gt;
Local-first database backup manager for PostgreSQL &amp; MongoDB. Schedule automatic snapshots, restore to any target, and wipe databases all from a sleek web UI running on your own machine. No cloud, no subscriptions, no BS."></a></p>


<h2>
  
  
  Why does every backup tool either cost money or suck?
</h2>

<p>Genuinely asked myself this for way too long.</p>

<p>pgAdmin has manual backup buried in menus. MongoDB Compass doesn't really do scheduled backups. Navicat is like $250. Every cloud backup tool wants your credit card just to get started.</p>

<p>For solo devs and indie hackers running multiple side projects, all you really need is:</p>

<ol>
<li>A place to store all your database connection strings</li>
<li>Something that dumps them automatically on a schedule</li>
<li>The ability to restore any snapshot to any target</li>
</ol>

<p>That's it. That's the whole thing. Why does this not exist as a simple, free, open source tool?</p>

<p>So I built it.</p>


<h2>
  
  
  What ChronoBase actually does
</h2>

<p>Here's the quick rundown:</p>

<p><strong>Connections</strong> — you paste in your PostgreSQL or MongoDB connection URLs, give them a name, group them by project (like <code>Production</code>, <code>Staging</code>, <code>MySaaS</code>). The passwords are masked in the UI so you're not staring at credentials all day.</p>

<p><strong>One-click backup</strong> — hit the button, it runs <code>pg_dump</code> or <code>mongodump</code> under the hood, saves the <code>.sql</code> file or MongoDB BSON folder to your local disk. It takes like 2 seconds for small DBs.</p>

<p><strong>Scheduled backups</strong> — set a cron expression (<code>0 2 * * *</code> = daily at 2am) and forget about it. The server runs in the background and fires backups while you sleep. You wake up and they're just... there.</p>

<p><strong>Restore</strong> — pick any snapshot, choose "same database" (overwrites with the old snapshot) or "push to different DB" (clone to staging, new server, whatever). One click.</p>

<p><strong>Wipe DB</strong> — this one's spicy. Sometimes you want to completely nuke the contents of a database before restoring a backup. The Wipe DB button drops every table, every schema (<code>DROP SCHEMA public CASCADE</code> for Postgres), then you restore clean. No leftover junk data.</p>

<p><strong>Danger Zone</strong> — full reset with typed confirmation. You literally have to type <code>DELETE EVERYTHING</code> to confirm. Because I've seen what happens when buttons don't have confirmation dialogs.</p>


<h2>
  
  
  The tech stack (and why I kept it stupid simple)
</h2>

<p>Okay so I went with <strong>Node.js + Express</strong> on the backend and <strong>vanilla ES modules</strong> on the frontend. No React, no bundler, no build step.</p>

<p>Why? Because I wanted anyone to be able to clone this, read the code, and immediately understand what's happening. No <code>npm run build</code>, no compiled bundle, no mystery.</p>

<p>Data is stored in a plain <strong>JSON file</strong> (<code>chronobase-data.json</code>). No SQLite, no native modules that break on Node v24 (learned that one the hard way mid-build). If you want to move to a new machine, you literally just copy the folder.</p>

<p>The project structure is modular — one file per concern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>src/
  database.js      ← JSON read/write
  tools.js         ← finds pg_dump/mongodump on your system
  backup.js        ← runs the actual dumps and restores
  scheduler.js     ← manages cron jobs
  routes/          ← one route file per resource
</code></pre>

</div>



<p>Frontend is the same — <code>public/js/pages/</code> has one file per page. Clean, readable, zero magic.</p>




<h2>
  
  
  The actual hard part: detecting CLI tools across Windows, macOS, and Linux
</h2>

<p>You know what's annoying? <code>pg_dump</code> lives in completely different places depending on OS, PostgreSQL version, and whether you installed it through an installer, Homebrew, or apt.</p>

<p>The <code>tools.js</code> module checks a list of known paths:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">PG_DUMP_CANDIDATES</span> <span class="o">=</span> <span class="p">[</span>
  <span class="dl">'</span><span class="s1">pg_dump</span><span class="dl">'</span><span class="p">,</span>
  <span class="dl">'</span><span class="s1">/opt/homebrew/bin/pg_dump</span><span class="dl">'</span><span class="p">,</span>
  <span class="dl">'</span><span class="s1">C:</span><span class="se">\\</span><span class="s1">Program Files</span><span class="se">\\</span><span class="s1">PostgreSQL</span><span class="se">\\</span><span class="s1">18</span><span class="se">\\</span><span class="s1">bin</span><span class="se">\\</span><span class="s1">pg_dump.exe</span><span class="dl">'</span><span class="p">,</span>
  <span class="dl">'</span><span class="s1">C:</span><span class="se">\\</span><span class="s1">Program Files</span><span class="se">\\</span><span class="s1">PostgreSQL</span><span class="se">\\</span><span class="s1">17</span><span class="se">\\</span><span class="s1">bin</span><span class="se">\\</span><span class="s1">pg_dump.exe</span><span class="dl">'</span><span class="p">,</span>
  <span class="c1">// ... and more</span>
<span class="p">];</span>
</code></pre>

</div>



<p>It probes each one by running <code>"&lt;path&gt;" --version</code> with a 5 second timeout. First one that responds gets used. Works on every platform without configuration.</p>

<p>The UI shows green/red dots in the sidebar telling you instantly which tools are detected. If something's missing, the Setup page gives you exact install commands for Windows, macOS, and Linux.</p>




<h2>
  
  
  How to run it right now (for real, it takes 3 minutes)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/Subham-Maity/chronobase.git
<span class="nb">cd </span>chronobase
npm <span class="nb">install
</span>node server.js
</code></pre>

</div>



<p>Open <code>http://localhost:3420</code>.</p>

<p>That's it. It's running. Add your first connection string and click Backup.</p>

<p>For Windows you can also double-click <code>START.bat</code> which auto-installs dependencies, starts the server, and opens the browser for you.</p>




<h2>
  
  
  Why local-first matters more than you think
</h2>

<p>Every time you send your database connection string to a cloud backup service, that's another place your credentials live that isn't yours.</p>

<p>ChronoBase keeps everything local. Your <code>.sql</code> files are on your disk. Your connection URLs are in a JSON file on your machine. Nothing is uploaded, transmitted, or stored anywhere external.</p>

<p>For solo devs this is usually fine as-is. For anything with real user data, you obviously want encryption at rest — that's on the roadmap. But the local-first baseline is the right default.</p>




<h2>
  
  
  What I want to add next
</h2>

<p>Current roadmap (also in the README if you want to contribute):</p>

<ul>
<li>
<strong>S3 / R2 / Backblaze upload</strong> — offsite backup support</li>
<li>
<strong>Backup compression</strong> — gzip .sql files, they can get large</li>
<li>
<strong>Slack/Discord notifications</strong> — ping when a scheduled backup fails</li>
<li>
<strong>MySQL / MariaDB</strong> — same engine, just different CLI tools</li>
<li>
<strong>Docker image</strong> — <code>docker run -p 3420:3420 chronobase</code>
</li>
<li>
<strong>Encryption</strong> — AES-256 for stored connection URLs</li>
</ul>

<p>If any of these sound useful to you, PRs are open. The codebase is intentionally small and readable — a decent Node dev can add a new route in like 20 minutes.</p>




<h2>
  
  
  The real reason I'm sharing this
</h2>

<p>I know there are thousands of devs running side projects and SaaS apps on $5 droplets who don't have a proper backup strategy. It's not because they don't care. It's because setting one up is annoying and all the existing tools are either overkill or paywalled.</p>

<p>This is meant to be the thing that just works. Run it on your laptop or your home server, add your connection strings, set a schedule, and stop thinking about it.</p>

<p>Because the backup you don't have is the one you'll wish you did at 1am when things go wrong.</p>




<p><strong>GitHub:</strong> <a href="https://github.com/Subham-Maity/chronobase" rel="noopener noreferrer">github.com/Subham-Maity/chronobase</a></p>

<p>If it saves you from a disaster, drop a ⭐ — it genuinely helps the project get discovered.</p>

<p>And if you've had your own database horror story, tell me in the comments. Misery loves company.</p>

