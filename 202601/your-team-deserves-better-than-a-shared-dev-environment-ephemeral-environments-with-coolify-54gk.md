---
Title: Your Team Deserves Better Than a Shared Dev Environment - Ephemeral Environments with Coolify
Description: 
Author: Amir Raouf
Date: 2026-01-29T21:44:19.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  How I Discovered Coolify (And Why I'm Never Going Back)
</h3>

<p>I discovered <a href="https://coolify.io/docs/" rel="noopener noreferrer">Coolify</a> through a LinkedIn post and decided to give it a shot. Within days, I'd migrated all my side projects to it. Eventually, I even built a Python script to automate spinning apps up and down on my Coolify instance. Now, I'm bringing this workflow to my day job—here's how we're using Coolify for ephemeral preview environments at a 4-6 person dev team.</p>

<p>Coolify struck the perfect balance: a clean UI that didn't hide the Docker underneath, making it trivial to trace logs, change environment variables, or deploy preset apps with a few clicks. Best of all, it's completely open-source, so I wasn't locked into someone else's platform or proprietary magic.</p>

<h2>
  
  
  Building Ephemeral Preview Environments: The Practical Guide
</h2>

<p>Let's cut to the chase. This setup is perfect for startups and small dev teams (5-6 developers) who:</p>

<p>Don't have dedicated DevOps engineers<br>
Can't justify $200+/month for Vercel/Railway<br>
Need isolated testing but don't need Kubernetes complexity<br>
Want to move fast without breaking things</p>
<h3>
  
  
  What We're Building
</h3>

<p>The Goal: Every pull request automatically gets its own complete environment:</p>

<ul>
<li>Isolated frontend + backend + database</li>
<li>Unique URL: pr-123.yourdomain.com</li>
<li>Automatic SSL certificates</li>
<li>Auto-cleanup when PR merges</li>
<li>Total cost: $12-40/month (scales with team size)</li>
</ul>
<h3>
  
  
  What You'll Need:
</h3>

<ul>
<li>AWS Lightsail instance or Hetzner instance or whatever</li>
<li>Coolify installed (free, open-source)</li>
<li>Bitbucket/GitHub repository</li>
<li>Wildcard DNS record</li>
<li>2-3 hours for initial setup</li>
</ul>

<p>The Stack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GitHub Repository
    ↓ (GitHub Actions on PR)
Coolify API
    ↓ (orchestrates deployment)
Docker Compose
    ├─ FastAPI (backend)
    ├─ Celery Worker
    ├─ React (frontend)
    └─ PostgreSQL
    ↓ (Traefik routes traffic)
pr-123.yourdomain.com (with SSL)
</code></pre>

</div>



<p>The Flow<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Developer creates PR → GitHub Action triggers
    ↓
Coolify receives deployment request
    ↓
Spins up Docker Compose stack
    ↓
Runs Alembic migrations + seeders
    ↓
Traefik maps containers to pr-123.yourdomain.com
    ↓
Preview ready! (3-5 minutes)
    ↓
PR merged/closed → Auto cleanup
</code></pre>

</div>



<h2>
  
  
  Now let's build it.
</h2>

<h3>
  
  
  Prerequisites
</h3>

<p>What You Need:</p>

<h4>
  
  
  Infrastructure
</h4>

<ul>
<li>AWS account (for Lightsail)</li>
<li>Domain name with DNS access</li>
<li>SSH access to servers</li>
</ul>

<h4>
  
  
  Code Repository
</h4>

<ul>
<li>GitHub repository with Actions enabled</li>
<li>Docker Compose file (already done)</li>
<li>Dockerfiles for services (already done)</li>
<li>Alembic migrations setup</li>
</ul>

<h4>
  
  
  Knowledge
</h4>

<ul>
<li>Basic Docker understanding</li>
<li>GitHub Actions basics</li>
<li>Linux command line comfort</li>
</ul>

<h4>
  
  
  Assumed Done:
</h4>

<p>✅ Dockerfiles for FastAPI, React, Celery worker<br>
✅ docker-compose.yml configured<br>
✅ Alembic migrations folder structure<br>
✅ Database seeder scripts</p>
<h2>
  
  
  Step 1: AWS Lightsail Setup
</h2>

<p>We'll keep AWS setup brief since you likely know this already.</p>
<h3>
  
  
  1.1 Create Lightsail Instance
</h3>

<p><strong>Choose your tier based on team size:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Team Size</th>
<th>Instance</th>
<th>vCPU</th>
<th>RAM</th>
<th>Storage</th>
<th>Price</th>
<th>Concurrent PRs</th>
</tr>
</thead>
<tbody>
<tr>
<td>2-3 devs</td>
<td>$12/mo</td>
<td>1</td>
<td>2 GB</td>
<td>40 GB</td>
<td>$12</td>
<td>2-3</td>
</tr>
<tr>
<td>4-5 devs</td>
<td>$24/mo</td>
<td>2</td>
<td>4 GB</td>
<td>80 GB</td>
<td>$24</td>
<td>4-5</td>
</tr>
<tr>
<td>6+ devs</td>
<td>$40/mo</td>
<td>4</td>
<td>8 GB</td>
<td>160 GB</td>
<td>$40</td>
<td>6-8</td>
</tr>
</tbody>
</table></div>

<p><strong>Quick Setup:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Launch instance with Ubuntu 22.04 or 24.04</span>
<span class="c"># 2. Add your SSH key</span>
<span class="c"># 3. Open ports in firewall:</span>
<span class="c">#    - 22 (SSH)</span>
<span class="c">#    - 80 (HTTP)</span>
<span class="c">#    - 443 (HTTPS)</span>
<span class="c">#    - 8000 (Coolify UI)</span>
<span class="c"># 4. Note your instance IP address</span>
</code></pre>

</div>



<h3>
  
  
  1.2 Configure DNS
</h3>

<p>Set up wildcard DNS for preview environments:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># In your DNS provider (Cloudflare, Route53, etc.)
*.yourdomain.com    A    &lt;lightsail-ip&gt;
coolify.yourdomain.com    A    &lt;lightsail-ip&gt;
</code></pre>

</div>



<p><strong>Why wildcard?</strong></p>

<ul>
<li>Allows automatic URLs: <code>pr-123.yourdomain.com</code>, <code>pr-456.yourdomain.com</code>
</li>
<li>Coolify generates these dynamically</li>
<li>No manual DNS changes per preview</li>
</ul>




<h2>
  
  
  Step 2: Install Coolify
</h2>

<p>SSH into your Lightsail instance and install Coolify.</p>

<h3>
  
  
  2.1 Run Installation Script
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># SSH into instance</span>
ssh ubuntu@

<span class="c"># Install Coolify (takes ~5 minutes)</span>
curl <span class="nt">-fsSL</span> https://cdn.coollabs.io/coolify/install.sh | bash

<span class="c"># Installation will:</span>
<span class="c"># - Install Docker &amp; Docker Compose</span>
<span class="c"># - Set up Coolify containers</span>
<span class="c"># - Configure Traefik reverse proxy</span>
<span class="c"># - Generate SSL certificates</span>
</code></pre>

</div>



<h3>
  
  
  2.2 Access Coolify UI
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Navigate to:</span>
http://:8000

<span class="c"># Or (if DNS is configured):</span>
https://coolify.yourdomain.com
</code></pre>

</div>



<h3>
  
  
  2.3 Initial Setup
</h3>

<ol>
<li>
<p><strong>Create admin account</strong></p>

<ul>
<li>Set strong password</li>
<li>Save credentials securely</li>
</ul>
</li>
<li>
<p><strong>Configure server</strong></p>

<ul>
<li>Coolify auto-detects localhost</li>
<li>Server should show as "Connected"</li>
</ul>
</li>
<li>
<p><strong>Generate API token</strong></p>

<ul>
<li>Go to: Settings → API Tokens</li>
<li>Click "Create Token"</li>
<li>Copy token (you'll need this for GitHub Actions)</li>
<li>Store securely (treat like a password)</li>
</ul>
</li>
</ol>




<h2>
  
  
  Step 3: Configure Your Application
</h2>

<h3>
  
  
  3.1 Project Structure
</h3>

<p>Your repository should look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>your-repo/
├── backend/
│   ├── Dockerfile
│   ├── app/
│   ├── alembic/
│   │   ├── versions/
│   │   └── env.py
│   └── requirements.txt
├── frontend/
│   ├── Dockerfile
│   ├── src/
│   └── package.json
├── worker/
│   ├── Dockerfile (or shares backend Dockerfile)
│   └── tasks.py
├── db/
│   └── seeders/
│       └── seed_data.py
├── docker-compose.yml
└── .github/
    └── workflows/
        └── preview.yml
</code></pre>

</div>



<h3>
  
  
  3.2 Docker Compose Configuration
</h3>

<p>Your <code>docker-compose.yml</code> (already done, but here's what Coolify needs):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.8'</span>

<span class="na">services</span><span class="pi">:</span>
  <span class="na">backend</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./backend</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">DATABASE_URL=postgresql://user:${DB_PASSWORD}@db:5432/appdb</span>
      <span class="pi">-</span> <span class="s">REDIS_URL=redis://redis:6379/0</span>
      <span class="pi">-</span> <span class="s">ENVIRONMENT=${ENVIRONMENT:-preview}</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8000:8000"</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="na">db</span><span class="pi">:</span>
        <span class="na">condition</span><span class="pi">:</span> <span class="s">service_healthy</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="c1"># Coolify uses these labels</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.backend-${PR_NUMBER}.rule=Host(`pr-${PR_NUMBER}-api.yourdomain.com`)"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.services.backend-${PR_NUMBER}.loadbalancer.server.port=8000"</span>

  <span class="na">frontend</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./frontend</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
      <span class="na">args</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">REACT_APP_API_URL=https://pr-${PR_NUMBER}-api.yourdomain.com</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">3000:80"</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.frontend-${PR_NUMBER}.rule=Host(`pr-${PR_NUMBER}.yourdomain.com`)"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.services.frontend-${PR_NUMBER}.loadbalancer.server.port=80"</span>

  <span class="na">worker</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./backend</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
    <span class="na">command</span><span class="pi">:</span> <span class="s">celery -A tasks worker --loglevel=info</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">DATABASE_URL=postgresql://user:${DB_PASSWORD}@db:5432/appdb</span>
      <span class="pi">-</span> <span class="s">REDIS_URL=redis://redis:6379/0</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">db</span>
      <span class="pi">-</span> <span class="s">redis</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>

  <span class="na">db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:16-alpine</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">POSTGRES_USER=user</span>
      <span class="pi">-</span> <span class="s">POSTGRES_PASSWORD=${DB_PASSWORD}</span>
      <span class="pi">-</span> <span class="s">POSTGRES_DB=appdb</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">postgres_data:/var/lib/postgresql/data</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD-SHELL"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">pg_isready</span><span class="nv"> </span><span class="s">-U</span><span class="nv"> </span><span class="s">user</span><span class="nv"> </span><span class="s">-d</span><span class="nv"> </span><span class="s">appdb"</span><span class="pi">]</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">10s</span>
      <span class="na">timeout</span><span class="pi">:</span> <span class="s">5s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">5</span>

  <span class="na">redis</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">redis:7-alpine</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">app-network</span>

<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">postgres_data</span><span class="pi">:</span>

<span class="na">networks</span><span class="pi">:</span>
  <span class="na">app-network</span><span class="pi">:</span>
    <span class="na">driver</span><span class="pi">:</span> <span class="s">bridge</span>
</code></pre>

</div>



<p><strong>Key Points:</strong></p>

<ul>
<li>
<code>${PR_NUMBER}</code> will be injected by GitHub Actions</li>
<li>
<code>${DB_PASSWORD}</code> set in Coolify environment variables</li>
<li>Traefik labels handle automatic routing</li>
<li>Health checks ensure database is ready before migrations</li>
</ul>




<h2>
  
  
  Step 4: GitHub Actions Integration
</h2>

<h3>
  
  
  4.1 Add Secrets to GitHub
</h3>

<p>Go to: Repository → Settings → Secrets and variables → Actions</p>

<p>Add these secrets:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>COOLIFY_TOKEN         = &lt;your-coolify-api-token&gt;
COOLIFY_URL           = https://coolify.yourdomain.com
LIGHTSAIL_HOST        = &lt;lightsail-ip&gt;
LIGHTSAIL_SSH_KEY     = &lt;private-ssh-key&gt;
DB_PASSWORD           = &lt;database-password&gt;
</code></pre>

</div>



<h3>
  
  
  4.2 Create GitHub Actions Workflow
</h3>

<p>Create <code>.github/workflows/preview.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Preview Environment</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">pull_request</span><span class="pi">:</span>
    <span class="na">types</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">opened</span><span class="pi">,</span> <span class="nv">synchronize</span><span class="pi">,</span> <span class="nv">reopened</span><span class="pi">]</span>
    <span class="na">branches</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">main</span>

<span class="na">env</span><span class="pi">:</span>
  <span class="na">PR_NUMBER</span><span class="pi">:</span> <span class="s">${{ github.event.pull_request.number }}</span>
  <span class="na">BRANCH_NAME</span><span class="pi">:</span> <span class="s">${{ github.head_ref }}</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">deploy-preview</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up SSH</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">webfactory/ssh-agent@v0.8.0</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">ssh-private-key</span><span class="pi">:</span> <span class="s">${{ secrets.LIGHTSAIL_SSH_KEY }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy to Coolify</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">COOLIFY_TOKEN</span><span class="pi">:</span> <span class="s">${{ secrets.COOLIFY_TOKEN }}</span>
          <span class="na">COOLIFY_URL</span><span class="pi">:</span> <span class="s">${{ secrets.COOLIFY_URL }}</span>
          <span class="na">DB_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.DB_PASSWORD }}</span>
          <span class="na">HOST</span><span class="pi">:</span> <span class="s">${{ secrets.LIGHTSAIL_HOST }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s"># Create deployment directory on server</span>
          <span class="s">ssh -o StrictHostKeyChecking=no ubuntu@$HOST "mkdir -p /opt/previews/pr-${PR_NUMBER}"</span>

          <span class="s"># Copy repository to server</span>
          <span class="s">rsync -avz --exclude 'node_modules' --exclude '.git' \</span>
            <span class="s">./ ubuntu@$HOST:/opt/previews/pr-${PR_NUMBER}/</span>

          <span class="s"># Deploy via Coolify API</span>
          <span class="s">ssh ubuntu@$HOST "cd /opt/previews/pr-${PR_NUMBER} &amp;&amp; \</span>
            <span class="s">export PR_NUMBER=${PR_NUMBER} &amp;&amp; \</span>
            <span class="s">export DB_PASSWORD=${DB_PASSWORD} &amp;&amp; \</span>
            <span class="s">docker-compose -p preview-pr-${PR_NUMBER} up -d --build"</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run Database Migrations</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">HOST</span><span class="pi">:</span> <span class="s">${{ secrets.LIGHTSAIL_HOST }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">ssh ubuntu@$HOST "docker exec preview-pr-${PR_NUMBER}-backend-1 \</span>
            <span class="s">alembic upgrade head"</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Seed Database</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">HOST</span><span class="pi">:</span> <span class="s">${{ secrets.LIGHTSAIL_HOST }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">ssh ubuntu@$HOST "docker exec preview-pr-${PR_NUMBER}-backend-1 \</span>
            <span class="s">python db/seeders/seed_data.py"</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Wait for Services</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">sleep </span><span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Comment PR with Preview URL</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/github-script@v7</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">github-token</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
          <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">const prNumber = context.payload.pull_request.number;</span>
            <span class="s">const comment = `## 🚀 Preview Environment Ready!</span>

            <span class="s">**Frontend:** https://pr-${prNumber}.yourdomain.com</span>
            <span class="s">**API:** https://pr-${prNumber}-api.yourdomain.com</span>

            <span class="s">*Last updated: ${new Date().toISOString()}*</span>
            <span class="s">`;</span>

            <span class="s">github.rest.issues.createComment({</span>
              <span class="s">...context.repo,</span>
              <span class="s">issue_number: prNumber,</span>
              <span class="s">body: comment</span>
            <span class="s">});</span>
</code></pre>

</div>



<h3>
  
  
  4.3 Cleanup Workflow
</h3>

<p>Create <code>.github/workflows/cleanup-preview.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Cleanup Preview Environment</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">pull_request</span><span class="pi">:</span>
    <span class="na">types</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">closed</span><span class="pi">]</span>
    <span class="na">branches</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">main</span>

<span class="na">env</span><span class="pi">:</span>
  <span class="na">PR_NUMBER</span><span class="pi">:</span> <span class="s">${{ github.event.pull_request.number }}</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">cleanup</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up SSH</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">webfactory/ssh-agent@v0.8.0</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">ssh-private-key</span><span class="pi">:</span> <span class="s">${{ secrets.LIGHTSAIL_SSH_KEY }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Stop and Remove Containers</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">HOST</span><span class="pi">:</span> <span class="s">${{ secrets.LIGHTSAIL_HOST }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">ssh -o StrictHostKeyChecking=no ubuntu@$HOST "\</span>
            <span class="s">cd /opt/previews/pr-${PR_NUMBER} &amp;&amp; \</span>
            <span class="s">docker-compose -p preview-pr-${PR_NUMBER} down -v &amp;&amp; \</span>
            <span class="s">cd .. &amp;&amp; \</span>
            <span class="s">rm -rf pr-${PR_NUMBER}"</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Comment PR</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/github-script@v7</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">github-token</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
          <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">const prNumber = context.payload.pull_request.number;</span>
            <span class="s">github.rest.issues.createComment({</span>
              <span class="s">...context.repo,</span>
              <span class="s">issue_number: prNumber,</span>
              <span class="s">body: `## 🧹 Preview Environment Cleaned Up\n\nResources freed for PR #${prNumber}`</span>
            <span class="s">});</span>
</code></pre>

</div>






<h2>
  
  
  Step 5: Traefik Port Mapping
</h2>

<p>Coolify uses <strong>Traefik</strong> as its reverse proxy to handle routing and SSL.</p>

<h3>
  
  
  5.1 How Traefik Works
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>External Request
    ↓
https://pr-123.yourdomain.com
    ↓
DNS resolves to Lightsail IP
    ↓
Traefik (listening on :80/:443)
    ↓
Reads Traefik labels from containers
    ↓
Routes to correct Docker container
    ↓
Container responds on internal port
</code></pre>

</div>



<h3>
  
  
  5.2 Traefik Labels Explained
</h3>

<p>In your <code>docker-compose.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">labels</span><span class="pi">:</span>
  <span class="c1"># Enable Traefik for this container</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>

  <span class="c1"># Define routing rule (matches incoming domain)</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.backend-${PR_NUMBER}.rule=Host(`pr-${PR_NUMBER}-api.yourdomain.com`)"</span>

  <span class="c1"># Tell Traefik which port the container listens on</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.services.backend-${PR_NUMBER}.loadbalancer.server.port=8000"</span>

  <span class="c1"># Optional: Enable HTTPS</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.backend-${PR_NUMBER}.tls=true"</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.backend-${PR_NUMBER}.tls.certresolver=letsencrypt"</span>
</code></pre>

</div>



<h3>
  
  
  5.3 SSL Certificates
</h3>

<p>Traefik automatically requests Let's Encrypt certificates when:</p>

<ul>
<li>Container has <code>tls=true</code> label</li>
<li>Domain resolves to server IP</li>
<li>Port 80/443 are accessible</li>
</ul>

<p><strong>No manual certificate management needed!</strong></p>

<h3>
  
  
  5.4 Verify Traefik Routing
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># SSH into server</span>
ssh ubuntu@

<span class="c"># Check Traefik dashboard (if enabled)</span>
docker logs coolify-proxy

<span class="c"># Test routing</span>
curl <span class="nt">-I</span> https://pr-123.yourdomain.com

<span class="c"># Should return 200 OK with SSL certificate</span>
</code></pre>

</div>






<h2>
  
  
  Step 6: Database Migrations &amp; Seeders
</h2>

<h3>
  
  
  6.1 Alembic Migrations
</h3>

<p>Your <code>backend/alembic/env.py</code> should use environment variable:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># alembic/env.py (excerpt)
</span><span class="kn">import</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">sqlalchemy</span> <span class="kn">import</span> <span class="n">engine_from_config</span><span class="p">,</span> <span class="n">pool</span>

<span class="n">config</span><span class="p">.</span><span class="nf">set_main_option</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">sqlalchemy.url</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">'</span><span class="s">DATABASE_URL</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">postgresql://user:pass@localhost/appdb</span><span class="sh">'</span><span class="p">)</span>
<span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  6.2 Running Migrations in Workflow
</h3>

<p>The GitHub Action runs migrations automatically:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run Database Migrations</span>
  <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">ssh ubuntu@$HOST "docker exec preview-pr-${PR_NUMBER}-backend-1 \</span>
      <span class="s">alembic upgrade head"</span>
</code></pre>

</div>



<p><strong>What happens:</strong></p>

<ol>
<li>GitHub Action SSHs into server</li>
<li>Executes Alembic inside backend container</li>
<li>Migrations run against isolated preview database</li>
<li>If migration fails, deployment fails (good!)</li>
</ol>

<h3>
  
  
  6.3 Database Seeders
</h3>

<p>Create <code>backend/db/seeders/seed_data.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># db/seeders/seed_data.py
</span><span class="kn">import</span> <span class="n">os</span>
<span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">sqlalchemy.ext.asyncio</span> <span class="kn">import</span> <span class="n">create_async_engine</span><span class="p">,</span> <span class="n">AsyncSession</span>
<span class="kn">from</span> <span class="n">sqlalchemy.orm</span> <span class="kn">import</span> <span class="n">sessionmaker</span>
<span class="kn">from</span> <span class="n">sqlalchemy</span> <span class="kn">import</span> <span class="n">select</span>

<span class="c1"># Your models
</span><span class="kn">from</span> <span class="n">app.models</span> <span class="kn">import</span> <span class="n">User</span><span class="p">,</span> <span class="n">Product</span><span class="p">,</span> <span class="n">Category</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">seed_database</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Seed preview database with test data</span><span class="sh">"""</span>

    <span class="n">database_url</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">'</span><span class="s">DATABASE_URL</span><span class="sh">'</span><span class="p">)</span>

    <span class="c1"># Convert to async URL if needed
</span>    <span class="k">if</span> <span class="n">database_url</span><span class="p">.</span><span class="nf">startswith</span><span class="p">(</span><span class="sh">'</span><span class="s">postgresql://</span><span class="sh">'</span><span class="p">):</span>
        <span class="n">database_url</span> <span class="o">=</span> <span class="n">database_url</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sh">'</span><span class="s">postgresql://</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">postgresql+asyncpg://</span><span class="sh">'</span><span class="p">)</span>

    <span class="n">engine</span> <span class="o">=</span> <span class="nf">create_async_engine</span><span class="p">(</span><span class="n">database_url</span><span class="p">)</span>
    <span class="n">async_session</span> <span class="o">=</span> <span class="nf">sessionmaker</span><span class="p">(</span>
        <span class="n">engine</span><span class="p">,</span> <span class="n">class_</span><span class="o">=</span><span class="n">AsyncSession</span><span class="p">,</span> <span class="n">expire_on_commit</span><span class="o">=</span><span class="bp">False</span>
    <span class="p">)</span>

    <span class="k">async</span> <span class="k">with</span> <span class="nf">async_session</span><span class="p">()</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
        <span class="c1"># Check if already seeded
</span>        <span class="n">result</span> <span class="o">=</span> <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="nf">select</span><span class="p">(</span><span class="n">User</span><span class="p">).</span><span class="nf">limit</span><span class="p">(</span><span class="mi">1</span><span class="p">))</span>
        <span class="k">if</span> <span class="n">result</span><span class="p">.</span><span class="nf">scalar_one_or_none</span><span class="p">():</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Database already seeded, skipping...</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">return</span>

        <span class="c1"># Seed users
</span>        <span class="n">users</span> <span class="o">=</span> <span class="p">[</span>
            <span class="nc">User</span><span class="p">(</span><span class="n">email</span><span class="o">=</span><span class="sh">"</span><span class="s">admin@example.com</span><span class="sh">"</span><span class="p">,</span> <span class="n">role</span><span class="o">=</span><span class="sh">"</span><span class="s">admin</span><span class="sh">"</span><span class="p">),</span>
            <span class="nc">User</span><span class="p">(</span><span class="n">email</span><span class="o">=</span><span class="sh">"</span><span class="s">user@example.com</span><span class="sh">"</span><span class="p">,</span> <span class="n">role</span><span class="o">=</span><span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">),</span>
        <span class="p">]</span>
        <span class="n">session</span><span class="p">.</span><span class="nf">add_all</span><span class="p">(</span><span class="n">users</span><span class="p">)</span>

        <span class="c1"># Seed categories
</span>        <span class="n">categories</span> <span class="o">=</span> <span class="p">[</span>
            <span class="nc">Category</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">Electronics</span><span class="sh">"</span><span class="p">),</span>
            <span class="nc">Category</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">Books</span><span class="sh">"</span><span class="p">),</span>
        <span class="p">]</span>
        <span class="n">session</span><span class="p">.</span><span class="nf">add_all</span><span class="p">(</span><span class="n">categories</span><span class="p">)</span>

        <span class="c1"># Seed products
</span>        <span class="n">products</span> <span class="o">=</span> <span class="p">[</span>
            <span class="nc">Product</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">Laptop</span><span class="sh">"</span><span class="p">,</span> <span class="n">price</span><span class="o">=</span><span class="mf">999.99</span><span class="p">,</span> <span class="n">category</span><span class="o">=</span><span class="n">categories</span><span class="p">[</span><span class="mi">0</span><span class="p">]),</span>
            <span class="nc">Product</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">Python Guide</span><span class="sh">"</span><span class="p">,</span> <span class="n">price</span><span class="o">=</span><span class="mf">29.99</span><span class="p">,</span> <span class="n">category</span><span class="o">=</span><span class="n">categories</span><span class="p">[</span><span class="mi">1</span><span class="p">]),</span>
        <span class="p">]</span>
        <span class="n">session</span><span class="p">.</span><span class="nf">add_all</span><span class="p">(</span><span class="n">products</span><span class="p">)</span>

        <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">commit</span><span class="p">()</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">✅ Database seeded successfully</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">asyncio</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="nf">seed_database</span><span class="p">())</span>
