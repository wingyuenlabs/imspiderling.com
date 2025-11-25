---
Title: Deploying My First Task Automation App — Lessons Learned and Tips for Beginners
Description: 
Author: Dillion Huston
Date: 2025-11-25T21:36:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey everyone! <br>
I just shipped my first real project to production: <strong><a href="https://dillonhtask.netlify.app/" rel="noopener noreferrer">https://dillonhtask.netlify.app/</a></strong></p>

<p>It’s a small but practical <strong>task automation app</strong>. Right now the live version focuses on <strong>sending email reminders</strong>, while the local version (on GitHub) includes an extra feature that automatically cleans up old files. More features are in the pipeline — this is just v0.1!</p>

<p>The main goal wasn’t to build the next Todoist killer. It was to finally go through the entire process of taking something from “works on my machine” → actually live on the internet, and document all the face-palm moments along the way.</p>

<p>Spoiler: there were many.</p>

<h2>
  
  
  What the App Does Right Now
</h2>

<ul>
<li>Online: Send yourself (or others) email reminders at a scheduled time
</li>
<li>Local-only (for now): Auto-delete old downloads/temp files
</li>
<li>Planned: More automations, proper backend, cron jobs, webhooks, notifications, etc.</li>
</ul>

<p>Even in its current tiny form, I already use the reminder feature daily.</p>

<h2>
  
  
  The Painful (But Invaluable) Problems I Hit
</h2>

<h3>
  
  
  1. “It Works on My Machine” Syndrome
</h3>

<p>Everything was perfect locally. Deployed to Netlify → half the CSS and all images 404’d.</p>

<p><strong>Root cause:</strong> I was using relative paths like <code>../assets/logo.png</code>. They work when you open <code>index.html</code> directly, but break when served from a sub-route.</p>

<p><strong>Fix:</strong> Switch to root-relative paths → <code>/assets/logo.png</code> (note the leading slash).</p>

<p><strong>Lesson:</strong> Production doesn’t care about your local file system. Always test the production build locally (<code>npm run build &amp;&amp; serve -s build</code>).</p>

<h3>
  
  
  2. The Dreaded SPA Refresh 404
</h3>

<p>Clicking a React Router link? Fine.<br><br>
Direct visit or page refresh on <code>/settings</code>? 404 city.</p>

<p><strong>Why:</strong> Netlify (like most static hosts) tries to find a real file at that path. There isn’t one — it’s client-side routing.</p>

<p><strong>Fix:</strong> Add a <code>_redirects</code> file inside the <code>public/</code> folder (or at the build output root):</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
text
/*    /index.html   200
That single line tells Netlify: “For every route, serve index.html but keep the 200 status” → React Router takes over.
3. Pointing Netlify at the Wrong Folder
My project looked like this at one point:
textproject/
├── src/
├── public/
├── build/
├── random-old-crap/
└── who-knows-what/
Netlify kept deploying an empty site because the publish directory was wrong or the build folder name changed between frameworks.
Fix: Cleaned everything up and explicitly set:

Build command: npm run build
Publish directory: dist (Vite) or build (Create React App)

4. Thinking Ahead: Environment Variables
The app doesn’t have a backend yet, but I already added .env variables on Netlify for future email API keys, etc.
Why bother now? Because adding them later means redeploying and potentially exposing secrets if you forget.
My Exact Deployment Checklist (Works Every Time Now)

npm run build locally → verify it works with serve -s build
Push to GitHub
Netlify → “New site from Git”
Connect repo
Settings:
Build command: npm run build
Publish directory: dist (or build)

Add _redirects file if it’s an SPA
Deploy → profit

Total time once everything is clean: ~60 seconds.
Biggest Takeaways (Especially for Beginners)

Deployment is a skill on its own — treat it as such
Local ≠ Production. Always test the actual build output
Use root-relative paths for assets (/assets/...)
For SPAs on Netlify/Vercel: you need a rewrite/redirect rule
Clean project structure saves hours of debugging
Set up environment variables early
Automate nothing until you’ve deployed something

What’s Next?

Bring the file-cleanup feature online (needs a tiny backend)
FastAPI + JWT backend
Scheduled tasks (Cron-like)
GitHub Actions integration
More automations (RSS → email, price tracking, etc.)

This was a “small” project, but it taught me more about real-world development than any tutorial ever did.
If you’ve deployed something recently and hit weird bugs (especially the infamous refresh 404), drop your war stories in the comments — I’d love to hear how you solved them! 🚀
P.S. Here’s the live app again: https://dillonhtask.netlify.app/
#webdev #beginners #deployment #react #netlify
</code></pre>

</div>

