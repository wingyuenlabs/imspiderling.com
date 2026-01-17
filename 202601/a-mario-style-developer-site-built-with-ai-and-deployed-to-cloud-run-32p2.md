---
Title: A Mario-Style Developer Site Built with AI and Deployed to Cloud Run
Description: 
Author: Serhii Serdiuk
Date: 2026-01-17T22:06:13.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/new-year-new-you-google-ai-2025-12-31">New Year, New You Portfolio Challenge Presented by Google AI</a></em></p>

<h2>
  
  
  About Me
</h2>

<p>Highly proficient Senior Software Engineer with over 7 years of specialized experience in front-end development. Brings a comprehensive understanding of modern technology stacks, including TypeScript and Angular. Proven expertise in constructing user-friendly interfaces, optimizing web applications, and system upgrades. Exhibits strong leadership qualities through mentoring roles and active participation in technical interviews. Dedicated to delivering high-quality software solutions for leading technology companies. Keen to apply my skills and experiences to contribute to a dynamic and innovative team.</p>

<h2>
  
  
  Portfolio
</h2>


<div class="ltag__cloud-run">
  <iframe height="600px" src="https://super-portfolio-873709033798.us-west1.run.app/">
  </iframe>
</div>


<h2>
  
  
  How I Built It
</h2>

<p>This project is a Mario-style, interactive portfolio built as a fast, static React app with a playful “game UI” look.</p>

<h3>
  
  
  Tech stack
</h3>

<ul>
<li>
<strong>React + TypeScript</strong>: UI and app structure.</li>
<li>
<strong>Vite</strong>: local dev and production builds.</li>
<li>
<strong>Tailwind CSS (CDN)</strong>: utility-first styling.</li>
<li>
<strong>Framer Motion</strong>: small interactions and page animations.</li>
<li>
<strong>Recharts</strong>: “POWER LEVELS” skills visualization.</li>
<li>
<strong>Docker (multi-stage) + Nginx</strong>: production build served as a static SPA.</li>
</ul>

<h3>
  
  
  AI tools used
</h3>

<ul>
<li>Mostly built using <strong>Google AI Studio</strong>.</li>
<li>Debugging using <strong>GitHub Copilot</strong>.</li>
</ul>

<h3>
  
  
  Deployment process
</h3>

<p>CI/CD is automated with <strong>GitHub Actions</strong>: authenticate to GCP, build/push a container to Artifact Registry, then deploy to <strong>Google Cloud Run</strong>.</p>

<h2>
  
  
  What I'm Most Proud Of
</h2>

<ul>
<li>Almost effortless built using modern AI tools.</li>
<li>Real production deployment path: containerized static hosting (Vite build → Nginx) plus one-push deploy to <strong>Google Cloud Run</strong>.</li>
</ul>

