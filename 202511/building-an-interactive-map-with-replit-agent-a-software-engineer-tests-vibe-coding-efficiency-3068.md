---
Title: Building an interactive map with Replit Agent: A software engineer tests vibe-coding efficiency
Description: 
Author: Makena Kong
Date: 2025-11-17T21:14:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Exploring funded brick-and-mortar companies on Wefunder isn’t easy, so <strong>I asked Replit Agent to build a map</strong> and see how much time AI could save me versus doing it manually.</p>

<p>In about <strong>3 hours</strong> and for roughly <strong>$22</strong>, it produced <strong>a fully searchable map of funded companies</strong>. Most of the cost came from the Agent inefficiently debugging an API integration and unnecessary reloads. If I had built it myself, it probably would’ve taken <strong>4–5 hours</strong> to design, build, test, and deploy.</p>

<p>Check out the code or explore the map below:</p>

<p><strong>Repo:</strong> 🔗 <a href="https://replit.com/@kenakingkong/Wefunder-Brick-and-Mortar-Map?v=1#design_guidelines.md" rel="noopener noreferrer">Wefunder Brick &amp; Mortar App by kenakingkong on Replit</a></p>

<p><strong>Demo:</strong> 🔗 <a href="https://wefunder-brick-and-mortar-map.replit.app/" rel="noopener noreferrer">wefunder-brick-and-mortar-map.replit.app</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyyco9v8awca5y8g62tpt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyyco9v8awca5y8g62tpt.png" alt=" " width="800" height="452"></a></p>

<p><strong>Keep reading</strong> for a full breakdown of what worked, what didn’t, and where AI saved time versus where it didn’t.</p>

<h2>
  
  
  Background
</h2>

<p>Wefunder has thousands of funded companies, but the only way to browse them is by scrolling through media-heavy cards on the Explore page. (eg: <a href="https://wefunder.com/explore/brick_and_mortar" rel="noopener noreferrer">wefunder.com/explore/brick_and_mortar</a>).</p>

<p>There <em>is</em> a map feature, but it only shows companies that are <em>currently raising</em>. I needed a better way to explore funded brick-and-mortar companies.</p>

<p>Replit launched Replit Agent in 2024, an AI agent for automating software development and building web apps.</p>

<p>I’m a full-stack engineer with 5+ years of experience (<a href="https://www.makenakong.com/dev" rel="noopener noreferrer">makenakong.com/dev</a>), so normally I’d just build this myself. But with all the hype around vibe-coding, this felt like a low-stakes project to test it out.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkybhvpbu6ozrpit9668.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkybhvpbu6ozrpit9668.png" alt=" " width="800" height="600"></a></p>

<h2>
  
  
  Building the app
</h2>

<h3>
  
  
  Getting started
</h3>

<p>My first prompt was:</p>

<blockquote>
<p>“Build me an interactive map of all funded brick-and-mortar companies on Wefunder.”</p>
</blockquote>

<p>The Agent started looking for a public Wefunder API, which exists but is not publicly accessible. I should’ve told it upfront that I had a private API endpoint ready.</p>

<p>After a few minutes, I updated the plan with the correct endpoint and example JSON. Another minute later, I approved the revised plan and hit <strong>Build the entire app.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F46h34of3bnr4tggd4w8r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F46h34of3bnr4tggd4w8r.png" alt=" " width="800" height="636"></a></p>

<p>Eventually, the app loaded, but without any companies. And that’s when things got hands-on.</p>

<h3>
  
  
  “Development”
</h3>

<p><strong>Setup and Initial Structure</strong></p>

<p>The agent installed the packages and generated core components, including <a href="https://leafletjs.com/" rel="noopener noreferrer">Leaflet</a>, an open-source JavaScript library for interactive maps. I’m unfamiliar with the package so it would’ve taken me a while to read the documentation and render it properly.</p>

<p><strong>API Endpoint Issues</strong></p>

