---
Title: I procrastinated on My Portfolio for years. AI Built It in 2 Hours.
Description: 
Author: Sakshay Arora
Date: 2026-01-16T21:34:03.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>The Shoemaker's Paradox:</strong> The shoemaker makes the best shoes for the village, but his own children go barefoot.</p>

<p>I have been a software engineer for over years. I’ve architected high-performance systems for enterprise clients and led teams of engineers.</p>

<p>Yet, for the last decade, I didn't have a portfolio website.</p>

<p>Every time I started, I got stuck in the weeds. I would over-engineer the stack, get bored with the CSS, or run out of "bandwidth" after a long week of coding for work. I fell into the classic trap of perfectionism.</p>

<p>Today, I finally broke the cycle. <strong>I decided to treat Claude (AI) not as a code generator, but as my Junior Developer.</strong></p>

<p>I focused purely on <strong>Prompt Engineering</strong> and Architecture. The result? I went from a blank screen to a deployed, interactive "Dual-World" portfolio in about 2 hours.</p>

<p>Here is exactly how I did it, so you can finally ship yours too.</p>

<h2>
  
  
  The Result: A "Dual-World" Interface
</h2>

<p>Before we get into the prompts, here is what we built.</p>

<p>I didn't want a generic template. I wanted to highlight my specific skill set: bridging the gap between Web and Mobile.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdo7a3fduapaj7slnb6qt.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdo7a3fduapaj7slnb6qt.gif" alt="Animation of a split-screen portfolio website. On the left, a user scrolls through sections like About and Experience. On the right, a sticky 3D mobile phone simulator automatically updates its screen to match the content, transitioning from a splash screen to a profile view and then to an app demo." width="600" height="321"></a></p>

<p><strong>Live Demo:</strong> <a href="https://sakshya.dev" rel="noopener noreferrer">sakshya.dev</a></p>




<h2>
  
  
  The Shift: From Coder to Architect
</h2>

<p>The biggest mistake developers make with AI is asking for code snippets ("Write a React component for a navbar").</p>

<p>To build a full product, you need to act like a <strong>Senior DEV</strong>. You need to define the <strong>Systems</strong>, not the syntax.</p>

<p>Here are the exact prompts I used to build the site, step-by-step.</p>

<h3>
  
  
  Prompt 1: The Stack &amp; Foundation
</h3>

<p><em>I wanted modern, performance-first tools. No legacy bloat.</em></p>

<blockquote>
<p><strong>Prompt:</strong><br>
"Role: Senior Frontend Architect.<br>
Task: Initialize a new portfolio project.<br>
Stack Constraints:</p>

<ul>
<li>Next.js 16+ (App Router)</li>
<li>TypeScript (Strict Mode)</li>
<li>Tailwind CSS v4</li>
<li>Framer Motion (for complex animations)</li>
<li>Biome (for linting)</li>
</ul>

<p>Action: Create a clean, modular folder structure for a single-page portfolio application. Ensure the architecture supports a 'feature-sliced' design pattern."</p>
</blockquote>

<h3>
  
  
  Prompt 2: The "Dual-World" Concept (The Most Important One)
</h3>

<p><em>This is where most people fail. They describe features ("add a div"). I described the **User Experience</em><em>.</em></p>

<blockquote>
<p><strong>Prompt:</strong><br>
"I am a React Native Tech Lead. My work bridges web and mobile development. I want the design to reflect this.<br>
<strong>Design Concept: 'The Dual-World Interface'</strong></p>

<ul>
<li>
<strong>Layout:</strong> Split screen desktop view.</li>
<li>
<strong>Left Side:</strong> Clean, minimalist typography. A timeline of my experience and projects.</li>
<li>
<strong>Right Side:</strong> A 'Sticky Phone Simulator' (3D Device Frame).</li>
</ul>

<p><strong>The Interaction (Crucial):</strong><br>
As the user scrolls the content on the Left, the screen inside the Phone on the Right must transition to match the context.</p>

<ul>
<li>
<em>Hero Section</em> → Phone shows my App Splash Screen.</li>
<li>
<em>Projects Section</em> → Phone simulates an interactive App Demo of that project.</li>
<li>
<em>Contact Section</em> → Phone shows a 'Messages' UI.</li>
</ul>

