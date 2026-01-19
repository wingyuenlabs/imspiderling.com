---
Title: Portfolio & Summarizing Dev Journals Using Google AI & Cloud Run
Description: 
Author: Emily Jing-Lan
Date: 2026-01-19T21:39:52.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/new-year-new-you-google-ai-2025-12-31">New Year, New You Portfolio Challenge Presented by Google AI</a></em></p>

<h2>
  
  
  About Me
</h2>

<p>I’m a software engineer with a background in education and instructional design. Before moving into engineering full time, I spent years teaching and designing learning materials, which shaped how I think about clarity, structure, and how people actually absorb technical information.</p>

<p>With this portfolio, I wanted to explore the intersection of <strong>software engineering, learning, and AI-assisted reflection</strong>—not just showcasing projects, but also capturing <em>how</em> I learn and reason about systems over time. This portfolio is both a technical artifact and a learning journal. I want to have a platform that I can store all of my works.</p>

<h2>
  
  
  Portfolio
</h2>

<p>Here is my live portfolio deployment on Google Cloud Run:</p>

<p>&lt;iframe<br>
  src="<a href="https://emily-stacy-portfolio-465092161508.us-central1.run.app/" rel="noopener noreferrer">https://emily-stacy-portfolio-465092161508.us-central1.run.app/</a>"<br>
  style="width: 100%; height: 800px; border: 1px solid #e2e8f0; border-radius: 12px;"<br>
  loading="lazy"</p>

<blockquote>

</blockquote>

<p>(For the deployment label requirement, I deployed this service with: <code>--update-labels dev-tutorial=blog-devcommunity2026</code>.)</p>

<h2>
  
  
  How I Built It
</h2>

<p><strong>Frontend</strong></p>

<ul>
<li>React + TypeScript</li>
<li>Custom journal schema to support multiple content formats (sections, tables, lists, code blocks)</li>
<li>Review Mode UI that hides details and emphasizes high-level takeaways</li>
</ul>

<p><strong>Backend</strong></p>

<ul>
<li>Node.js + Express</li>
<li>Deployed as a containerized service on <strong>Google Cloud Run</strong>
</li>
<li>A custom summarization endpoint that sends journal content to Google Gemini and normalizes the response</li>
</ul>

<p><strong>Google AI</strong></p>

<ul>
<li>Google Gemini (<code>gemini-2.5-flash</code>) for summarization</li>
<li><p>Instead of assuming a fixed AI response format, I built a resilient extraction layer that can interpret multiple JSON shapes (sections, arrays, nested objects, tables, etc.) with the help of Gemini (Note: Since I’m on the free tier, requests may return 429 (quota exceeded) errors when the usage limit is reached)</p></li>
<li><p>This mirrors real-world conditions where AI outputs aren’t always predictable</p></li>
</ul>

<p><strong>Design Decisions</strong></p>

<ul>
<li><p>I intentionally treated AI as a <em>helper</em>, not a source of truth</p></li>
<li>
<p>The system falls back gracefully when summaries can’t be confidently </p>
<h2>
  
  
  What I'm Most Proud Of
</h2>

</li>
<li><p><strong>AI integration</strong>: Handling inconsistent AI outputs without breaking the UI was one of the hardest and most rewarding parts.</p></li>
<li><p><strong>Learning-first design</strong>: The portfolio doesn’t just show results—it captures the thinking process behind them.</p></li>
<li><p><strong>End-to-end ownership</strong>: From frontend design to backend APIs to Cloud Run deployment and CI/CD, this project represents full-stack ownership.</p></li>
<li>
<p><strong>Practical AI usage</strong>: Instead of flashy demos, the AI feature solves a real personal problem—reviewing and retaining complex technical knowledge.</p>

</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekio9vaikoh3iftbt5ts.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekio9vaikoh3iftbt5ts.png" alt=" " width="800" height="470"></a></p>