<p>First, I tried using the Explore page endpoint, but hit a CORS wall.</p>

<p>Then I generated an external API key and tried using Wefunder’s private external API (the one KingsCrowd uses).</p>

<p>The JSON structure was totally different.<br>
It wasn’t filterable by category.<br>
This confused the AI and resulted in non–brick-and-mortar companies showing up.</p>

<p>After watching the agent repeatedly debug and regenerate with no progress, I finally parsed the endpoint explore page data myself and hardcoded it.</p>

<p><em>This would’ve gone much smoother if Wefunder had a clean, public API. And, this would’ve been faster if I had parsed the JSON response and debugged it myself as well.</em></p>

<p><strong>Fine Tuning</strong></p>

<p>Once the data displayed correctly, I fine-tuned the UI/UX through small prompts, like:</p>

<ul>
<li>Remove all icons and make the total raised and investors inline</li>
<li>Add a “funded ” to the total raised and investors section</li>
<li>Remove search by category functionality from the sidebar</li>
<li>I asked it to fix the position of the search icon in the search box and it took 8 minutes and cost $1.38.</li>
</ul>

<p>Each request took between 1–8 minutes and cost $0.30-$2.00.</p>

<p><em>Realistically, I could’ve done most of these edits manually faster and for free.</em></p>

<h3>
  
  
  Hosting
</h3>

<p>The agent published and hosted the site directly. It failed once, then succeeded.</p>

<h3>
  
  
  Other issues
</h3>

<ul>
<li>Replit Agent sometimes removed only parts of a feature instead of all of it, leading to compilation errors.</li>
<li>It often reloaded the entire app after tiny changes, instead of doing partial reloads, burning extra credits.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fba69ibaux7anzia5bz0y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fba69ibaux7anzia5bz0y.png" alt=" " width="800" height="452"></a><br>
<em>The search functionality still needs work but I’m running low on credits.</em></p>

<h2>
  
  
  Comparison
</h2>

<h3>
  
  
  What I Liked
</h3>

<ul>
<li>It saved time on setup and boilerplate.</li>
<li>The default UI was perfectly fine for a simple app.</li>
</ul>

<h3>
  
  
  What I Didn’t Like
</h3>

<ul>
<li>Watching it reload endlessly.</li>
<li>Watching it chase the wrong debugging paths.</li>
<li>Watching my credits evaporate when I could’ve fixed things faster myself.</li>
<li>Not knowing what the code looked like at every moment.</li>
</ul>

<h2>
  
  
  Replit AI vs Me: Who Should Handle What?
</h2>

<p><strong>Installation &amp; Setup:</strong> Agent, it was fast and runs in background</p>

<p><strong>Overall design &amp; layout:</strong> Agent, the default design was decent, but my design-first approach might’ve saved fine-tuning time</p>

<p><strong>Front-end components:</strong> Agent was fast</p>

<p><strong>Debugging the API:</strong> Me, Agent took longer and cost more</p>

<p><strong>UI fine-tuning:</strong> Me, Agent followed prompts but I’m still faster and free</p>

<p><strong>Publishing:</strong> Agent, super convenient</p>

<p><strong>Code quality:</strong> Me, but Agent was fine</p>

<h2>
  
  
  Thanks for reading!
</h2>

<p>If you found this article useful, <strong>please like and share it with someone who might enjoy it.</strong></p>

<p>And let me know, <strong>have you built anything with Replit Agent?</strong> Drop a comment below, I’d love to hear about your experiences.</p>

<p>You can also read this post on <a href="https://wefunder.com/feed/221975-building-an-interactive-map-with-replit-agent-a-software-engineer-tests-vibe" rel="noopener noreferrer">Wefunder</a> or on <a href="https://medium.com/@makenakong/building-an-interactive-map-with-replit-agent-a-software-engineer-tests-vibe-coding-efficiency-c86efbbc8fc3" rel="noopener noreferrer">Medium</a>.</p>

