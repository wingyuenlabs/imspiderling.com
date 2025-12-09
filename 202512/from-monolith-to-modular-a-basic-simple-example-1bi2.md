---
Title: From Monolith to Modular (A basic simple example)
Description: 
Author: Arsalan Bardsiri
Date: 2025-12-09T21:29:42.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  From Monolith to Modular
</h1>

<p><strong>Subtitle:</strong> <em>How I’m learning System Design by breaking my own code.</em></p>

<p>We often hear about "scaling to millions of users," but we rarely talk about the first critical step: moving from "it works on my machine" to "it works as a system."</p>

<p>As part of my deep dive into system design (following the(<a href="https://bytebytego.com/courses/system-design-interview" rel="noopener noreferrer">https://bytebytego.com/courses/system-design-interview</a>)), I am building the infrastructure for a scalable web application from scratch. My goal is to move beyond theory and actually implement the architectural patterns that power the internet.</p>

<p>Here is what I learned moving from a Single Server architecture to a 3-Tier application.</p>




<h2>
  
  
  Phase 1: The Simple Single Server (2-Tier)
</h2>

<p><strong>Repository:</strong> <a href="https://www.google.com/search?q=%5Bhttps://github.com/arsalanbardsiri/simple-single-server-app%5D(https://github.com/arsalanbardsiri/simple-single-server-app)" rel="noopener noreferrer"><code>arsalanbardsiri/simple-single-server-app</code></a></p>

<p>Every great system starts as a monolith. My first iteration was a classic "2-Tier" application. In this setup, the architecture is deceptively simple:</p>

<ol>
<li> <strong>Client Tier:</strong> The user interface (browser).</li>
<li> <strong>Server Tier:</strong> A single machine handling both the business logic (Web Server) and the data persistence (Database).</li>
</ol>

<h3>
  
  
  The Architecture
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8xmaxasmr5bin6gvs872.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8xmaxasmr5bin6gvs872.png" alt=" " width="800" height="468"></a></p>

<h3>
  
  
  Why start here?
</h3>

<p>It is the fastest way to ship. You have zero network latency between your application and your database because they live on the same hardware. Deployment is a single command. For a student project or a proof-of-concept, this is perfect.</p>

<h3>
  
  
  The Wall
</h3>

<p>However, as I learned from the "Zero to Millions" chapter, this simplicity is a trap.</p>

<ul>
<li>
<strong>Vertical Scaling Limits:</strong> To handle more users, I have to upgrade the <em>entire</em> server (more RAM, better CPU). I cannot just upgrade the database; I have to upgrade the web server too.[1]</li>
<li>
<strong>Single Point of Failure (SPOF):</strong> If my web server crashes due to a buggy script, my database goes down with it. The entire system is fragile.[2]</li>
<li>
<strong>Security Risk:</strong> In this 2-tier setup, the database often shares the same environment as the public-facing web server, increasing the attack surface.[2]</li>
</ul>




<h2>
  
  
  Phase 2: The Evolution to 3-Tier Architecture
</h2>

<p><strong>Repository:</strong> <a href="https://www.google.com/search?q=%5Bhttps://github.com/arsalanbardsiri/3tier-simple-app%5D(https://github.com/arsalanbardsiri/3tier-simple-app)" rel="noopener noreferrer"><code>arsalanbardsiri/3tier-simple-app</code></a></p>

<p>To solve the fragility of the single server, I refactored the application into a <strong>3-Tier Architecture</strong>. This is the industry standard for a reason. It physically and logically separates the system into three distinct layers.</p>

<h3>
  
  
  The Architecture
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F68109kil2ukkc979zmlj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F68109kil2ukkc979zmlj.png" alt=" " width="800" height="468"></a></p>

<ol>
<li> <strong>Presentation Tier:</strong> The frontend (Client) that the user sees.</li>
<li> <strong>Application Tier:</strong> The backend API (Business Logic) that processes requests.</li>
<li> <strong>Data Tier:</strong> The dedicated database server that stores information.</li>
</ol>

<h3>
  
  
  The "Aha!" Moment
</h3>

<p>Moving to this architecture introduced complexity—I now have to manage connection strings, network latency, and multiple deployments. So, why do it?</p>

<p><strong>1. Independent Scalability</strong><br>
This is the biggest unlock. If my application performs heavy calculations (CPU intensive), I can add more <strong>Application Servers</strong> without touching the database. If my application stores massive amounts of logs (Storage intensive), I can upgrade the <strong>Database Server</strong> without touching the application code.[3]</p>

<p><strong>2. Improved Security</strong><br>
In my single-server app, the database lived on the public-facing server. In the 3-tier app, the Database Tier is hidden behind the Application Tier. It resides in a private network layer, accessible <em>only</em> by the Application Server, never by the public internet.[4]</p>

<p><strong>3. Fault Tolerance</strong><br>
By decoupling the components, I have prepared the system for redundancy. If I need to restart the web server to deploy a patch, the database remains active.</p>




<h2>
  
  
  What's Next?
</h2>

<p>I have successfully separated the concerns, but I still have a Single Point of Failure: I only have <em>one</em> Application Server and <em>one</em> Database Server.</p>

<p>My next step in the ByteByteGo roadmap is to implement a <strong>Load Balancer</strong> (likely NGINX) to distribute traffic across multiple application servers, ensuring that if one goes down, the users never notice.</p>

<p>Follow my GitHub to see the code evolve:</p>

<ul>
<li><a href="https://github.com/arsalanbardsiri/simple-single-server-app" rel="noopener noreferrer">arsalanbardsiri/simple-single-server-app</a></li>
<li><a href="https://github.com/arsalanbardsiri/3tier-simple-app" rel="noopener noreferrer">arsalanbardsiri/3tier-simple-app</a></li>
</ul>

