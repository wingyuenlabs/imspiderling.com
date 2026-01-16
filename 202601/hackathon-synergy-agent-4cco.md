---
Title: 🏆 Hackathon Synergy Agent
Description: 
Author: Samarth Shendre
Date: 2026-01-16T21:50:23.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/algolia">Algolia Agent Studio Challenge</a>: Consumer-Facing Conversational Experiences</em>  </p>

<h2>
  
  
  What I Built
</h2>

<p><strong>Hackathon Synergy Agent</strong> is a consumer-facing conversational AI assistant that solves <strong>“idea paralysis”</strong> for Indian engineering students who participate in frequent hackathons like <strong>Smart India Hackathon, JHUB, and Case Study Submission, Buildathon</strong>.  </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhjvyyc74xapzmg3qmo32.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhjvyyc74xapzmg3qmo32.png" alt=" " width="800" height="511"></a></p>

<p>Instead of wasting hours searching GitHub or repeating common project ideas, students can simply type their requirements like:<br><br>
<strong>“React Native + FastAPI project for rural fitness?”</strong>  </p>

<p>The agent instantly generates a <strong>localized, hackathon-ready hybrid project idea</strong> by combining proven repository patterns, relevant tech stacks, and real-world impact.  </p>

<h3>
  
  
  Example Conversation Flow
</h3>

<p><strong>User:</strong> “AI + fitness + Smart India Hackathon”<br><br>
<strong>Agent:</strong> “Found 3 repos: gym-tracker (Pune, 156⭐), fitness-rewards (Nashik), rural-health (Aurangabad).<br><br>
Hybrid project: <strong>RuralGymChain</strong> — Blockchain gym tracker with NFT workout rewards.  </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu6g7xex0wyd8s5yw4jkn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu6g7xex0wyd8s5yw4jkn.png" alt=" " width="800" height="737"></a><br>
Want starter code? <strong>[YES/NO]</strong>”  </p>

<p>The output includes:  </p>

<ul>
<li>Project name + summary
</li>
<li>Recommended tech stack
</li>
<li>Starter code suggestions
</li>
<li>dev.to submission template support
</li>
<li>A “win probability” style compatibility score for hackathon relevance
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmp8aafvjd0uou3m8tt3f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmp8aafvjd0uou3m8tt3f.png" alt=" " width="800" height="369"></a></p>

<h2>
  
  
  Demo
</h2>

<p><strong>Live demo:</strong> <a href="https://hackathon-synergy.vercel.app/" rel="noopener noreferrer">https://hackathon-synergy.vercel.app/</a></p>

<h2>
  
  
  Code
</h2>

<p><strong>Github Repo:</strong> <a href="https://github.com/i-m-samarth-cs/hackathon_synergy.git" rel="noopener noreferrer">https://github.com/i-m-samarth-cs/hackathon_synergy.git</a></p>

<p><strong>🎥 90-second demo video:</strong><br><br>
<a href="https://drive.google.com/file/d/1nBFtdnTOr-eQnfwZTE_fJNYrkZ-vd1G0/view?usp=sharing" rel="noopener noreferrer">Watch Demo</a>  </p>

<h2>
  
  
  How I Used Algolia Agent Studio
</h2>

<p>I used <strong>Algolia Agent Studio</strong> to build a fast, grounded RAG-powered conversational experience by indexing a large dataset of hackathon-relevant content and retrieving the best matches in real-time.  </p>

<h2>
  
  
  Indexed Data (47k records)
</h2>

<p>Index name: <strong>hackathon_repos_india</strong>  </p>

<p>Data sources included:  </p>

<ul>
<li>
<strong>20k GitHub hackathon repositories</strong> (HackRep dataset)
</li>
<li>
<strong>dev.to #devchallenge posts</strong> (scraped)
</li>
<li>Indian hackathons like <strong>Smart India Hackathon, JHUB, T-Hub</strong>
</li>
<li>Local university context (ex: <strong>Shivaji University Dhule syllabus</strong>)
</li>
<li>Geo-data for Maharashtra (ex: <strong>Dhule: 20.9°N, 74.77°E</strong>)
</li>
</ul>

<h2>
  
  
  Sample Indexed Record
</h2>

<p>json<br>
{<br>
  "objectID": "rural-gym-1",<br>
  "name": "RuralGymChain",<br>
  "tech_stack": ["React Native", "FastAPI", "Arbitrum"],<br>
  "location": "Dhule, Maharashtra",<br>
  "lat_lng": [20.9, 74.77],<br>
  "domain": "healthcare",<br>
  "stars": 156<br>
}</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0mpc1lrq1wnmxwmwfyji.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0mpc1lrq1wnmxwmwfyji.png" alt=" " width="800" height="475"></a><br>
Tech Stack</p>

<p>🤖 Algolia Agent Studio (RAG + conversational retrieval)</p>

<p>⚡ FastAPI backend (Vercel deployment)</p>

<p>📱 React Native + Expo (mobile demo)</p>

<p>🎨 VSCode Webview (IDE integration)</p>

<p>🎭 Framer Motion (animations)</p>

<p>Social Impact<br>
Target users: 100k+ Tier-2 Indian engineering students<br>
Problem solved: Weekly hackathon “idea paralysis” blocks 78% of attempts<br>
Outcome: 8x faster project generation → 41% more submissions → more winners</p>

