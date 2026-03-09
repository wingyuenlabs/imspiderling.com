---
Title: Why I ditched "Soft Deletes" for S3: Building a Physical Purge Workflow
Description: 
Author: Marx Oblan III
Date: 2026-03-09T21:57:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>As a Laravel dev, I’ve relied on SoftDeletes for years. It’s great for accidentally deleted users or blog posts, but it’s a massive liability when you’re handling sensitive files like .env configurations, private keys, or PII.</p>

<p>I started auditing how the "big" file-sharing services handle data, and I realized most of them are just flipping a deleted_at flag in a DB. The actual object stays in an S3 bucket for 30 days "just in case."</p>

<p>I got a bit paranoid about this "ghost data" sitting in cloud storage, so I spent my weekends building FortByte.io.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjix09xr9d66ue3coq303.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjix09xr9d66ue3coq303.png" alt=" " width="800" height="421"></a></p>

<p>The Goal: True Zero-Persistence<br>
I wanted a "burn after reading" workflow where the second a link expires (or is viewed), the S3 object is physically and permanently wiped. No 30-day retention, no "trash" folder, and no recovery.</p>

<p>The Stack:<br>
Backend: Laravel </p>

<p>Frontend: Inertia.js/Vue.js</p>

<p>Storage: S3</p>

<p>Infrastructure: DigitalOcean</p>

<p>How it works:<br>
Instead of just hiding the record, I’ve set up a triggered workflow. When a file hits its expiry limit:</p>

<p>The database record is purged.</p>

<p>An S3 DeleteObject command is issued immediately.</p>

<p>Any temporary logs associated with the transfer are cleared.</p>

<p>I did include an optional 30-day retention toggle for users who actually want a safety net, but by default, "Delete" actually means "Gone."</p>

<p>Why I built it:<br>
It was originally just to solve my own workflow headaches when sending keys to clients, but I’ve made it live and included a free tier for anyone else who is tired of leaving a digital trail.</p>

<p>I’d honestly love some feedback from other devs on the UI or the purging logic. If you've ever dealt with the "ghost data" problem in your own apps, how did you handle it?</p>