<p><strong>Vibe:</strong> Dark mode, cyan accents, glassmorphism. Make it look premium."</p>
</blockquote>

<h3>
  
  
  Prompt 3: The Mobile Reality Check
</h3>

<p><em>A common AI pitfall is forgetting responsiveness. I handled this explicitly.</em></p>

<blockquote>
<p><strong>Prompt:</strong><br>
"Make it mobile responsive:</p>

<ul>
<li>On mobile devices (phones/tablets), <strong>hide the 3D phone simulator entirely</strong> (it takes too much space).</li>
<li>Collapse the layout to a clean, single-column vertical scroll.</li>
<li>Add a hamburger menu with a smooth slide-out drawer for navigation."</li>
</ul>
</blockquote>

<h3>
  
  
  Prompt 4: The Content Architecture
</h3>

<p><em>As a Senior Engineer, I know hardcoded text is technical debt. I forced the AI to separate concerns so I could update the site later without touching React components.</em></p>

<blockquote>
<p><strong>Prompt:</strong><br>
"Refactor all hardcoded text. Create a single configuration file at <code>src/config/content.ts</code>.<br>
Structure it so I can easily update:</p>

<ul>
<li>Profile Bio &amp; Titles</li>
<li>Social Links (GitHub, LinkedIn, StackOverflow)</li>
<li>Experience Timeline Data</li>
<li>Projects Data (including links to GitHub/Live Demos)</li>
<li>SEO Metadata"</li>
</ul>
</blockquote>

<h3>
  
  
  Prompt 5: The Polish
</h3>

<p><em>The difference between a "prototype" and a "product" is the details.</em></p>

<blockquote>
<p><strong>Prompt:</strong><br>
"Add the final polish:</p>

<ol>
<li>
<strong>Theme:</strong> Add a light/dark mode toggle (floating action button).</li>
<li>
<strong>Smooth Scroll:</strong> Ensure clicking nav links scrolls smoothly to the section.</li>
<li>
<strong>Active State:</strong> The nav links should highlight as I scroll past their sections.</li>
<li>
<strong>Animations:</strong> Use Framer Motion to stagger the entry of the list items."</li>
</ol>
</blockquote>




<h2>
  
  
  The Outcome
</h2>

<ul>
<li>
<strong>Total Time:</strong> ~2 Hours</li>
<li>
<strong>Lines of UI Code Written Manually:</strong> 0</li>
<li>
<strong>Lines of Config Edited:</strong> ~50 (Just pasting my resume)</li>
<li>
<strong>Cost:</strong> $0 (Deployed on Vercel)</li>
</ul>

<p>I spent my time tweaking the <em>animation timings</em> and the <em>copy</em>, rather than debugging <code>z-index</code> issues or fighting with CSS Grid alignment.</p>

<h2>
  
  
  Final Thoughts: Don't Let Code Stop You
</h2>

<p>If you are reading this and thinking, <em>"I'll build my portfolio when I have time to learn the perfect animation library,"</em> stop.</p>

<p>As developers, we often wear our ability to code from scratch as a badge of honor. But sometimes, that pride is just a barrier to shipping. Your career story is worth telling, and it doesn't matter if the <code>&lt;div&gt;</code> was written by you or an LLM.</p>

<p><strong>Your work deserves to be seen. Don't wait another decade to tell your story.</strong></p>




<h3>
  
  
  Resources
</h3>

<ul>
<li>
<strong>Live Site:</strong> <a href="https://sakshya.dev" rel="noopener noreferrer">sakshya.dev</a>
</li>
<li>
<strong>GitHub Repo:</strong> <a href="https://www.google.com/search?q=https://github.com/sakshya73/sakshya.dev" rel="noopener noreferrer">github.com/sakshya73/sakshya.dev</a> (Feel free to fork it!)</li>
<li>
<strong>Connect:</strong> <a href="https://www.linkedin.com/in/sakshyaarora" rel="noopener noreferrer">LinkedIn</a>
</li>
</ul>

<p><em>Built entirely with AI assistance using Claude.</em></p>

