---
Title: I built a $150 SaaS Billing Platform in 12 hours using an Autonomous AI Agent. Here’s why deployment is the new bottleneck.
Description: 
Author: Вололимир Салдан
Date: 2026-03-07T21:35:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Challenge: Zero to Production in 12 hours<br>
Everyone is talking about AI coding assistants. I wanted to go further. I built an autonomous development system and gave it a high-stakes stress test:</p>

<p>"Build a production-ready SaaS billing platform from scratch. No human code. No manual deployment. Go."</p>

<p>The Result: BillingCore — a functional billing engine with subscriptions, usage-based metrics, and a dashboard.<br>
The Cost: ~$150 in AI API tokens.<br>
The Time: 12 hours (including a few "AI existential crises").</p>

<p>🛠 The Stack (Chosen Autonomously)<br>
I didn't pick the tech. The system did, based on a "performance vs. dev speed" trade-off:</p>

<p>Backend: Fastify (picked over Express for lower overhead) + Prisma + PostgreSQL + Redis.</p>

<p>Frontend: React 18 + Vite + TypeScript.</p>

<p>Infrastructure: Dockerized services deployed to Railway.</p>

<p>Auth: JWT Bearer tokens + bcrypt.</p>

<p>📉 The "Aha!" Moment: Why AI Still Struggles<br>
Writing 10,000 lines of code was the easy part. The system hit a brick wall during Production Deployment.</p>

<p>The Cookie War: Initially, the agent tried using httpOnly cookies for auth. On localhost, it worked. On Railway, with different subdomains for API and Frontend, CORS and browser security policies killed the session.</p>

<p>How the AI fixed it: Instead of me stepping in, the agent analyzed the 401 errors, recognized the Cross-Origin limitation, and refactored the entire auth strategy from Cookies to Authorization: Bearer tokens across 7 frontend pages and 30+ backend endpoints.</p>

<p>📊 By the Numbers<br>
Lines of Code: 10,000+</p>

<p>Database Schema: 12 tables (complex relations: Subscriptions ↔ Plans ↔ Usage Metrics).</p>

<p>API: 30+ RESTful endpoints with OpenAPI/Swagger docs.</p>

<p>Production Services: 4 (PostgreSQL, Redis, Backend, Frontend).</p>

<p>🧠 Lessons Learned<br>
The bottleneck for AI-driven software isn't logic anymore—it's environment synchronization. AI can write a perfect function in seconds, but making that function work across Docker, Redis, and a browser’s security sandbox is where the real "seniority" of an autonomous system is tested.</p>

<p>🔗 Explore<br>
Live Demo: <a href="https://harmonious-blessing-production-32c7.up.railway.app" rel="noopener noreferrer">https://harmonious-blessing-production-32c7.up.railway.app</a></p>

<p>Source Code: <a href="https://github.com/globoohotron-rgb/billingcore" rel="noopener noreferrer">https://github.com/globoohotron-rgb/billingcore</a></p>

<p>What would you ask an autonomous agent to build next? Let’s discuss in the comments.</p>

