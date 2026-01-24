---
Title: 🚀 Building a Real-World CI/CD Pipeline with Jenkins, Docker, and Docker Compose
Description: 
Author: Vivek Singh
Date: 2026-01-24T21:46:50.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>From broken builds to a fully automated deployment pipeline.</strong></p>
</blockquote>

<p>As part of my DevOps learning journey, I wanted to build something practical, realistic, and close to production workflows — not just follow tutorials.</p>

<p>So I built a two-tier web application and automated its entire build, test, and deployment lifecycle using Jenkins, Docker, and Docker Compose.</p>

<p>This article walks through:<br>
    • The architecture I built<br>
    • The CI/CD pipeline design<br>
    • Real issues I faced (and fixed)<br>
    • Key DevOps learnings</p>


<h2>
  
  
  🧱 Project Overview
</h2>

<p>The goal was simple but important:</p>

<blockquote>
<p><strong>Automate everything from code commit to deployment — with no manual steps.</strong></p>
</blockquote>

<p>Tech Stack<br>
    • Frontend / API: Flask (Python)<br>
    • Database: MySQL<br>
    • CI/CD: Jenkins (Pipeline as Code)<br>
    • Containerization: Docker<br>
    • Orchestration: Docker Compose<br>
    • Version Control: GitHub</p>


<h2>
  
  
  🏗 Architecture Overview
</h2>

<p>The application follows a classic two-tier architecture:<br>
    • Web tier: Flask application<br>
    • Data tier: MySQL database<br>
    • Both services run in separate Docker containers<br>
    • Managed via Docker Compose<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GitHub
  ↓
Jenkins Pipeline
  ↓
Docker Build
  ↓
Tests
  ↓
Docker Compose Deploy
</code></pre>

</div>






<h2>
  
  
  🐳 Docker &amp; Docker Compose Setup
</h2>

<p>Docker Compose is used to define and run multiple containers together.</p>

<p>Services:<br>
    • db: MySQL 8 container<br>
    • web: Flask application container</p>

<p>Key highlights:<br>
    • Database initialization using init.sql<br>
    • Environment variables for DB connectivity<br>
    • Service dependency (web depends on db)</p>

<p>📸 Image: docker-compose.yml (services definition)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9y4joyoo6hcy3uynr1a1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9y4joyoo6hcy3uynr1a1.png" alt=" " width="800" height="520"></a></p>




<h2>
  
  
  🐍 Flask Application
</h2>

<p>The Flask app exposes simple endpoints:<br>
    • / → health check<br>
    • /users → fetches data from MySQL</p>

<p>The database connection is handled using environment variables, making the app environment-agnostic.</p>

<p>📸 Image: db.py (MySQL connection using env variables)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3rw44sm3f5eb7v8rha16.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3rw44sm3f5eb7v8rha16.png" alt=" " width="800" height="520"></a></p>

<p>📸 Image: app.py (Flask routes and app runner)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbttqwwxjtnuyy1jget58.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbttqwwxjtnuyy1jget58.png" alt=" " width="800" height="520"></a></p>

<p>This design ensures:<br>
    • No hardcoded credentials<br>
    • Easy containerization<br>
    • Smooth CI/CD execution</p>




<h2>
  
  
  🐋 Dockerfile (Application Image)
</h2>

<p>The Flask app is containerized using a simple, optimized Dockerfile:<br>
    • Lightweight base image (python:3.9-slim)<br>
    • Dependency installation via requirements.txt<br>
    • Exposes port 5000<br>
    • Runs the app using CMD</p>

<p>📸 Image: Dockerfile</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd21ha4tl9cs45roosrhb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd21ha4tl9cs45roosrhb.png" alt=" " width="800" height="520"></a></p>




<h2>
  
  
  ⚙️ Jenkins CI/CD Pipeline
</h2>

<p>This is where the real DevOps work happens.</p>

<p>The pipeline is written using Jenkins Declarative Pipeline and stored as a Jenkinsfile in the repository.</p>

<p>Pipeline Stages:<br>
    1.  Checkout – Pull source code from GitHub<br>
    2.  Build Image – Build Docker images using Docker Compose<br>
    3.  Run Tests – Start DB container and validate application<br>
    4.  Deploy – Bring up all services using Docker Compose</p>

<p>📸 Image: Jenkinsfile (pipeline stages)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5rxuoq3mb6ft94zkz0p7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5rxuoq3mb6ft94zkz0p7.png" alt=" " width="800" height="520"></a></p>




<h2>
  
  
  📊 Jenkins Stage View (Pipeline Execution)
</h2>

<p>The Jenkins Stage View clearly shows:<br>
    • Which stage failed<br>
    • How long each stage took<br>
    • When the pipeline finally turned green</p>

<p>This helped massively during debugging.</p>

<p>📸 Image: Jenkins Stage View (failed → successful build)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo09a9t9ggjgplgws0pj0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo09a9t9ggjgplgws0pj0.png" alt=" " width="800" height="435"></a></p>




<h2>
  
  
  🧨 Real Issues I Faced (and Fixed)
</h2>

<p>This project was not smooth, and that’s where the learning came from.</p>

<p>🔴 Issue 1: Jenkins couldn’t find Jenkinsfile<br>
    • Root cause: Case sensitivity<br>
    • jenkinsfile ≠ Jenkinsfile</p>

<p>✅ Fixed by forcing correct filename in Git</p>




<h2>
  
  
  🔴 Issue 2: docker: not found in Jenkins
</h2>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>• Jenkins was running inside a container
• Docker CLI wasn’t available inside Jenkins
</code></pre>

</div>

<p>✅ Installed Docker CLI inside Jenkins container</p>




<h2>
  
  
  🔴 Issue 3: Docker Compose v1 vs v2 mismatch
</h2>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>• Jenkins environment only had docker-compose
• Pipeline was using docker compose
</code></pre>

</div>

<p>✅ Aligned Jenkinsfile with available tooling</p>




<h2>
  
  
  🔴 Issue 4: Permission denied on /var/run/docker.sock
</h2>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>• Jenkins user couldn’t talk to Docker daemon
• Linux socket permission issue
</code></pre>

</div>

<p>✅ Fixed by adjusting Docker socket permissions (for local CI)</p>




<p>✅ Final Result</p>

<p>After resolving all issues:<br>
    • ✔ Fully automated CI/CD pipeline<br>
    • ✔ Build → Test → Deploy without manual steps<br>
    • ✔ Dockerized multi-container application<br>
    • ✔ Jenkins pipeline running clean and green</p>

<p>📸 Image: Successful Jenkins pipeline run</p>




<h2>
  
  
  📚 Key Learnings
</h2>

<p>This project taught me far more than any tutorial:<br>
    • DevOps is more about debugging systems than writing YAML<br>
    • Linux fundamentals matter a lot<br>
    • CI/CD environments behave differently than local machines<br>
    • Logs are your best debugging tool<br>
    • Small permission issues can break entire pipelines<br>
    • Automation only works when every layer is understood</p>




<h2>
  
  
  🔗 Source Code
</h2>

<p>GitHub Repository:<br>
👉 <a href="https://github.com/developedbyviv/Two-tier-Architecture-Application" rel="noopener noreferrer">https://github.com/developedbyviv/Two-tier-Architecture-Application</a></p>