</code></pre>

</div>



<h3>
  
  
  6.4 Seeder Execution
</h3>

<p>GitHub Action runs seeder after migrations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Seed Database</span>
  <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">ssh ubuntu@$HOST "docker exec preview-pr-${PR_NUMBER}-backend-1 \</span>
      <span class="s">python db/seeders/seed_data.py"</span>
</code></pre>

</div>



<p><strong>Flow:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>PR opened
  ↓
Containers start
  ↓
Alembic runs: CREATE TABLE users, products...
  ↓
Seeder runs: INSERT test data
  ↓
Preview ready with data
</code></pre>

</div>






<h2>
  
  
  Step 7: Auto-Cleanup
</h2>

<h3>
  
  
  7.1 How Cleanup Works
</h3>

<p>When PR is merged or closed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>PR closed/merged
    ↓
GitHub webhook triggers cleanup workflow
    ↓
SSH into server
    ↓
docker-compose down -v
    (stops containers, removes volumes)
    ↓
rm -rf /opt/previews/pr-123
    (removes code directory)
    ↓
Traefik auto-updates routes
    ↓
Resources freed
</code></pre>

</div>



<h3>
  
  
  7.2 Verify Cleanup
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check for orphaned containers</span>
docker ps <span class="nt">-a</span> | <span class="nb">grep </span>preview-pr

