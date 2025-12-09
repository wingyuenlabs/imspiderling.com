---
Title: Containerized Microservices with Full Automation
Description: 
Author: Sherifdeen Adebayo
Date: 2025-12-09T21:18:11.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  The Big Picture
</h3>

<p><strong>What we're building:</strong><br>
A production-ready TODO application with:</p>

<ul>
<li>Multiple microservices (5 different programming languages!)</li>
<li>Automated infrastructure provisioning (Terraform)</li>
<li>Automated configuration management (Ansible)</li>
<li>CI/CD pipelines with drift detection</li>
<li>Automatic HTTPS with Traefik</li>
<li>Zero-trust security model</li>
</ul>

<p><strong>Architecture:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>                           ┌──────────────┐
                           │   Traefik    │
                           │ (HTTPS/Proxy)│
                           └───────┬──────┘
                                   │
            ┌──────────────────────┼──────────────────────┐
            │                      │                      │
    ┌───────▼────────┐    ┌───────▼────────┐    ┌───────▼────────┐
    │   Frontend     │    │   Auth API     │    │   Todos API    │
    │   (Vue.js)     │    │     (Go)       │    │   (Node.js)    │
    └────────────────┘    └────────────────┘    └────────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
            ┌───────▼────────┐ ┌──▼────┐  ┌─────▼──────┐
            │   Users API    │ │ Redis │  │ Log        │
            │ (Java Spring)  │ │ Queue │  │ Processor  │
            └────────────────┘ └───────┘  │ (Python)   │
                                           └────────────┘
</code></pre>

</div>



<h3>
  
  
  Understanding the Application
</h3>

<h4>
  
  
  The Services
</h4>

<p><strong>1. Frontend (Vue.js)</strong></p>

<ul>
<li>User interface</li>
<li>Login page → TODO dashboard</li>
<li>Communicates with backend APIs</li>
<li>Port: 80/443 (via Traefik)</li>
</ul>

<p><strong>2. Auth API (Go)</strong></p>

<ul>
<li>Handles user authentication</li>
<li>Issues JWT tokens</li>
<li>Endpoint: <code>/api/auth</code>
</li>
</ul>

<p><strong>3. Todos API (Node.js)</strong></p>

<ul>
<li>Manages TODO items</li>
<li>CRUD operations</li>
<li>Requires valid JWT token</li>
<li>Endpoint: <code>/api/todos</code>
</li>
</ul>

<p><strong>4. Users API (Java Spring Boot)</strong></p>

<ul>
<li>User management</li>
<li>Profile operations</li>
<li>Endpoint: <code>/api/users</code>
</li>
</ul>

<p><strong>5. Log Processor (Python)</strong></p>

<ul>
<li>Processes background tasks</li>
<li>Consumes from Redis queue</li>
<li>Writes audit logs</li>
</ul>

<p><strong>6. Redis Queue</strong></p>

<ul>
<li>Message broker</li>
<li>Task queue for async operations</li>
</ul>

<h3>
  
  
  Phase 1: Containerization
</h3>

<p>Let's containerize each service. The key is understanding that each language has its own best practices.</p>

<h4>
  
  
  Frontend Dockerfile (Vue.js)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Multi-stage build for optimized production image</span>

<span class="c"># Stage 1: Build the application</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">node:18-alpine</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">builder</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="c"># Copy package files</span>
<span class="k">COPY</span><span class="s"> package*.json ./</span>

<span class="c"># Install dependencies</span>
<span class="k">RUN </span>npm ci <span class="nt">--only</span><span class="o">=</span>production

<span class="c"># Copy source code</span>
<span class="k">COPY</span><span class="s"> . .</span>

<span class="c"># Build for production</span>
<span class="k">RUN </span>npm run build

<span class="c"># Stage 2: Serve with nginx</span>
<span class="k">FROM</span><span class="s"> nginx:alpine</span>

<span class="c"># Copy built assets from builder stage</span>
<span class="k">COPY</span><span class="s"> --from=builder /app/dist /usr/share/nginx/html</span>

<span class="c"># Copy nginx configuration</span>
<span class="k">COPY</span><span class="s"> nginx.conf /etc/nginx/conf.d/default.conf</span>

<span class="c"># Health check</span>
<span class="k">HEALTHCHECK</span><span class="s"> --interval=30s --timeout=3s \</span>
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

<span class="k">EXPOSE</span><span class="s"> 80</span>

<span class="k">CMD</span><span class="s"> ["nginx", "-g", "daemon off;"]</span>
</code></pre>

</div>



<p><strong>Why multi-stage builds?</strong></p>

<ul>
<li>
<strong>Builder stage:</strong> 800MB (includes build tools)</li>
<li>
<strong>Final stage:</strong> 25MB (only nginx + static files)</li>
<li>97% size reduction!</li>
</ul>

