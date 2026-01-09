---
Title: 🚀 Unlocking the Future: My AI Agent Mesh Portfolio Backend for the New Year, New You Challenge
Description: 
Author: Pascal Reitermann
Date: 2026-01-09T21:35:27.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/new-year-new-you-google-ai-2025-12-31">New Year, New You Portfolio Challenge Presented by Google AI</a></em></p>

<h2>
  
  
  About Me
</h2>

<p>Hello DEV.to community! I'm Pascal, a Senior Engineer with a passion for building robust, event-driven systems. My career has spanned significant challenges, including managing and scaling over 14,000 endpoints at dmTECH. Starting from march, I'm leveraging this extensive enterprise background as a Customer Technical Trainer at Solace, where I empower developers to master Event-Driven Architecture (EDA) and messaging technologies.</p>

<p>This portfolio represents my next step: merging my deep understanding of distributed systems with the transformative power of Google AI. My goal is to showcase not just what I've done, but how I build scalable, intelligent, and cloud-native applications — and to connect with the incredible community driving innovation in AI and cloud engineering.</p>

<h2>
  
  
  Portfolio
</h2>

<p>Get a live look at my <a href="https://portfolio-backend-964537274400.europe-west3.run.app" rel="noopener noreferrer">AI Agent Mesh</a> in action! This interactive component demonstrates real-time insights and asynchronous processing.</p>


<div class="ltag__cloud-run">
  <iframe height="600px" src="https://portfolio-backend-964537274400.europe-west3.run.app">
  </iframe>
</div>


<p><em>Note: As a serverless application, the first request might experience a brief "cold start" delay.</em></p>

<h2>
  
  
  How I Built It
</h2>

<p>My journey in this challenge was about pushing the boundaries of what a portfolio can be, leveraging <strong>Google Cloud Platform (GCP)</strong> and <strong>Google AI</strong> to create an <strong>AI Agent Mesh</strong>.</p>

<p><strong>Tech Stack</strong>:</p>

<ul>
<li>
<strong>Language</strong>: Go (Golang) – Chosen for its efficiency, concurrency, and robust standard library (net/http) for a lean, high-performance backend.</li>
<li>
<strong>AI Model</strong>: Google Gemini 2.0 Flash – The core intelligence powering dynamic interactions and context-aware responses.</li>
<li>
<strong>Compute</strong>: Google Cloud Run – Fully serverless, offering "scale-to-zero" for cost efficiency and automatic scaling under load.</li>
<li>
<strong>Messaging</strong>: Google Pub/Sub – The backbone of my Event-Driven Architecture, enabling asynchronous communication and decoupling services.</li>
<li>
<strong>CI/CD</strong>: GitHub Actions – Automating Docker builds, image pushes to Google Artifact Registry, and deployments to Cloud Run.</li>
<li>
<strong>IaC</strong>: Terraform – Managing IAM roles and cloud infrastructure with code for consistency and repeatability.</li>
</ul>

<p><strong>Design Decisions &amp; Development Process</strong>:</p>

<p>From the outset, I aimed for an "AI-native" and "cloud-native" design. My backend, built with the Go standard library, emphasizes minimal overhead, crucial for serverless environments. The "Agent Mesh" concept means that user interactions don't block the UI; instead, they trigger a cascade of lightweight, asynchronous events handled by Pub/Sub, allowing for parallel processing, logging, and interaction with Gemini.</p>

<p><strong>Google AI Tools in Action – My DevOps Partner</strong>:</p>

<p>A pivotal aspect of this project was my <strong>Human-AI Collaboration with Google Gemini</strong>. Gemini wasn't just a chatbot; it was an integral partner throughout the development and deployment cycle:</p>

<ul>
<li>
<strong>Architecture &amp; Best Practices</strong>: Gemini helped refine my event-driven architecture, advising on optimal Pub/Sub topic structures and Go concurrency patterns.</li>
<li>
<strong>Cloud Engineering &amp; Debugging</strong>: When faced with complex platform challenges—such as multi-platform Docker builds (<code>linux/amd64</code> for Cloud Run from my ARM64 local machine), intricate IAM permissions (<code>artifactregistry.writer</code>, <code>run.developer</code>), or <code>exec format error</code> — Gemini acted as a Senior SRE, providing precise debugging steps and <code>gcloud</code> commands. This significantly accelerated problem-solving and deployment.</li>
<li>
<strong>Code Optimization</strong>: Gemini assisted in ensuring my Go code adhered to idiomatic patterns, making it clean, maintainable, and efficient for serverless execution.</li>
</ul>

<h2>
  
  
  What I'm Most Proud Of
</h2>

<ol>
<li>
<strong>The "Agent Mesh" in Action</strong>: I'm particularly proud of demonstrating how an event-driven approach, powered by Pub/Sub, allows for a truly decoupled and scalable AI interaction. It showcases how to move beyond simple request-response to a more robust, extensible system where user input can trigger multiple parallel intelligent agents.</li>
<li>
<strong>Lean &amp; Performant Go Backend</strong>: Building with just the Go standard library for the backend was a deliberate choice. It proves that powerful, efficient applications don't always need heavy frameworks, especially in serverless contexts where binary size and cold start times are critical.</li>
<li>
<strong>Mastering the DevOps Struggles with AI</strong>: Overcoming deployment hurdles - from Docker architecture mismatches to intricate Google Cloud IAM policies - with Gemini as my guide was incredibly rewarding. It highlights how modern engineers can leverage AI not just for code, but for navigating the complexities of cloud infrastructure, making the entire CI/CD pipeline robust and efficient.</li>
</ol>