<span class="c"># Check for orphaned volumes</span>
docker volume <span class="nb">ls</span> | <span class="nb">grep </span>preview-pr

<span class="c"># Check disk usage</span>
<span class="nb">df</span> <span class="nt">-h</span>

<span class="c"># Check preview directories</span>
<span class="nb">ls</span> /opt/previews/
</code></pre>

</div>



<h3>
  
  
  7.3 Manual Cleanup (if needed)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># SSH into server</span>
ssh ubuntu@

<span class="c"># Stop and remove specific preview</span>
<span class="nb">cd</span> /opt/previews/pr-123
docker-compose <span class="nt">-p</span> preview-pr-123 down <span class="nt">-v</span>
<span class="nb">cd</span> ..
<span class="nb">rm</span> <span class="nt">-rf</span> pr-123

<span class="c"># Nuclear option: clean ALL previews</span>
docker system prune <span class="nt">-a</span> <span class="nt">--volumes</span> <span class="nt">-f</span>
<span class="nb">rm</span> <span class="nt">-rf</span> /opt/previews/pr-<span class="k">*</span>
</code></pre>

</div>






<h2>
  
  
  Cost Breakdown
</h2>

<h3>
  
  
  Monthly Costs
</h3>

<p><strong>Infrastructure:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AWS Lightsail $12/mo:    $12/month (2-3 concurrent previews)
AWS Lightsail $24/mo:    $24/month (4-5 concurrent previews)
AWS Lightsail $40/mo:    $40/month (6-8 concurrent previews)