<p><strong>Frontend nginx config:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">server</span> <span class="p">{</span>
    <span class="kn">listen</span> <span class="mi">80</span><span class="p">;</span>
    <span class="kn">root</span> <span class="n">/usr/share/nginx/html</span><span class="p">;</span>
    <span class="kn">index</span> <span class="s">index.html</span><span class="p">;</span>

    <span class="c1"># SPA routing - send all requests to index.html</span>
    <span class="kn">location</span> <span class="n">/</span> <span class="p">{</span>
        <span class="kn">try_files</span> <span class="nv">$uri</span> <span class="nv">$uri</span><span class="n">/</span> <span class="n">/index.html</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1"># Cache static assets</span>
    <span class="kn">location</span> <span class="p">~</span><span class="sr">*</span> <span class="err">\</span><span class="s">.(js|css|png|jpg|jpeg|gif|ico|svg)</span>$ <span class="p">{</span>
        <span class="kn">expires</span> <span class="s">1y</span><span class="p">;</span>
        <span class="kn">add_header</span> <span class="s">Cache-Control</span> <span class="s">"public,</span> <span class="s">immutable"</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1"># Security headers</span>
    <span class="kn">add_header</span> <span class="s">X-Frame-Options</span> <span class="s">"SAMEORIGIN"</span> <span class="s">always</span><span class="p">;</span>
    <span class="kn">add_header</span> <span class="s">X-Content-Type-Options</span> <span class="s">"nosniff"</span> <span class="s">always</span><span class="p">;</span>
    <span class="kn">add_header</span> <span class="s">X-XSS-Protection</span> <span class="s">"1</span><span class="p">;</span> <span class="kn">mode=block"</span> <span class="s">always</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  Auth API Dockerfile (Go)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Multi-stage build for Go</span>

<span class="c"># Stage 1: Build the binary</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">golang:1.21-alpine</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">builder</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="c"># Copy go mod files</span>
<span class="k">COPY</span><span class="s"> go.mod go.sum ./</span>

<span class="c"># Download dependencies</span>
<span class="k">RUN </span>go mod download

<span class="c"># Copy source code</span>
<span class="k">COPY</span><span class="s"> . .</span>

<span class="c"># Build the binary</span>
<span class="c"># CGO_ENABLED=0 creates a static binary (no external dependencies)</span>
<span class="k">RUN </span><span class="nv">CGO_ENABLED</span><span class="o">=</span>0 <span class="nv">GOOS</span><span class="o">=</span>linux go build <span class="nt">-a</span> <span class="nt">-installsuffix</span> cgo <span class="nt">-o</span> main .

<span class="c"># Stage 2: Create minimal runtime image</span>
<span class="k">FROM</span><span class="s"> alpine:latest</span>

<span class="c"># Add ca-certificates for HTTPS calls</span>
<span class="k">RUN </span>apk <span class="nt">--no-cache</span> add ca-certificates

<span class="k">WORKDIR</span><span class="s"> /root/</span>

<span class="c"># Copy the binary from builder</span>
<span class="k">COPY</span><span class="s"> --from=builder /app/main .</span>

<span class="c"># Health check</span>
<span class="k">HEALTHCHECK</span><span class="s"> --interval=30s --timeout=3s \</span>
  CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1

<span class="k">EXPOSE</span><span class="s"> 8080</span>

<span class="k">CMD</span><span class="s"> ["./main"]</span>
</code></pre>

</div>



<p><strong>Why this approach?</strong></p>

<ul>
<li>
<strong>Builder stage:</strong> 400MB</li>
<li>
<strong>Final stage:</strong> 15MB (just Alpine + binary)</li>
<li>Static binary = no runtime dependencies</li>
<li>Faster startup, smaller attack surface</li>
</ul>

<h4>
  
  
  Todos API Dockerfile (Node.js)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="s"> node:18-alpine</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="c"># Install dependencies first (better caching)</span>
<span class="k">COPY</span><span class="s"> package*.json ./</span>
<span class="k">RUN </span>npm ci <span class="nt">--only</span><span class="o">=</span>production

<span class="c"># Copy application code</span>
<span class="k">COPY</span><span class="s"> . .</span>

<span class="c"># Create non-root user for security</span>
<span class="k">RUN </span>addgroup <span class="nt">-g</span> 1001 <span class="nt">-S</span> nodejs <span class="o">&amp;&amp;</span> <span class="se">\
</span>    adduser <span class="nt">-S</span> nodejs <span class="nt">-u</span> 1001 <span class="o">&amp;&amp;</span> <span class="se">\
</span>    <span class="nb">chown</span> <span class="nt">-R</span> nodejs:nodejs /app

<span class="k">USER</span><span class="s"> nodejs</span>

<span class="c"># Health check</span>
<span class="k">HEALTHCHECK</span><span class="s"> --interval=30s --timeout=3s \</span>
  CMD node healthcheck.js || exit 1

<span class="k">EXPOSE</span><span class="s"> 3000</span>

<span class="k">CMD</span><span class="s"> ["node", "server.js"]</span>
</code></pre>

</div>



<p><strong>Security note:</strong> Running as non-root user limits damage if container is compromised.</p>

<h4>
  
  
  Users API Dockerfile (Java Spring Boot)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Multi-stage build for Java</span>

<span class="c"># Stage 1: Build with Maven</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">maven:3.9-eclipse-temurin-17</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">builder</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="c"># Copy pom.xml first (dependency caching)</span>
<span class="k">COPY</span><span class="s"> pom.xml ./</span>
<span class="k">RUN </span>mvn dependency:go-offline

<span class="c"># Copy source and build</span>
<span class="k">COPY</span><span class="s"> src ./src</span>
<span class="k">RUN </span>mvn clean package <span class="nt">-DskipTests</span>

<span class="c"># Stage 2: Runtime</span>
<span class="k">FROM</span><span class="s"> eclipse-temurin:17-jre-alpine</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="c"># Copy JAR from builder</span>
<span class="k">COPY</span><span class="s"> --from=builder /app/target/*.jar app.jar</span>

<span class="c"># Health check</span>
<span class="k">HEALTHCHECK</span><span class="s"> --interval=30s --timeout=3s \</span>
  CMD wget --quiet --tries=1 --spider http://localhost:8080/actuator/health || exit 1

<span class="k">EXPOSE</span><span class="s"> 8080</span>

<span class="c"># Use exec form to ensure proper signal handling</span>
<span class="k">ENTRYPOINT</span><span class="s"> ["java", "-jar", "/app/app.jar"]</span>
</code></pre>

</div>



<p><strong>Java-specific optimizations:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Production optimization flags</span>
<span class="k">ENTRYPOINT</span><span class="s"> ["java", \</span>
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+ExitOnOutOfMemoryError", \
  "-jar", "/app/app.jar"]
</code></pre>

</div>



<h4>
  
  
  Log Processor Dockerfile (Python)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="s"> python:3.11-slim</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="c"># Install dependencies</span>
<span class="k">COPY</span><span class="s"> requirements.txt ./</span>
<span class="k">RUN </span>pip <span class="nb">install</span> <span class="nt">--no-cache-dir</span> <span class="nt">-r</span> requirements.txt

<span class="c"># Copy application</span>
<span class="k">COPY</span><span class="s"> . .</span>

<span class="c"># Create non-root user</span>
<span class="k">RUN </span>useradd <span class="nt">-m</span> <span class="nt">-u</span> 1001 processor <span class="o">&amp;&amp;</span> <span class="se">\
</span>    <span class="nb">chown</span> <span class="nt">-R</span> processor:processor /app

<span class="k">USER</span><span class="s"> processor</span>

<span class="c"># Health check (check if process is running)</span>
<span class="k">HEALTHCHECK</span><span class="s"> --interval=30s --timeout=3s \</span>
  CMD ps aux | grep processor.py || exit 1

<span class="k">CMD</span><span class="s"> ["python", "processor.py"]</span>
</code></pre>

</div>



<h4>
  
  
  Docker Compose - Orchestrating Everything
</h4>

<p>Now let's tie it all together with <code>docker-compose.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.8'</span>

<span class="na">services</span><span class="pi">:</span>
  <span class="c1"># Traefik reverse proxy</span>
  <span class="na">traefik</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">traefik:v2.10</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">traefik</span>
    <span class="na">command</span><span class="pi">:</span>
      <span class="c1"># API and dashboard</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--api.dashboard=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--api.insecure=true"</span>

      <span class="c1"># Docker provider</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--providers.docker=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--providers.docker.exposedbydefault=false"</span>

      <span class="c1"># Entrypoints</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--entrypoints.web.address=:80"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--entrypoints.websecure.address=:443"</span>

      <span class="c1"># HTTP to HTTPS redirect</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--entrypoints.web.http.redirections.entrypoint.to=websecure"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--entrypoints.web.http.redirections.entrypoint.scheme=https"</span>

      <span class="c1"># Let's Encrypt</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--certificatesresolvers.letsencrypt.acme.tlschallenge=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL}"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">80:80"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">443:443"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8080:8080"</span>  <span class="c1"># Dashboard</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">/var/run/docker.sock:/var/run/docker.sock:ro"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">./letsencrypt:/letsencrypt"</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">web</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

  <span class="c1"># Frontend</span>
  <span class="na">frontend</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./frontend</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">frontend</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.frontend.rule=Host(`${DOMAIN}`)"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.frontend.entrypoints=websecure"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.frontend.tls.certresolver=letsencrypt"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.services.frontend.loadbalancer.server.port=80"</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">web</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

  <span class="c1"># Auth API</span>
  <span class="na">auth</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./auth-api</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">auth-api</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">DB_HOST=postgres</span>
      <span class="pi">-</span> <span class="s">DB_PORT=5432</span>
      <span class="pi">-</span> <span class="s">DB_NAME=${DB_NAME}</span>
      <span class="pi">-</span> <span class="s">DB_USER=${DB_USER}</span>
      <span class="pi">-</span> <span class="s">DB_PASSWORD=${DB_PASSWORD}</span>
      <span class="pi">-</span> <span class="s">JWT_SECRET=${JWT_SECRET}</span>
      <span class="pi">-</span> <span class="s">REDIS_URL=redis://redis:6379</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.auth.rule=Host(`${DOMAIN}`)</span><span class="nv"> </span><span class="s">&amp;&amp;</span><span class="nv"> </span><span class="s">PathPrefix(`/api/auth`)"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.auth.entrypoints=websecure"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.auth.tls.certresolver=letsencrypt"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.services.auth.loadbalancer.server.port=8080"</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">postgres</span>
      <span class="pi">-</span> <span class="s">redis</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">web</span>
      <span class="pi">-</span> <span class="s">backend</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

  <span class="c1"># Todos API</span>
  <span class="na">todos</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./todos-api</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">todos-api</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">DB_HOST=postgres</span>
      <span class="pi">-</span> <span class="s">DB_PORT=5432</span>
      <span class="pi">-</span> <span class="s">DB_NAME=${DB_NAME}</span>
      <span class="pi">-</span> <span class="s">DB_USER=${DB_USER}</span>
      <span class="pi">-</span> <span class="s">DB_PASSWORD=${DB_PASSWORD}</span>
      <span class="pi">-</span> <span class="s">REDIS_URL=redis://redis:6379</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.todos.rule=Host(`${DOMAIN}`)</span><span class="nv"> </span><span class="s">&amp;&amp;</span><span class="nv"> </span><span class="s">PathPrefix(`/api/todos`)"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.todos.entrypoints=websecure"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.todos.tls.certresolver=letsencrypt"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.services.todos.loadbalancer.server.port=3000"</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">postgres</span>
      <span class="pi">-</span> <span class="s">redis</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">web</span>
      <span class="pi">-</span> <span class="s">backend</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

  <span class="c1"># Users API</span>
  <span class="na">users</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./users-api</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">users-api</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/${DB_NAME}</span>
      <span class="pi">-</span> <span class="s">SPRING_DATASOURCE_USERNAME=${DB_USER}</span>
      <span class="pi">-</span> <span class="s">SPRING_DATASOURCE_PASSWORD=${DB_PASSWORD}</span>
      <span class="pi">-</span> <span class="s">SPRING_REDIS_HOST=redis</span>
      <span class="pi">-</span> <span class="s">SPRING_REDIS_PORT=6379</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.users.rule=Host(`${DOMAIN}`)</span><span class="nv"> </span><span class="s">&amp;&amp;</span><span class="nv"> </span><span class="s">PathPrefix(`/api/users`)"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.users.entrypoints=websecure"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.users.tls.certresolver=letsencrypt"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.services.users.loadbalancer.server.port=8080"</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">postgres</span>
      <span class="pi">-</span> <span class="s">redis</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">web</span>
      <span class="pi">-</span> <span class="s">backend</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

  <span class="c1"># Log Processor</span>
  <span class="na">log-processor</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">./log-processor</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">Dockerfile</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">log-processor</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">REDIS_URL=redis://redis:6379</span>
      <span class="pi">-</span> <span class="s">LOG_PATH=/logs</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">./logs:/logs</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">redis</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">backend</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

  <span class="c1"># Redis</span>
  <span class="na">redis</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">redis:7-alpine</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">redis</span>
    <span class="na">command</span><span class="pi">:</span> <span class="s">redis-server --appendonly yes</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">redis-data:/data</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">backend</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">redis-cli"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">ping"</span><span class="pi">]</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">10s</span>
      <span class="na">timeout</span><span class="pi">:</span> <span class="s">3s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">3</span>

  <span class="c1"># PostgreSQL</span>
  <span class="na">postgres</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:15-alpine</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">postgres</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">POSTGRES_DB=${DB_NAME}</span>
      <span class="pi">-</span> <span class="s">POSTGRES_USER=${DB_USER}</span>
      <span class="pi">-</span> <span class="s">POSTGRES_PASSWORD=${DB_PASSWORD}</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">postgres-data:/var/lib/postgresql/data</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">backend</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD-SHELL"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">pg_isready</span><span class="nv"> </span><span class="s">-U</span><span class="nv"> </span><span class="s">${DB_USER}"</span><span class="pi">]</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">10s</span>
      <span class="na">timeout</span><span class="pi">:</span> <span class="s">3s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">3</span>

<span class="na">networks</span><span class="pi">:</span>
  <span class="na">web</span><span class="pi">:</span>
    <span class="na">driver</span><span class="pi">:</span> <span class="s">bridge</span>
  <span class="na">backend</span><span class="pi">:</span>
    <span class="na">driver</span><span class="pi">:</span> <span class="s">bridge</span>

<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">postgres-data</span><span class="pi">:</span>
  <span class="na">redis-data</span><span class="pi">:</span>
</code></pre>

</div>



<p><strong>Key concepts in this compose file:</strong></p>

<p><strong>1. Networks:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">networks</span><span class="pi">:</span>
  <span class="na">web</span><span class="pi">:</span>      <span class="c1"># Public-facing services</span>
  <span class="na">backend</span><span class="pi">:</span>  <span class="c1"># Internal services only</span>
</code></pre>

</div>



<ul>
<li>Frontend, APIs → <code>web</code> network (accessible via Traefik)</li>
<li>Database, Redis → <code>backend</code> network only (isolated)</li>
<li>This provides network-level security</li>
</ul>

<p><strong>2. Traefik Labels:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">labels</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.enable=true"</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.auth.rule=Host(`${DOMAIN}`)</span><span class="nv"> </span><span class="s">&amp;&amp;</span><span class="nv"> </span><span class="s">PathPrefix(`/api/auth`)"</span>
  <span class="pi">-</span> <span class="s2">"</span><span class="s">traefik.http.routers.auth.tls.certresolver=letsencrypt"</span>
</code></pre>

</div>



<p>These labels tell Traefik how to route traffic:</p>

<ul>
<li>Route requests to <code>yourdomain.com/api/auth</code> → auth service</li>
<li>Automatically get SSL certificate from Let's Encrypt</li>
<li>Handle HTTPS termination</li>
</ul>

<p><strong>3. Environment Variables:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">environment</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s">DB_HOST=postgres</span>
  <span class="pi">-</span> <span class="s">JWT_SECRET=${JWT_SECRET}</span>
</code></pre>

</div>



<p>Secrets come from <code>.env</code> file (never committed to git!).</p>

<h4>
  
  
  Environment Configuration
</h4>

<p>Create <code>.env</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Domain configuration</span>
<span class="nv">DOMAIN</span><span class="o">=</span>your-domain.com
<span class="nv">ACME_EMAIL</span><span class="o">=</span>your-email@example.com

<span class="c"># Database</span>
<span class="nv">DB_NAME</span><span class="o">=</span>todoapp
<span class="nv">DB_USER</span><span class="o">=</span>todouser
<span class="nv">DB_PASSWORD</span><span class="o">=</span>change-this-strong-password

<span class="c"># Security</span>
<span class="nv">JWT_SECRET</span><span class="o">=</span>change-this-to-random-string-min-32-chars

<span class="c"># Optional: Docker registry</span>
<span class="nv">DOCKER_REGISTRY</span><span class="o">=</span>ghcr.io/yourusername
</code></pre>

</div>



<p><strong>Security checklist for .env:</strong></p>

<ul>
<li>[ ] Never commit .env to git</li>
<li>[ ] Add .env to .gitignore</li>
<li>[ ] Use strong passwords (20+ characters)</li>
<li>[ ] Use different passwords for each service</li>
<li>[ ] Rotate secrets regularly</li>
</ul>

<h3>
  
  
  Phase 2: Infrastructure as Code with Terraform
</h3>

<p>Now let's provision the cloud infrastructure automatically.</p>

<h4>
  
  
  Project Structure
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>infra/
├── terraform/
│   ├── main.tf              # Main configuration
│   ├── variables.tf         # Input variables
│   ├── outputs.tf           # Output values
│   ├── provider.tf          # Provider configuration
│   └── backend.tf           # Remote state configuration
├── ansible/
│   ├── inventory/           # Dynamic inventory
│   ├── roles/
│   │   ├── dependencies/    # Install Docker, etc.
│   │   └── deploy/          # Deploy application
│   ├── playbook.yml         # Main playbook
│   └── ansible.cfg          # Ansible configuration
└── scripts/
    ├── deploy.sh            # Deployment orchestration
    └── drift-check.sh       # Drift detection
</code></pre>

</div>



<h4>
  
  
  Terraform Configuration
</h4>

<p><strong>provider.tf:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Provider configuration</span>
<span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">required_version</span> <span class="p">=</span> <span class="s2">"&gt;= 1.0"</span>

  <span class="nx">required_providers</span> <span class="p">{</span>
    <span class="nx">aws</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">source</span>  <span class="p">=</span> <span class="s2">"hashicorp/aws"</span>
      <span class="nx">version</span> <span class="p">=</span> <span class="s2">"~&gt; 5.0"</span>
    <span class="p">}</span>

    <span class="nx">local</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">source</span>  <span class="p">=</span> <span class="s2">"hashicorp/local"</span>
      <span class="nx">version</span> <span class="p">=</span> <span class="s2">"~&gt; 2.0"</span>
    <span class="p">}</span>

    <span class="kc">null</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">source</span>  <span class="p">=</span> <span class="s2">"hashicorp/null"</span>
      <span class="nx">version</span> <span class="p">=</span> <span class="s2">"~&gt; 3.0"</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">provider</span> <span class="s2">"aws"</span> <span class="p">{</span>
  <span class="nx">region</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">aws_region</span>

  <span class="nx">default_tags</span> <span class="p">{</span>
    <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">Project</span>     <span class="p">=</span> <span class="s2">"todo-app"</span>
      <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
      <span class="nx">ManagedBy</span>   <span class="p">=</span> <span class="s2">"terraform"</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>backend.tf:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Remote state storage - crucial for team collaboration</span>
<span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">backend</span> <span class="s2">"s3"</span> <span class="p">{</span>
    <span class="nx">bucket</span>         <span class="p">=</span> <span class="s2">"your-terraform-state-bucket"</span>
    <span class="nx">key</span>            <span class="p">=</span> <span class="s2">"todo-app/terraform.tfstate"</span>
    <span class="nx">region</span>         <span class="p">=</span> <span class="s2">"us-east-1"</span>
    <span class="nx">encrypt</span>        <span class="p">=</span> <span class="kc">true</span>
    <span class="nx">dynamodb_table</span> <span class="p">=</span> <span class="s2">"terraform-state-lock"</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Why remote state?</strong></p>

<ul>
<li>Team collaboration - everyone sees same state</li>
<li>State locking - prevents concurrent modifications</li>
<li>Backup - state is backed up in S3</li>
<li>Encryption - sensitive data encrypted at rest</li>
</ul>

<p><strong>variables.tf:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"aws_region"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"AWS region to deploy resources"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"us-east-1"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"environment"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Environment name"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"production"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"instance_type"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"EC2 instance type"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"t3.medium"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"ssh_public_key"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"SSH public key for access"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"domain_name"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Domain name for the application"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"alert_email"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Email for drift detection alerts"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"app_port"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Application port"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">80</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>main.tf:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># VPC for network isolation</span>
<span class="nx">resource</span> <span class="s2">"aws_vpc"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">cidr_block</span>           <span class="p">=</span> <span class="s2">"10.0.0.0/16"</span>
  <span class="nx">enable_dns_hostnames</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="nx">enable_dns_support</span>   <span class="p">=</span> <span class="kc">true</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-vpc"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Internet Gateway</span>
<span class="nx">resource</span> <span class="s2">"aws_internet_gateway"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span> <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-igw"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Public Subnet</span>
<span class="nx">resource</span> <span class="s2">"aws_subnet"</span> <span class="s2">"public"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span>                  <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">cidr_block</span>              <span class="p">=</span> <span class="s2">"10.0.1.0/24"</span>
  <span class="nx">availability_zone</span>       <span class="p">=</span> <span class="s2">"${var.aws_region}a"</span>
  <span class="nx">map_public_ip_on_launch</span> <span class="p">=</span> <span class="kc">true</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-public-subnet"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Route Table</span>
<span class="nx">resource</span> <span class="s2">"aws_route_table"</span> <span class="s2">"public"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span> <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="nx">route</span> <span class="p">{</span>
    <span class="nx">cidr_block</span> <span class="p">=</span> <span class="s2">"0.0.0.0/0"</span>
    <span class="nx">gateway_id</span> <span class="p">=</span> <span class="nx">aws_internet_gateway</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-public-rt"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Route Table Association</span>
<span class="nx">resource</span> <span class="s2">"aws_route_table_association"</span> <span class="s2">"public"</span> <span class="p">{</span>
  <span class="nx">subnet_id</span>      <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">route_table_id</span> <span class="p">=</span> <span class="nx">aws_route_table</span><span class="p">.</span><span class="nx">public</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>

<span class="c1"># Security Group</span>
<span class="nx">resource</span> <span class="s2">"aws_security_group"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">name</span>        <span class="p">=</span> <span class="s2">"todo-app-sg"</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Security group for TODO application"</span>
  <span class="nx">vpc_id</span>      <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="c1"># SSH</span>
  <span class="nx">ingress</span> <span class="p">{</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">22</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">22</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"tcp"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
    <span class="nx">description</span> <span class="p">=</span> <span class="s2">"SSH access"</span>
  <span class="p">}</span>

  <span class="c1"># HTTP</span>
  <span class="nx">ingress</span> <span class="p">{</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">80</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">80</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"tcp"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
    <span class="nx">description</span> <span class="p">=</span> <span class="s2">"HTTP access"</span>
  <span class="p">}</span>

  <span class="c1"># HTTPS</span>
  <span class="nx">ingress</span> <span class="p">{</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">443</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">443</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"tcp"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
    <span class="nx">description</span> <span class="p">=</span> <span class="s2">"HTTPS access"</span>
  <span class="p">}</span>

  <span class="c1"># Outbound - allow all</span>
  <span class="nx">egress</span> <span class="p">{</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">0</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">0</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"-1"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
    <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Allow all outbound"</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-sg"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># SSH Key Pair</span>
<span class="nx">resource</span> <span class="s2">"aws_key_pair"</span> <span class="s2">"deployer"</span> <span class="p">{</span>
  <span class="nx">key_name</span>   <span class="p">=</span> <span class="s2">"todo-app-deployer"</span>
  <span class="nx">public_key</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">ssh_public_key</span>
<span class="p">}</span>

<span class="c1"># Latest Ubuntu AMI</span>
<span class="nx">data</span> <span class="s2">"aws_ami"</span> <span class="s2">"ubuntu"</span> <span class="p">{</span>
  <span class="nx">most_recent</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="nx">owners</span>      <span class="p">=</span> <span class="p">[</span><span class="s2">"099720109477"</span><span class="p">]</span>  <span class="c1"># Canonical</span>

  <span class="nx">filter</span> <span class="p">{</span>
    <span class="nx">name</span>   <span class="p">=</span> <span class="s2">"name"</span>
    <span class="nx">values</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"</span><span class="p">]</span>
  <span class="p">}</span>

  <span class="nx">filter</span> <span class="p">{</span>
    <span class="nx">name</span>   <span class="p">=</span> <span class="s2">"virtualization-type"</span>
    <span class="nx">values</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"hvm"</span><span class="p">]</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># EC2 Instance</span>
<span class="nx">resource</span> <span class="s2">"aws_instance"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">ami</span>                    <span class="p">=</span> <span class="nx">data</span><span class="p">.</span><span class="nx">aws_ami</span><span class="p">.</span><span class="nx">ubuntu</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">instance_type</span>          <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">instance_type</span>
  <span class="nx">key_name</span>               <span class="p">=</span> <span class="nx">aws_key_pair</span><span class="p">.</span><span class="nx">deployer</span><span class="p">.</span><span class="nx">key_name</span>
  <span class="nx">subnet_id</span>              <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">vpc_security_group_ids</span> <span class="p">=</span> <span class="p">[</span><span class="nx">aws_security_group</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">id</span><span class="p">]</span>

  <span class="nx">root_block_device</span> <span class="p">{</span>
    <span class="nx">volume_size</span> <span class="p">=</span> <span class="mi">30</span>
    <span class="nx">volume_type</span> <span class="p">=</span> <span class="s2">"gp3"</span>
    <span class="nx">encrypted</span>   <span class="p">=</span> <span class="kc">true</span>
  <span class="p">}</span>

  <span class="nx">user_data</span> <span class="p">=</span> <span class="o">&lt;&lt;-</span><span class="no">EOF</span><span class="sh">
              #!/bin/bash
              apt-get update
              apt-get install -y python3 python3-pip
</span><span class="no">              EOF

</span>  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-server"</span>
  <span class="p">}</span>

  <span class="c1"># Lifecycle rule for idempotency</span>
  <span class="nx">lifecycle</span> <span class="p">{</span>
    <span class="nx">ignore_changes</span> <span class="p">=</span> <span class="p">[</span>
      <span class="nx">user_data</span><span class="p">,</span>  <span class="c1"># Don't recreate if user_data changes</span>
      <span class="nx">ami</span><span class="p">,</span>        <span class="c1"># Don't recreate on AMI updates unless forced</span>
    <span class="p">]</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Elastic IP for stable public IP</span>
<span class="nx">resource</span> <span class="s2">"aws_eip"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">instance</span> <span class="p">=</span> <span class="nx">aws_instance</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">domain</span>   <span class="p">=</span> <span class="s2">"vpc"</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-eip"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Generate Ansible inventory</span>
<span class="nx">resource</span> <span class="s2">"local_file"</span> <span class="s2">"ansible_inventory"</span> <span class="p">{</span>
  <span class="nx">content</span> <span class="p">=</span> <span class="nx">templatefile</span><span class="p">(</span><span class="s2">"${path.module}/templates/inventory.tpl"</span><span class="p">,</span> <span class="p">{</span>
    <span class="nx">app_server_ip</span> <span class="p">=</span> <span class="nx">aws_eip</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">public_ip</span>
    <span class="nx">ssh_key_path</span>  <span class="p">=</span> <span class="s2">"~/.ssh/id_rsa"</span>
    <span class="nx">ssh_user</span>      <span class="p">=</span> <span class="s2">"ubuntu"</span>
  <span class="p">})</span>

  <span class="nx">filename</span> <span class="p">=</span> <span class="s2">"${path.module}/../ansible/inventory/hosts"</span>

  <span class="c1"># Only regenerate if values change</span>
  <span class="nx">lifecycle</span> <span class="p">{</span>
    <span class="nx">create_before_destroy</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Trigger Ansible after provisioning</span>
<span class="nx">resource</span> <span class="s2">"null_resource"</span> <span class="s2">"ansible_provisioner"</span> <span class="p">{</span>
  <span class="c1"># Run when instance changes</span>
  <span class="nx">triggers</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">instance_id</span> <span class="p">=</span> <span class="nx">aws_instance</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">id</span>
    <span class="nx">timestamp</span>   <span class="p">=</span> <span class="nx">timestamp</span><span class="p">()</span>
  <span class="p">}</span>

  <span class="c1"># Wait for instance to be ready</span>
  <span class="nx">provisioner</span> <span class="s2">"local-exec"</span> <span class="p">{</span>
    <span class="nx">command</span> <span class="p">=</span> <span class="o">&lt;&lt;-</span><span class="no">EOT</span><span class="sh">
      echo "Waiting for SSH to be ready..."
      until ssh -o StrictHostKeyChecking=no -o ConnectTimeout=2 ubuntu@${aws_eip.app.public_ip} echo "SSH Ready"; do
        sleep 5
      done

      echo "Running Ansible playbook..."
      cd ${path.module}/../ansible
      ansible-playbook -i inventory/hosts playbook.yml
</span><span class="no">    EOT
</span>  <span class="p">}</span>

  <span class="nx">depends_on</span> <span class="p">=</span> <span class="p">[</span>
    <span class="nx">local_file</span><span class="p">.</span><span class="nx">ansible_inventory</span><span class="p">,</span>
    <span class="nx">aws_eip</span><span class="p">.</span><span class="nx">app</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>templates/inventory.tpl:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="nn">[app_servers]</span>
<span class="err">todo-app</span> <span class="py">ansible_host</span><span class="p">=</span><span class="s">${app_server_ip} ansible_user=${ssh_user} ansible_ssh_private_key_file=${ssh_key_path}</span>

<span class="nn">[app_servers:vars]</span>
<span class="py">ansible_python_interpreter</span><span class="p">=</span><span class="s">/usr/bin/python3</span>
</code></pre>

</div>



<p><strong>outputs.tf:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">output</span> <span class="s2">"instance_public_ip"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Public IP of the application server"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_eip</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">public_ip</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"instance_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ID of the EC2 instance"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_instance</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"domain_name"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Domain name for the application"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">domain_name</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"ssh_command"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"SSH command to connect to the server"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="s2">"ssh ubuntu@${aws_eip.app.public_ip}"</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  Understanding Terraform Idempotency
</h4>

<p><strong>What is idempotency?</strong><br>
Running the same Terraform code multiple times produces the same result without creating duplicates.</p>

<p><strong>Example - Non-idempotent (bad):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aws_instance"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">ami</span>           <span class="p">=</span> <span class="s2">"ami-12345"</span>
  <span class="nx">instance_type</span> <span class="p">=</span> <span class="s2">"t3.medium"</span>

  <span class="c1"># This causes recreation on every apply!</span>
  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Timestamp</span> <span class="p">=</span> <span class="nx">timestamp</span><span class="p">()</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Idempotent (good):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aws_instance"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">ami</span>           <span class="p">=</span> <span class="s2">"ami-12345"</span>
  <span class="nx">instance_type</span> <span class="p">=</span> <span class="s2">"t3.medium"</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"todo-app-server"</span>
  <span class="p">}</span>

  <span class="nx">lifecycle</span> <span class="p">{</span>
    <span class="nx">ignore_changes</span> <span class="p">=</span> <span class="p">[</span>
      <span class="nx">tags</span><span class="p">[</span><span class="s2">"Timestamp"</span><span class="p">],</span>
      <span class="nx">user_data</span>
    <span class="p">]</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  Drift Detection
</h4>

<p><strong>What is drift?</strong><br>
Drift occurs when actual infrastructure differs from Terraform state (manual changes, external tools, etc.).</p>

<p><strong>drift-check.sh:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-e</span>

<span class="nb">echo</span> <span class="s2">"Checking for infrastructure drift..."</span>

<span class="c"># Run terraform plan and capture output</span>
<span class="nv">PLAN_OUTPUT</span><span class="o">=</span><span class="si">$(</span>terraform plan <span class="nt">-detailed-exitcode</span> <span class="nt">-no-color</span> 2&gt;&amp;1<span class="si">)</span> <span class="o">||</span> <span class="nv">EXIT_CODE</span><span class="o">=</span><span class="nv">$?</span>

<span class="c"># Exit codes:</span>
<span class="c"># 0 = no changes</span>
<span class="c"># 1 = error</span>
<span class="c"># 2 = changes detected (drift!)</span>

<span class="k">if</span> <span class="o">[</span> <span class="nv">$EXIT_CODE</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"✅ No drift detected - infrastructure matches desired state"</span>
    <span class="nb">exit </span>0

<span class="k">elif</span> <span class="o">[</span> <span class="nv">$EXIT_CODE</span> <span class="nt">-eq</span> 2 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"⚠️  DRIFT DETECTED - infrastructure has changed!"</span>
    <span class="nb">echo</span> <span class="s2">""</span>
    <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$PLAN_OUTPUT</span><span class="s2">"</span>
    <span class="nb">echo</span> <span class="s2">""</span>

    <span class="c"># Send email alert</span>
    ./send-drift-alert.sh <span class="s2">"</span><span class="nv">$PLAN_OUTPUT</span><span class="s2">"</span>

    <span class="c"># In CI/CD, pause for manual approval</span>
    <span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$CI</span><span class="s2">"</span> <span class="o">=</span> <span class="s2">"true"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
        </span><span class="nb">echo</span> <span class="s2">"Pausing for manual approval..."</span>
        <span class="c"># GitHub Actions, GitLab CI, etc. have approval mechanisms</span>
        <span class="nb">exit </span>2
    <span class="k">fi

else
    </span><span class="nb">echo</span> <span class="s2">"❌ Error running terraform plan"</span>
    <span class="nb">echo</span> <span class="s2">"</span><span class="nv">$PLAN_OUTPUT</span><span class="s2">"</span>
    <span class="nb">exit </span>1
<span class="k">fi</span>
</code></pre>

</div>



<p><strong>send-drift-alert.sh:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>

<span class="nv">DRIFT_DETAILS</span><span class="o">=</span><span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span>
<span class="nv">ALERT_EMAIL</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">ALERT_EMAIL</span><span class="k">:-</span><span class="nv">admin</span><span class="p">@example.com</span><span class="k">}</span><span class="s2">"</span>

<span class="c"># Using AWS SES</span>
aws ses send-email <span class="se">\</span>
  <span class="nt">--from</span> <span class="s2">"terraform@example.com"</span> <span class="se">\</span>
  <span class="nt">--to</span> <span class="s2">"</span><span class="nv">$ALERT_EMAIL</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">--subject</span> <span class="s2">"⚠️ Terraform Drift Detected"</span> <span class="se">\</span>
  <span class="nt">--text</span> <span class="s2">"</span><span class="nv">$DRIFT_DETAILS</span><span class="s2">"</span>

<span class="c"># Or using curl with Mailgun, SendGrid, etc.</span>
curl <span class="nt">-s</span> <span class="nt">--user</span> <span class="s2">"api:</span><span class="nv">$MAILGUN_API_KEY</span><span class="s2">"</span> <span class="se">\</span>
  https://api.mailgun.net/v3/<span class="nv">$MAILGUN_DOMAIN</span>/messages <span class="se">\</span>
  <span class="nt">-F</span> <span class="nv">from</span><span class="o">=</span><span class="s2">"terraform@example.com"</span> <span class="se">\</span>
  <span class="nt">-F</span> <span class="nv">to</span><span class="o">=</span><span class="s2">"</span><span class="nv">$ALERT_EMAIL</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-F</span> <span class="nv">subject</span><span class="o">=</span><span class="s2">"⚠️ Terraform Drift Detected"</span> <span class="se">\</span>
  <span class="nt">-F</span> <span class="nv">text</span><span class="o">=</span><span class="s2">"</span><span class="nv">$DRIFT_DETAILS</span><span class="s2">"</span>
</code></pre>

</div>



<h3>
  
  
  Phase 3: Configuration Management with Ansible
</h3>

<p>Terraform provisions infrastructure, Ansible configures it.</p>

<h4>
  
  
  Ansible Project Structure
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ansible/
├── inventory/
│   └── hosts                 # Generated by Terraform
├── roles/
│   ├── dependencies/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── handlers/
│   │       └── main.yml
│   └── deploy/
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       │   └── .env.j2
│       └── handlers/
│           └── main.yml
├── playbook.yml
└── ansible.cfg
</code></pre>

</div>



<h4>
  
  
  ansible.cfg
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="nn">[defaults]</span>
<span class="py">inventory</span> <span class="p">=</span> <span class="s">inventory/hosts</span>
<span class="py">remote_user</span> <span class="p">=</span> <span class="s">ubuntu</span>
<span class="py">private_key_file</span> <span class="p">=</span> <span class="s">~/.ssh/id_rsa</span>
<span class="py">host_key_checking</span> <span class="p">=</span> <span class="s">False</span>
<span class="py">retry_files_enabled</span> <span class="p">=</span> <span class="s">False</span>

<span class="c"># Faster execution
</span><span class="py">gathering</span> <span class="p">=</span> <span class="s">smart</span>
<span class="py">fact_caching</span> <span class="p">=</span> <span class="s">jsonfile</span>
<span class="py">fact_caching_connection</span> <span class="p">=</span> <span class="s">/tmp/ansible_facts</span>
<span class="py">fact_caching_timeout</span> <span class="p">=</span> <span class="s">3600</span>

<span class="c"># Better output
</span><span class="py">stdout_callback</span> <span class="p">=</span> <span class="s">yaml</span>
<span class="py">bin_ansible_callbacks</span> <span class="p">=</span> <span class="s">True</span>

<span class="nn">[ssh_connection]</span>
<span class="py">pipelining</span> <span class="p">=</span> <span class="s">True</span>
</code></pre>

</div>



<h4>
  
  
  roles/dependencies/tasks/main.yml
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="c1"># Install required system dependencies</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Update apt cache</span>
  <span class="na">apt</span><span class="pi">:</span>
    <span class="na">update_cache</span><span class="pi">:</span> <span class="s">yes</span>
    <span class="na">cache_valid_time</span><span class="pi">:</span> <span class="m">3600</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install required packages</span>
  <span class="na">apt</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">apt-transport-https</span>
      <span class="pi">-</span> <span class="s">ca-certificates</span>
      <span class="pi">-</span> <span class="s">curl</span>
      <span class="pi">-</span> <span class="s">gnupg</span>
      <span class="pi">-</span> <span class="s">lsb-release</span>
      <span class="pi">-</span> <span class="s">python3-pip</span>
      <span class="pi">-</span> <span class="s">git</span>
      <span class="pi">-</span> <span class="s">ufw</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">present</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Add Docker GPG key</span>
  <span class="na">apt_key</span><span class="pi">:</span>
    <span class="na">url</span><span class="pi">:</span> <span class="s">https://download.docker.com/linux/ubuntu/gpg</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">present</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Add Docker repository</span>
  <span class="na">apt_repository</span><span class="pi">:</span>
    <span class="na">repo</span><span class="pi">:</span> <span class="s2">"</span><span class="s">deb</span><span class="nv"> </span><span class="s">[arch=amd64]</span><span class="nv"> </span><span class="s">https://download.docker.com/linux/ubuntu</span><span class="nv"> </span><span class="s">{{</span><span class="nv"> </span><span class="s">ansible_distribution_release</span><span class="nv"> </span><span class="s">}}</span><span class="nv"> </span><span class="s">stable"</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">present</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Docker</span>
  <span class="na">apt</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">docker-ce</span>
      <span class="pi">-</span> <span class="s">docker-ce-cli</span>
      <span class="pi">-</span> <span class="s">containerd.io</span>
      <span class="pi">-</span> <span class="s">docker-buildx-plugin</span>
      <span class="pi">-</span> <span class="s">docker-compose-plugin</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">present</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>
  <span class="na">notify</span><span class="pi">:</span> <span class="s">Restart Docker</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Add user to docker group</span>
  <span class="na">user</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">ansible_user</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">groups</span><span class="pi">:</span> <span class="s">docker</span>
    <span class="na">append</span><span class="pi">:</span> <span class="s">yes</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Docker Compose (standalone)</span>
  <span class="na">get_url</span><span class="pi">:</span>
    <span class="na">url</span><span class="pi">:</span> <span class="s2">"</span><span class="s">https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-linux-x86_64"</span>
    <span class="na">dest</span><span class="pi">:</span> <span class="s">/usr/local/bin/docker-compose</span>
    <span class="na">mode</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0755'</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Configure UFW firewall</span>
  <span class="na">ufw</span><span class="pi">:</span>
    <span class="na">rule</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">item.rule</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">port</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">item.port</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">proto</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">item.proto</span><span class="nv"> </span><span class="s">}}"</span>
  <span class="na">loop</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="pi">{</span> <span class="nv">rule</span><span class="pi">:</span> <span class="s1">'</span><span class="s">allow'</span><span class="pi">,</span> <span class="nv">port</span><span class="pi">:</span> <span class="s1">'</span><span class="s">22'</span><span class="pi">,</span> <span class="nv">proto</span><span class="pi">:</span> <span class="s1">'</span><span class="s">tcp'</span> <span class="pi">}</span>
    <span class="pi">-</span> <span class="pi">{</span> <span class="nv">rule</span><span class="pi">:</span> <span class="s1">'</span><span class="s">allow'</span><span class="pi">,</span> <span class="nv">port</span><span class="pi">:</span> <span class="s1">'</span><span class="s">80'</span><span class="pi">,</span> <span class="nv">proto</span><span class="pi">:</span> <span class="s1">'</span><span class="s">tcp'</span> <span class="pi">}</span>
    <span class="pi">-</span> <span class="pi">{</span> <span class="nv">rule</span><span class="pi">:</span> <span class="s1">'</span><span class="s">allow'</span><span class="pi">,</span> <span class="nv">port</span><span class="pi">:</span> <span class="s1">'</span><span class="s">443'</span><span class="pi">,</span> <span class="nv">proto</span><span class="pi">:</span> <span class="s1">'</span><span class="s">tcp'</span> <span class="pi">}</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Enable UFW</span>
  <span class="na">ufw</span><span class="pi">:</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">enabled</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>
</code></pre>

</div>



<h4>
  
  
  roles/dependencies/handlers/main.yml
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Restart Docker</span>
  <span class="na">systemd</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">docker</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">restarted</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="s">yes</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>
</code></pre>

</div>



<h4>
  
  
  roles/deploy/tasks/main.yml
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="c1"># Deploy the application</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Create application directory</span>
  <span class="na">file</span><span class="pi">:</span>
    <span class="na">path</span><span class="pi">:</span> <span class="s">/opt/todo-app</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">directory</span>
    <span class="na">owner</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">ansible_user</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">group</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">ansible_user</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">mode</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0755'</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">yes</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Clone application repository</span>
  <span class="na">git</span><span class="pi">:</span>
    <span class="na">repo</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">app_repo_url</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">dest</span><span class="pi">:</span> <span class="s">/opt/todo-app</span>
    <span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">app_branch</span><span class="nv"> </span><span class="s">|</span><span class="nv"> </span><span class="s">default('main')</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">force</span><span class="pi">:</span> <span class="s">yes</span>
  <span class="na">register</span><span class="pi">:</span> <span class="s">git_clone</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Create environment file from template</span>
  <span class="na">template</span><span class="pi">:</span>
    <span class="na">src</span><span class="pi">:</span> <span class="s">.env.j2</span>
    <span class="na">dest</span><span class="pi">:</span> <span class="s">/opt/todo-app/.env</span>
    <span class="na">owner</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">ansible_user</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">mode</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0600'</span>
  <span class="na">no_log</span><span class="pi">:</span> <span class="s">yes</span>  <span class="c1"># Don't log sensitive env vars</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Create letsencrypt directory</span>
  <span class="na">file</span><span class="pi">:</span>
    <span class="na">path</span><span class="pi">:</span> <span class="s">/opt/todo-app/letsencrypt</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">directory</span>
    <span class="na">mode</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0755'</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Pull latest Docker images</span>
  <span class="na">community.docker.docker_compose</span><span class="pi">:</span>
    <span class="na">project_src</span><span class="pi">:</span> <span class="s">/opt/todo-app</span>
    <span class="na">pull</span><span class="pi">:</span> <span class="s">yes</span>
  <span class="na">when</span><span class="pi">:</span> <span class="s">git_clone.changed</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Start application with Docker Compose</span>
  <span class="na">community.docker.docker_compose</span><span class="pi">:</span>
    <span class="na">project_src</span><span class="pi">:</span> <span class="s">/opt/todo-app</span>
    <span class="na">state</span><span class="pi">:</span> <span class="s">present</span>
    <span class="na">restarted</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">git_clone.changed</span><span class="nv"> </span><span class="s">}}"</span>
  <span class="na">register</span><span class="pi">:</span> <span class="s">compose_output</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Wait for application to be healthy</span>
  <span class="na">uri</span><span class="pi">:</span>
    <span class="na">url</span><span class="pi">:</span> <span class="s2">"</span><span class="s">https://{{</span><span class="nv"> </span><span class="s">domain_name</span><span class="nv"> </span><span class="s">}}/health"</span>
    <span class="na">status_code</span><span class="pi">:</span> <span class="m">200</span>
    <span class="na">validate_certs</span><span class="pi">:</span> <span class="s">no</span>
  <span class="na">retries</span><span class="pi">:</span> <span class="m">10</span>
  <span class="na">delay</span><span class="pi">:</span> <span class="m">10</span>
  <span class="na">register</span><span class="pi">:</span> <span class="s">health_check</span>
  <span class="na">until</span><span class="pi">:</span> <span class="s">health_check.status == </span><span class="m">200</span>

<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Display deployment status</span>
  <span class="na">debug</span><span class="pi">:</span>
    <span class="na">msg</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Application</span><span class="nv"> </span><span class="s">deployed</span><span class="nv"> </span><span class="s">successfully</span><span class="nv"> </span><span class="s">at</span><span class="nv"> </span><span class="s">https://{{</span><span class="nv"> </span><span class="s">domain_name</span><span class="nv"> </span><span class="s">}}"</span>
</code></pre>

</div>



<h4>
  
  
  roles/deploy/templates/.env.j2
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Auto-generated by Ansible - DO NOT EDIT MANUALLY

# Domain configuration
DOMAIN={{ domain_name }}
ACME_EMAIL={{ acme_email }}

# Database
DB_NAME={{ db_name }}
DB_USER={{ db_user }}
DB_PASSWORD={{ db_password }}

# Security
JWT_SECRET={{ jwt_secret }}

# Application
NODE_ENV=production
LOG_LEVEL=info
</code></pre>

</div>



<h4>
  
  
  playbook.yml
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy TODO Application</span>
  <span class="na">hosts</span><span class="pi">:</span> <span class="s">app_servers</span>
  <span class="na">become</span><span class="pi">:</span> <span class="s">no</span>

  <span class="na">vars</span><span class="pi">:</span>
    <span class="na">app_repo_url</span><span class="pi">:</span> <span class="s2">"</span><span class="s">https://github.com/yourusername/todo-app.git"</span>
    <span class="na">app_branch</span><span class="pi">:</span> <span class="s2">"</span><span class="s">main"</span>
    <span class="na">domain_name</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">lookup('env',</span><span class="nv"> </span><span class="s">'DOMAIN')</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">acme_email</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">lookup('env',</span><span class="nv"> </span><span class="s">'ACME_EMAIL')</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">db_name</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">lookup('env',</span><span class="nv"> </span><span class="s">'DB_NAME')</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">db_user</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">lookup('env',</span><span class="nv"> </span><span class="s">'DB_USER')</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">db_password</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">lookup('env',</span><span class="nv"> </span><span class="s">'DB_PASSWORD')</span><span class="nv"> </span><span class="s">}}"</span>
    <span class="na">jwt_secret</span><span class="pi">:</span> <span class="s2">"</span><span class="s">{{</span><span class="nv"> </span><span class="s">lookup('env',</span><span class="nv"> </span><span class="s">'JWT_SECRET')</span><span class="nv"> </span><span class="s">}}"</span>

  <span class="na">roles</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">dependencies</span>
    <span class="pi">-</span> <span class="s">deploy</span>

  <span class="na">post_tasks</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Verify deployment</span>
      <span class="na">uri</span><span class="pi">:</span>
        <span class="na">url</span><span class="pi">:</span> <span class="s2">"</span><span class="s">https://{{</span><span class="nv"> </span><span class="s">domain_name</span><span class="nv"> </span><span class="s">}}"</span>
        <span class="na">status_code</span><span class="pi">:</span> <span class="m">200</span>
        <span class="na">validate_certs</span><span class="pi">:</span> <span class="s">yes</span>
      <span class="na">delegate_to</span><span class="pi">:</span> <span class="s">localhost</span>

    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Display application URL</span>
      <span class="na">debug</span><span class="pi">:</span>
        <span class="na">msg</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Application</span><span class="nv"> </span><span class="s">is</span><span class="nv"> </span><span class="s">live</span><span class="nv"> </span><span class="s">at</span><span class="nv"> </span><span class="s">https://{{</span><span class="nv"> </span><span class="s">domain_name</span><span class="nv"> </span><span class="s">}}"</span>
</code></pre>

</div>



<h3>
  
  
  Phase 4: CI/CD Pipeline
</h3>

<p>Now let's automate everything with GitHub Actions.</p>

<h4>
  
  
  .github/workflows/infrastructure.yml
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Infrastructure Deployment</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">]</span>
    <span class="na">paths</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">infra/terraform/**'</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">infra/ansible/**'</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">.github/workflows/infrastructure.yml'</span>
  <span class="na">workflow_dispatch</span><span class="pi">:</span>  <span class="c1"># Manual trigger</span>

<span class="na">env</span><span class="pi">:</span>
  <span class="na">TF_VERSION</span><span class="pi">:</span> <span class="s1">'</span><span class="s">1.6.0'</span>
  <span class="na">AWS_REGION</span><span class="pi">:</span> <span class="s1">'</span><span class="s">us-east-1'</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">terraform-plan</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Terraform Plan &amp; Drift Detection</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">outputs</span><span class="pi">:</span>
      <span class="na">has_changes</span><span class="pi">:</span> <span class="s">${{ steps.plan.outputs.has_changes }}</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v3</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Setup Terraform</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">hashicorp/setup-terraform@v2</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">terraform_version</span><span class="pi">:</span> <span class="s">${{ env.TF_VERSION }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Configure AWS credentials</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">aws-actions/configure-aws-credentials@v2</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">aws-access-key-id</span><span class="pi">:</span> <span class="s">${{ secrets.AWS_ACCESS_KEY_ID }}</span>
          <span class="na">aws-secret-access-key</span><span class="pi">:</span> <span class="s">${{ secrets.AWS_SECRET_ACCESS_KEY }}</span>
          <span class="na">aws-region</span><span class="pi">:</span> <span class="s">${{ env.AWS_REGION }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Terraform Init</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd infra/terraform</span>
          <span class="s">terraform init</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Terraform Plan</span>
        <span class="na">id</span><span class="pi">:</span> <span class="s">plan</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd infra/terraform</span>
          <span class="s">terraform plan -detailed-exitcode -out=tfplan || EXIT_CODE=$?</span>

          <span class="s">if [ $EXIT_CODE -eq 0 ]; then</span>
            <span class="s">echo "has_changes=false" &gt;&gt; $GITHUB_OUTPUT</span>
            <span class="s">echo "✅ No infrastructure changes detected"</span>
          <span class="s">elif [ $EXIT_CODE -eq 2 ]; then</span>
            <span class="s">echo "has_changes=true" &gt;&gt; $GITHUB_OUTPUT</span>
            <span class="s">echo "⚠️  Infrastructure drift detected!"</span>
          <span class="s">else</span>
            <span class="s">echo "❌ Terraform plan failed"</span>
            <span class="s">exit 1</span>
          <span class="s">fi</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Save plan</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">steps.plan.outputs.has_changes == 'true'</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v3</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">tfplan</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">infra/terraform/tfplan</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Send drift alert email</span>
        <span class="na">if</span><span class="pi">:</span> <span class="s">steps.plan.outputs.has_changes == 'true'</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">dawidd6/action-send-mail@v3</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">server_address</span><span class="pi">:</span> <span class="s">smtp.gmail.com</span>
          <span class="na">server_port</span><span class="pi">:</span> <span class="m">465</span>
          <span class="na">username</span><span class="pi">:</span> <span class="s">${{ secrets.MAIL_USERNAME }}</span>
          <span class="na">password</span><span class="pi">:</span> <span class="s">${{ secrets.MAIL_PASSWORD }}</span>
          <span class="na">subject</span><span class="pi">:</span> <span class="s">⚠️ Terraform Drift Detected - TODO App</span>
          <span class="na">to</span><span class="pi">:</span> <span class="s">${{ secrets.ALERT_EMAIL }}</span>
          <span class="na">from</span><span class="pi">:</span> <span class="s">Terraform CI/CD</span>
          <span class="na">body</span><span class="pi">:</span> <span class="pi">|</span>
            <span class="s">Infrastructure drift has been detected!</span>

            <span class="s">Review the changes and approve the workflow to apply:</span>
            <span class="s">${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}</span>

  <span class="na">terraform-apply</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Terraform Apply</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="s">terraform-plan</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">needs.terraform-plan.outputs.has_changes == 'true'</span>
    <span class="na">environment</span><span class="pi">:</span> <span class="s">production</span>  <span class="c1"># Requires manual approval</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v3</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Setup Terraform</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">hashicorp/setup-terraform@v2</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">terraform_version</span><span class="pi">:</span> <span class="s">${{ env.TF_VERSION }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Configure AWS credentials</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">aws-actions/configure-aws-credentials@v2</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">aws-access-key-id</span><span class="pi">:</span> <span class="s">${{ secrets.AWS_ACCESS_KEY_ID }}</span>
          <span class="na">aws-secret-access-key</span><span class="pi">:</span> <span class="s">${{ secrets.AWS_SECRET_ACCESS_KEY }}</span>
          <span class="na">aws-region</span><span class="pi">:</span> <span class="s">${{ env.AWS_REGION }}</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Download plan</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/download-artifact@v3</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">tfplan</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">infra/terraform/</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Terraform Init</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd infra/terraform</span>
          <span class="s">terraform init</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Terraform Apply</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd infra/terraform</span>
          <span class="s">terraform apply tfplan</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Save outputs</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd infra/terraform</span>
          <span class="s">terraform output -json &gt; outputs.json</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Upload outputs</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/upload-artifact@v3</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">name</span><span class="pi">:</span> <span class="s">terraform-outputs</span>
          <span class="na">path</span><span class="pi">:</span> <span class="s">infra/terraform/outputs.json</span>

  <span class="na">ansible-deploy</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Ansible Deployment</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">needs</span><span class="pi">:</span> <span class="s">terraform-apply</span>
    <span class="na">if</span><span class="pi">:</span> <span class="s">always() &amp;&amp; (needs.terraform-apply.result == 'success' || needs.terraform-plan.outputs.has_changes == 'false')</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v3</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Setup Python</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-python@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">python-version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.11'</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Ansible</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">pip install ansible</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Setup SSH key</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">mkdir -p ~/.ssh</span>
          <span class="s">echo "${{ secrets.SSH_PRIVATE_KEY }}" &gt; ~/.ssh/id_rsa</span>
          <span class="s">chmod 600 ~/.ssh/id_rsa</span>
          <span class="s">ssh-keyscan -H ${{ secrets.SERVER_IP }} &gt;&gt; ~/.ssh/known_hosts</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run Ansible playbook</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">DOMAIN</span><span class="pi">:</span> <span class="s">${{ secrets.DOMAIN }}</span>
          <span class="na">ACME_EMAIL</span><span class="pi">:</span> <span class="s">${{ secrets.ACME_EMAIL }}</span>
          <span class="na">DB_NAME</span><span class="pi">:</span> <span class="s">${{ secrets.DB_NAME }}</span>
          <span class="na">DB_USER</span><span class="pi">:</span> <span class="s">${{ secrets.DB_USER }}</span>
          <span class="na">DB_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.DB_PASSWORD }}</span>
          <span class="na">JWT_SECRET</span><span class="pi">:</span> <span class="s">${{ secrets.JWT_SECRET }}</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">cd infra/ansible</span>
          <span class="s">ansible-playbook -i inventory/hosts playbook.yml</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Verify deployment</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">sleep 30  # Wait for services to stabilize</span>
          <span class="s">curl -f https://${{ secrets.DOMAIN }}/health || exit 1</span>
          <span class="s">echo "✅ Deployment verified!"</span>
</code></pre>

</div>



<h4>
  
  
  .github/workflows/application.yml
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Application Deployment</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">push</span><span class="pi">:</span>
    <span class="na">branches</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">main</span><span class="pi">]</span>
    <span class="na">paths</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">frontend/**'</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">auth-api/**'</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">todos-api/**'</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">users-api/**'</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">log-processor/**'</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">docker-compose.yml'</span>
  <span class="na">workflow_dispatch</span><span class="pi">:</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">deploy</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy Application</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>

    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v3</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Setup SSH key</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">mkdir -p ~/.ssh</span>
          <span class="s">echo "${{ secrets.SSH_PRIVATE_KEY }}" &gt; ~/.ssh/id_rsa</span>
          <span class="s">chmod 600 ~/.ssh/id_rsa</span>
          <span class="s">ssh-keyscan -H ${{ secrets.SERVER_IP }} &gt;&gt; ~/.ssh/known_hosts</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Deploy to server</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">ssh ubuntu@${{ secrets.SERVER_IP }} &lt;&lt; 'EOF'</span>
            <span class="s">cd /opt/todo-app</span>
            <span class="s">git pull origin main</span>
            <span class="s">docker-compose pull</span>
            <span class="s">docker-compose up -d --build</span>
          <span class="s">EOF</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Wait for deployment</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">sleep </span><span class="m">30</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Health check</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">curl -f https://${{ secrets.DOMAIN }}/health || exit 1</span>
          <span class="s">echo "✅ Application deployed successfully!"</span>
</code></pre>

</div>



<h3>
  
  
  Understanding the CI/CD Flow
</h3>

<p><strong>Infrastructure changes (Terraform/Ansible):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Push to main
   ↓
2. Run terraform plan
   ↓
3. Detect drift? → Send email
   ↓
4. Pause for manual approval (GitHub Environment protection)
   ↓
5. Apply changes
   ↓
6. Run Ansible
   ↓
7. Verify deployment
</code></pre>

</div>



<p><strong>Application changes:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Push to main
   ↓
2. SSH to server
   ↓
3. Git pull
   ↓
4. docker-compose pull
   ↓
5. docker-compose up
   ↓
6. Health check
</code></pre>

</div>



<h3>
  
  
  Testing the Complete Setup
</h3>

<h4>
  
  
  Local Testing
</h4>

<p><strong>1. Test containers locally:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start everything</span>
docker-compose up <span class="nt">-d</span>

<span class="c"># Check status</span>
docker-compose ps

<span class="c"># View logs</span>
docker-compose logs <span class="nt">-f</span>

<span class="c"># Test frontend</span>
curl http://localhost

<span class="c"># Test APIs</span>
curl http://localhost/api/auth/health
curl http://localhost/api/todos/health
curl http://localhost/api/users/health

<span class="c"># Stop everything</span>
docker-compose down
</code></pre>

</div>



<p><strong>2. Test Terraform:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>infra/terraform

<span class="c"># Initialize</span>
terraform init

<span class="c"># Validate</span>
terraform validate

<span class="c"># Plan (dry run)</span>
terraform plan

<span class="c"># Apply (create infrastructure)</span>
terraform apply

<span class="c"># Show outputs</span>
terraform output

<span class="c"># Destroy (cleanup)</span>
terraform destroy
</code></pre>

</div>



<p><strong>3. Test Ansible:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>infra/ansible

<span class="c"># Test connection</span>
ansible all <span class="nt">-m</span> ping

<span class="c"># Check syntax</span>
ansible-playbook playbook.yml <span class="nt">--syntax-check</span>

<span class="c"># Dry run</span>
ansible-playbook playbook.yml <span class="nt">--check</span>

<span class="c"># Run for real</span>
ansible-playbook playbook.yml

<span class="c"># Run specific role</span>
ansible-playbook playbook.yml <span class="nt">--tags</span> deploy
</code></pre>

</div>



<h4>
  
  
  Production Deployment
</h4>

<p><strong>Complete deployment from scratch:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Clone the repository</span>
git clone https://github.com/yourusername/todo-app.git
<span class="nb">cd </span>todo-app

<span class="c"># 2. Configure secrets</span>
<span class="nb">cp</span> .env.example .env
<span class="c"># Edit .env with your values</span>

<span class="c"># 3. Initialize Terraform</span>
<span class="nb">cd </span>infra/terraform
terraform init

<span class="c"># 4. Create infrastructure</span>
terraform plan
terraform apply

<span class="c"># Wait for Ansible to complete (triggered automatically)</span>

<span class="c"># 5. Configure DNS</span>
<span class="c"># Point your domain to the Elastic IP shown in terraform outputs</span>

<span class="c"># 6. Verify deployment</span>
curl https://your-domain.com
</code></pre>

</div>



<p><strong>Expected result:</strong></p>

<ul>
<li>Login page loads at <a href="https://your-domain.com" rel="noopener noreferrer">https://your-domain.com</a>
</li>
<li>HTTPS works (automatic certificate from Let's Encrypt)</li>
<li>APIs respond at /api/auth, /api/todos, /api/users</li>
</ul>

<h3>
  
  
  Troubleshooting
</h3>

<h4>
  
  
  Issue: Terraform fails with "state locked"
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check lock info</span>
terraform force-unlock &lt;LOCK_ID&gt;

<span class="c"># Or wait for other operation to complete</span>
</code></pre>

</div>



<h4>
  
  
  Issue: Ansible can't connect to server
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Test SSH manually</span>
ssh <span class="nt">-i</span> ~/.ssh/id_rsa ubuntu@&lt;SERVER_IP&gt;

<span class="c"># Check inventory</span>
ansible-inventory <span class="nt">--list</span> <span class="nt">-i</span> inventory/hosts

<span class="c"># Verbose output</span>
ansible-playbook playbook.yml <span class="nt">-vvv</span>
</code></pre>

</div>



<h4>
  
  
  Issue: Containers won't start
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check logs</span>
docker-compose logs &lt;service-name&gt;

<span class="c"># Check disk space</span>
<span class="nb">df</span> <span class="nt">-h</span>

<span class="c"># Check memory</span>
free <span class="nt">-h</span>

<span class="c"># Restart specific service</span>
docker-compose restart &lt;service-name&gt;
</code></pre>

</div>



<h4>
  
  
  Issue: HTTPS not working
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check Traefik logs</span>
docker logs traefik

<span class="c"># Verify DNS points to server</span>
dig your-domain.com

<span class="c"># Check certificate</span>
docker <span class="nb">exec </span>traefik <span class="nb">cat</span> /letsencrypt/acme.json

<span class="c"># Force certificate renewal</span>
docker-compose down
<span class="nb">rm</span> <span class="nt">-rf</span> letsencrypt/acme.json
docker-compose up <span class="nt">-d</span>
</code></pre>

</div>



