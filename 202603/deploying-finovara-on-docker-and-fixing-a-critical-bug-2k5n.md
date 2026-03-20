---
Title: Deploying Finovara on Docker and fixing a critical bug.
Description: 
Author: Marcin Parśniak
Date: 2026-03-20T21:35:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>I recently decided to move my Spring Boot app <strong>Finovara</strong>  to Docker.</p>

<p>At first it was just supposed to be a small improvement to the setup. <br>
In reality, it turned into a debugging session that took way longer than I expected.</p>


<h2>
  
  
  Why I even bothered with Docker
</h2>

<p>There were a few reasons behind this:</p>

<ul>
<li>I wanted <strong>more control over the environment</strong>
</li>
<li>I needed a <strong>separate database (<code>finovara-test</code>)</strong> for automated tests</li>
<li>I was tired of manually setting everything up locally</li>
<li>and I wanted something closer to a <strong>real production setup</strong>
</li>
</ul>


<h2>
  
  
  Initial setup
</h2>

<p>I started with a simple PostgreSQL container and connected my app to it.</p>

<p>Here’s a simplified version of what I used:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">finovara-db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:15</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">finovara-db</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">always</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="na">POSTGRES_DB</span><span class="pi">:</span> <span class="s">finovara</span>
      <span class="na">POSTGRES_USER</span><span class="pi">:</span> <span class="s">postgres</span>
      <span class="na">POSTGRES_PASSWORD</span><span class="pi">:</span> <span class="s">${DB_PASSWORD}</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">5432:5432"</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD-SHELL"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">pg_isready</span><span class="nv"> </span><span class="s">-U</span><span class="nv"> </span><span class="s">postgres"</span><span class="pi">]</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">5s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">10</span>

</code></pre>

</div>


<p><strong>Dockerfile:</strong><br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="s"> eclipse-temurin:21-jdk</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">COPY</span><span class="s"> target/*.jar app.jar</span>
<span class="k">ENTRYPOINT</span><span class="s"> ["java", "-jar", "app.jar"]</span>
</code></pre>

</div>


<p>Everything started without issues.<br>
Database was running, app connected, no errors.</p>

<p>So I thought I was done.</p>


<h2>
  
  
  The moment things got weird
</h2>

<p>I made some changes in the app, rebuilt everything, ran it again and nothing changed.</p>

<ul>
<li>same responses</li>
<li>same data</li>
<li>same behavior</li>
</ul>

<p>At this point I was pretty sure I messed something up.</p>


<h2>
  
  
  What I thought vs what it actually was
</h2>

<p>My first guess was Docker caching.<br>
Seemed like the obvious explanation.</p>

<p>But after digging a bit more, it turned out to be something else entirely.</p>

<p>The app was connecting to <strong>a completely different database</strong> than I expected.</p>

<p>Everything looked correct:</p>

<ul>
<li>same DB name</li>
<li>same user</li>
<li>same config</li>
</ul>

<p>So from the outside it looked like everything was fine, but I was basically working on one database and checking another.</p>


<h2>
  
  
  Fixing it
</h2>

<p>The fix wasn’t one single thing, more like a combination of small adjustments:</p>

<ul>
<li>I <strong>changed the password</strong> in my <code>.env</code> file</li>
<li>I <strong>changed the port</strong>, because something else was already using 5432</li>
<li>I cleaned up and adjusted the <strong>docker-compose config</strong>
</li>
<li>and most importantly, I actually verified which database the app connects to</li>
</ul>

<p>After that, everything finally made sense again and changes started showing up immediately.</p>

<p>Thanks for reading and visit my github!<br>
</p>
<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://assets.dev.to/assets/github-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/M4rc1nek" rel="noopener noreferrer">
        M4rc1nek
      </a> / <a href="https://github.com/M4rc1nek/finovara-backend" rel="noopener noreferrer">
        finovara-backend
      </a>
    </h2>
    <h3>
      Backend service for a personal finance management application
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">Finovara</h1>
</div>

<p><strong>Finovara</strong> is a financial management platform designed to help users effectively track
analyze, and optimize their income, expenses, and savings
The application provides a secure, bank-like experience focused on transparency,
financial awareness, and long-term money planning.</p>
<div class="markdown-heading">
<h2 class="heading-element">🎯 Purpose of the Application</h2>

</div>
<p>Finovara aims to support users in making better financial decisions by offering
clear insights into their financial activity and helping them maintain control
over their budgets and savings.</p>
<p>The platform focuses on:</p>
<ul>
<li>organizing income and expenses in a structured way</li>
<li>visualizing financial data through charts and statistics</li>
<li>supporting saving goals and spending limits</li>
<li>providing a virtual wallet concept for daily financial management</li>
</ul>
<div class="markdown-heading">
<h2 class="heading-element">🚀 Key Features</h2>

</div>
<ul>
<li>Secure user authentication and authorization</li>
<li>Income and expense tracking</li>
<li>Categorization of financial operations</li>
<li>Interactive charts and financial statistics</li>
<li>Reports summarizing spending and income trends</li>
<li>Virtual wallet management</li>
<li>Savings goals (e.g. piggy banks)</li>
<li>Spending limits and budget control</li>
<li>Scalable architecture prepared for future financial…</li>
</ul>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/M4rc1nek/finovara-backend" rel="noopener noreferrer">View on GitHub</a></div>
</div>


