---
Title: awesome-trending-repos: Auto-Tracking GitHub Trending
Description: 
Author: Furkan Köykıran
Date: 2026-03-17T21:37:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every day, hundreds of new projects appear on GitHub's trending list. However, tracking these projects and determining which ones are truly important can be time-consuming. To solve this problem, I developed the <a href="https://github.com/furkankoykiran/awesome-trending-repos" rel="noopener noreferrer">awesome-trending-repos</a> project.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" alt="GitHub Actions" width="800" height="367"></a><br>
<em>Daily automated updates with GitHub Actions</em></p>

<h2>
  
  
  What is the Project?
</h2>

<p>awesome-trending-repos is an automatically updated list that tracks projects from GitHub's trending page daily. The project runs every day at midnight UTC, analyzes trending repos, and writes the results to the README.md file.</p>

<p><strong>Key features:</strong></p>

<ul>
<li>🔄 Daily automatic updates</li>
<li>📊 Historical comparisons and ranking changes</li>
<li>📈 ASCII charts and trend visualizations</li>
<li>🏆 Fastest growing repositories (rising stars)</li>
<li>💾 7-day historical data storage</li>
</ul>

<h2>
  
  
  How Does It Work?
</h2>

<p>The project runs automatically using GitHub Actions. Every day at midnight (UTC), a workflow is triggered, scrapes the GitHub trending page, analyzes the data, and writes the results to the README.md file.</p>

<p><strong>Data collection process:</strong></p>

<ol>
<li>Scrape GitHub trending page (with Cheerio)</li>
<li>Get additional data from GitHub Search API (as fallback)</li>
<li>Categorize projects by programming language</li>
<li>Compare with historical data</li>
<li>Automatically update README.md</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frjvid897tnn6ob4yg8i9.png" alt="GitHub Actions" width="800" height="367"></a><br>
<em>Automatic workflow management with CI/CD pipeline</em></p>

<h2>
  
  
  Features
</h2>

<h3>
  
  
  Daily Automation
</h3>

<p>It runs automatically every day using GitHub Actions. No manual operation required. The system pulls trending data every day at midnight UTC and updates README.md.</p>

<h3>
  
  
  Historical Tracking
</h3>

<p>The project stores the last 7 days of data. This way you can see how a project's position in the trending list has changed. It provides detailed information about new entries, rising projects, and declining ones.</p>

<h3>
  
  
  Language-Based Ranking
</h3>

<p>It categorizes trending projects by programming language. You can track projects in popular languages like JavaScript, Python, TypeScript, Go, Rust separately.</p>

<h3>
  
  
  Visualization
</h3>

<p>With ASCII charts, you can visually track the rise and fall of projects. You can see from the charts how many stars projects have received and how fast they're growing.</p>

<h2>
  
  
  Technical Details
</h2>

<p>The project is written in Node.js and uses ES Modules. It was automated with GitHub Actions and integrated with GitHub API using the Octokit library.</p>

<p><strong>Technologies used:</strong></p>

<ul>
<li>Node.js 20+</li>
<li>GitHub Actions (cron-based scheduling)</li>
<li>Octokit (GitHub API client)</li>
<li>Cheerio (web scraping)</li>
<li>Axios (HTTP requests)</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>With the awesome-trending-repos project, you can automatically track trending projects on GitHub. With the daily updated list, you won't miss the newest and most popular projects.</p>

<p>The project creates a valuable resource both for personal use and for the developer community. It saves time through automation and makes tracking trends easier.</p>




<p><em>This article was submitted to DEV.to via <a href="https://github.com/furkankoykiran/DevTo-MCP" rel="noopener noreferrer">DevTo-MCP</a>.</em></p>

