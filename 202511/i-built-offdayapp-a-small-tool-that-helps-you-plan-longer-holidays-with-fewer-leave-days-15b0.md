---
Title: I Built Offday.app — A Small Tool That Helps You Plan Longer Holidays with Fewer Leave Days
Description: 
Author: İlyas Özkurt
Date: 2025-11-23T22:05:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’ve been experimenting with side projects for years, usually to solve small personal problems. Offday.app started exactly like that: a tiny tool I needed for myself, but it gradually turned into something people from many countries began using. So I thought it might be worth sharing here.</p>

<h2>
  
  
  The Problem I Wanted to Solve
</h2>

<p>Every year, I try to maximize my holidays by combining annual leave with public holidays. Doing this manually is tedious: checking calendars, calculating date ranges, and tryinga to see which combinations create the longest break.</p>

<p>The idea behind Offday.app is straightforward:</p>

<p><strong>Select your country → the app analyzes that year’s public holidays → it suggests date ranges where a few leave days turn into surprisingly long holidays.</strong></p>

<p>It’s simple, but it saves a lot of time and catches opportunities you probably wouldn’t notice.</p>

<h2>
  
  
  Global Usage
</h2>

<p>I initially built it only for Turkey. After refining the holiday logic, I made it work for every country. Since then, users from more than 50 countries have generated hundreds of holiday plans, even though I barely promoted it.</p>

<h2>
  
  
  Tech Stack
</h2>

<p>Since this is dev.to, here’s what’s under the hood:</p>

<ul>
<li><p><strong>Frontend:</strong> Vue 3, Vite, Tailwind</p></li>
<li><p><strong>Backend:</strong> Laravel</p></li>
<li><p><strong>Infrastructure:</strong> AWS (CloudFront, S3, EC2)</p></li>
<li><p><strong>Logic:</strong> holiday parsing, date-range scoring, caching, multi-locale output</p></li>
</ul>

<p>The date engine doesn’t brute-force all combinations. Instead, it focuses on identifying “peak opportunity windows” around existing public holidays.</p>

<h2>
  
  
  Why I’m Sharing It
</h2>

<p>I enjoy seeing other developers’ weekend projects here, so I wanted to contribute mine. Offday.app is simple, but it genuinely solves a small, real problem. If it helps someone squeeze a few extra days into their next holiday, that’s more than enough motivation for me.</p>

<p>You can try it here:<br><br>
<strong><a href="https://offday.app" rel="noopener noreferrer">https://offday.app</a></strong></p>

<p>I’m currently working on improvements such as:</p>

<ul>
<li><p>plan sharing without accounts</p></li>
<li><p>exportable holiday cards</p></li>
<li><p>more detailed country-specific rules</p></li>
<li><p>better UX for country selection and date review</p></li>
</ul>

<p>If you have suggestions or feedback, I’d be happy to hear them.</p>