Coolify (open-source):   $0/month
Domain:                  ~$12/year ($1/month)
──────────────────────────────────────
Total:                   $13-41/month
</code></pre>

</div>



<h3>
  
  
  ROI Calculation
</h3>

<p><strong>Time Savings (Conservative):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Per PR:
  - Developer time saved: 2 hours (no env conflicts)
  - QA time saved: 3 hours (immediate testing)
  Total: 5 hours per PR

Monthly (12 PRs):
  - 12 PRs × 5 hours = 60 hours saved
  - Value: 60 hours × $70/hour = $4,200/month

ROI:
  - Investment: $13-41/month
  - Return: $4,200/month
  - ROI: 10,000%+
</code></pre>

</div>



<h3>
  
  
  vs. Managed Alternatives
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Platform</th>
<th>Cost/Month</th>
<th>Previews</th>
<th>Setup</th>
<th>Control</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Coolify (This Setup)</strong></td>
<td>$13-41</td>
<td>Unlimited</td>
<td>3 hours</td>
<td>Full</td>
</tr>
<tr>
<td>Vercel</td>
<td>$200+</td>
<td>Unlimited</td>
<td>30 min</td>
<td>Limited</td>
</tr>
<tr>
<td>Railway</td>
<td>$150+</td>
<td>Per service</td>
<td>1 hour</td>
<td>Limited</td>
</tr>
<tr>
<td>Heroku Review Apps</td>
<td>$100+</td>
<td>Limited</td>
<td>2 hours</td>
<td>Limited</td>
</tr>
<tr>
<td>Render</td>
<td>$100+</td>
<td>Per service</td>
<td>1 hour</td>
<td>Limited</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Conclusion
</h2>

<p>You now have:</p>

<ul>
<li>Automatic preview environments for every PR</li>
<li>Isolated testing (no more "who broke dev?")</li>
<li>Database migrations + seeders running automatically</li>
<li>SSL certificates handled by Traefik</li>
<li>Auto-cleanup on PR close</li>
<li>Total cost: $13-41/month</li>
</ul>

<p><strong>Next Steps:</strong></p>

<ol>
<li>Try this with a small project first</li>
<li>Gradually migrate team workflows</li>
<li>Monitor costs and resource usage</li>
<li>Scale Lightsail instance as team grows</li>
</ol>

<p><strong>Questions or issues?</strong> Drop them in the comments below!</p>

