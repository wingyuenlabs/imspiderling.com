---
Title: From Code to Cloud: Seamless CI/CD with GitHub Actions and Azure Web Apps
Description: 
Author: Ibrahim Bio Abubakar
Date: 2025-09-13T21:39:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>Have you ever built a cool Node.js app on your local machine and thought, <em>“Now what? How do I get this running in the cloud without manually pushing code every time?”</em> That’s exactly where CI/CD (Continuous Integration and Continuous Deployment) comes to the rescue.</p>

<p>In this article, I’ll walk you through how I took a simple Node.js application <strong>from code to cloud</strong> using <strong>GitHub Actions, Docker, Kubernetes, and Azure Web App Services</strong>. We’ll set up a seamless pipeline that automatically builds, tests, and deploys changes the moment you push to GitHub—no more manual deployments, no more “it works on my machine” headaches.</p>

<p>By the end, you’ll see how easy it is to:</p>

<ul>
<li>Containerize your Node.js app with Docker 🐳</li>
<li>Orchestrate deployments with Kubernetes ⚙️</li>
<li>Automate CI/CD using GitHub Actions 🤖</li>
<li>Deploy effortlessly to Azure Web Apps ☁️</li>
</ul>

<p>Let’s dive in and turn your local project into a production-ready cloud app!</p>




<h2>
  
  
  Prerequisites - Download These First!
</h2>

<p>Before starting, you need to install these tools on your machine:</p>

<ul>
<li>
<strong>Node.js (v18 or higher)</strong> – <a href="https://nodejs.org/" rel="noopener noreferrer">Download</a> (choose LTS version 20.x)
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   node <span class="nt">--version</span>
   npm <span class="nt">--version</span>
</code></pre>

</div>



<ul>
<li>
<strong>Git</strong> – <a href="https://git-scm.com/downloads" rel="noopener noreferrer">Download</a>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   git <span class="nt">--version</span>
</code></pre>

</div>



<ul>
<li>
<strong>Docker Desktop</strong> – <a href="https://www.docker.com/products/docker-desktop/" rel="noopener noreferrer">Download</a>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   docker <span class="nt">--version</span>
   docker-compose <span class="nt">--version</span>
</code></pre>

</div>



<ul>
<li>
<strong>GitHub Account</strong> – <a href="https://github.com" rel="noopener noreferrer">Sign up here</a>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr77px5u4hroe40x8vnoq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr77px5u4hroe40x8vnoq.png" alt="Image 1" width="800" height="279"></a></p>

<ul>
<li>
<strong>Code Editor (optional, recommended)</strong> – <a href="https://code.visualstudio.com/" rel="noopener noreferrer">VS Code</a>
</li>
</ul>




<h2>
  
  
  Step 1: Set Up Git for Version Control
</h2>

<p>Configure Git:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git config <span class="nt">--global</span> user.name <span class="s2">"Your Name"</span>
git config <span class="nt">--global</span> user.email <span class="s2">"you@example.com"</span>
git config <span class="nt">--global</span> init.defaultBranch main
</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdbquadaqmmgdmkgjykkb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdbquadaqmmgdmkgjykkb.png" alt="Image 2" width="800" height="246"></a></p>

<p>Initialize your project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>my-devops-project
<span class="nb">cd </span>my-devops-project
git init
</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftkmiz002pkuvw9ow4a68.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftkmiz002pkuvw9ow4a68.png" alt="Image 3" width="800" height="194"></a></p>




<h2>
  
  
  Step 2: Build a Node.js Web App
</h2>

<p>Initialize project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create package.json with default settings</span>
npm init <span class="nt">-y</span>
</code></pre>

</div>



<p>Output of command in VScode terminal:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz3wosz8o4m5krml97k28.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz3wosz8o4m5krml97k28.png" alt="Image 4" width="800" height="281"></a></p>

<p>Update <strong>package.json</strong> with scripts, metadata, and dev tools (<code>Jest</code>, <code>ESLint</code>, <code>Supertest</code>).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">{</span>
  <span class="s2">"name"</span>: <span class="s2">"my-devops-project"</span>,
  <span class="s2">"version"</span>: <span class="s2">"1.0.0"</span>,
  <span class="s2">"description"</span>: <span class="s2">"DevOps learning project with Node.js"</span>,
  <span class="s2">"main"</span>: <span class="s2">"app.js"</span>,
  <span class="s2">"scripts"</span>: <span class="o">{</span>
    <span class="s2">"start"</span>: <span class="s2">"node app.js"</span>,
    <span class="s2">"test"</span>: <span class="s2">"jest"</span>,
    <span class="s2">"dev"</span>: <span class="s2">"node app.js"</span>,
    <span class="s2">"lint"</span>: <span class="s2">"eslint ."</span>
  <span class="o">}</span>,
  <span class="s2">"keywords"</span>: <span class="o">[</span><span class="s2">"devops"</span>, <span class="s2">"nodejs"</span>, <span class="s2">"docker"</span><span class="o">]</span>,
  <span class="s2">"author"</span>: <span class="s2">"Your Name"</span>,
  <span class="s2">"license"</span>: <span class="s2">"MIT"</span>,
  <span class="s2">"engines"</span>: <span class="o">{</span>
    <span class="s2">"node"</span>: <span class="s2">"&gt;=18.0.0"</span>
  <span class="o">}</span>,
  <span class="s2">"devDependencies"</span>: <span class="o">{</span>
    <span class="s2">"jest"</span>: <span class="s2">"^29.7.0"</span>,
    <span class="s2">"eslint"</span>: <span class="s2">"^8.57.0"</span>,
    <span class="s2">"supertest"</span>: <span class="s2">"^7.1.4"</span>
  <span class="o">}</span>
<span class="o">}</span>

</code></pre>

</div>



<p>Then create <code>app.js</code> with routes for <code>/</code>, <code>/health</code>, <code>/info</code>, <code>/metrics</code>, and graceful shutdown logic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Creates file</span>

<span class="nb">touch </span>app.js

</code></pre>

</div>



<p>Copy and paste this into <code>app.js</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>// core modules
const http <span class="o">=</span> require<span class="o">(</span><span class="s2">"http"</span><span class="o">)</span><span class="p">;</span>
const url <span class="o">=</span> require<span class="o">(</span><span class="s2">"url"</span><span class="o">)</span><span class="p">;</span>

// environment configuration
const PORT <span class="o">=</span> process.env.PORT <span class="o">||</span> 3000<span class="p">;</span>
const ENVIRONMENT <span class="o">=</span> process.env.NODE_ENV <span class="o">||</span> <span class="s2">"development"</span><span class="p">;</span>

<span class="nb">let </span>requestCount <span class="o">=</span> 0<span class="p">;</span>

// helper: send JSON responses
<span class="k">function </span>sendJSON<span class="o">(</span>res, statusCode, data<span class="o">)</span> <span class="o">{</span>
  res.statusCode <span class="o">=</span> statusCode<span class="p">;</span>
  res.setHeader<span class="o">(</span><span class="s2">"Content-Type"</span>, <span class="s2">"application/json"</span><span class="o">)</span><span class="p">;</span>
  res.end<span class="o">(</span>JSON.stringify<span class="o">(</span>data, null, 2<span class="o">))</span><span class="p">;</span>
<span class="o">}</span>

// helper: send HTML responses
<span class="k">function </span>sendHTML<span class="o">(</span>res, statusCode, content<span class="o">)</span> <span class="o">{</span>
  res.statusCode <span class="o">=</span> statusCode<span class="p">;</span>
  res.setHeader<span class="o">(</span><span class="s2">"Content-Type"</span>, <span class="s2">"text/html"</span><span class="o">)</span><span class="p">;</span>
  res.end<span class="o">(</span>content<span class="o">)</span><span class="p">;</span>
<span class="o">}</span>

// helper: send Prometheus metrics
<span class="k">function </span>sendMetrics<span class="o">(</span>res<span class="o">)</span> <span class="o">{</span>
  const mem <span class="o">=</span> process.memoryUsage<span class="o">()</span><span class="p">;</span>
  const metrics <span class="o">=</span> <span class="sb">`</span>
<span class="c"># HELP http_requests_total Total HTTP requests</span>
<span class="c"># TYPE http_requests_total counter</span>
http_requests_total <span class="k">${</span><span class="nv">requestCount</span><span class="k">}</span>

<span class="c"># HELP app_uptime_seconds Application uptime in seconds</span>
<span class="c"># TYPE app_uptime_seconds gauge</span>
app_uptime_seconds <span class="k">${</span><span class="nv">process</span><span class="p">.uptime()</span><span class="k">}</span>

<span class="c"># HELP nodejs_memory_usage_bytes Node.js memory usage</span>
<span class="c"># TYPE nodejs_memory_usage_bytes gauge</span>
nodejs_memory_usage_bytes<span class="o">{</span><span class="nb">type</span><span class="o">=</span><span class="s2">"rss"</span><span class="o">}</span> <span class="k">${</span><span class="nv">mem</span><span class="p">.rss</span><span class="k">}</span>
nodejs_memory_usage_bytes<span class="o">{</span><span class="nb">type</span><span class="o">=</span><span class="s2">"heapUsed"</span><span class="o">}</span> <span class="k">${</span><span class="nv">mem</span><span class="p">.heapUsed</span><span class="k">}</span>
nodejs_memory_usage_bytes<span class="o">{</span><span class="nb">type</span><span class="o">=</span><span class="s2">"heapTotal"</span><span class="o">}</span> <span class="k">${</span><span class="nv">mem</span><span class="p">.heapTotal</span><span class="k">}</span>
nodejs_memory_usage_bytes<span class="o">{</span><span class="nb">type</span><span class="o">=</span><span class="s2">"external"</span><span class="o">}</span> <span class="k">${</span><span class="nv">mem</span><span class="p">.external</span><span class="k">}</span>
<span class="sb">`</span><span class="p">;</span>
  res.statusCode <span class="o">=</span> 200<span class="p">;</span>
  res.setHeader<span class="o">(</span><span class="s2">"Content-Type"</span>, <span class="s2">"text/plain"</span><span class="o">)</span><span class="p">;</span>
  res.end<span class="o">(</span>metrics<span class="o">)</span><span class="p">;</span>
<span class="o">}</span>

// main server
const server <span class="o">=</span> http.createServer<span class="o">((</span>req, res<span class="o">)</span> <span class="o">=&gt;</span> <span class="o">{</span>
  requestCount++<span class="p">;</span>
  const timestamp <span class="o">=</span> new Date<span class="o">()</span>.toISOString<span class="o">()</span><span class="p">;</span>
  const <span class="o">{</span> pathname <span class="o">}</span> <span class="o">=</span> url.parse<span class="o">(</span>req.url, <span class="nb">true</span><span class="o">)</span><span class="p">;</span>

  // logging
  console.log<span class="o">(</span>
    <span class="sb">`</span><span class="k">${</span><span class="nv">timestamp</span><span class="k">}</span> - <span class="k">${</span><span class="nv">req</span><span class="p">.method</span><span class="k">}</span> <span class="k">${</span><span class="nv">pathname</span><span class="k">}</span> - <span class="k">${</span><span class="p">
      req.headers[</span><span class="s2">"user-agent"</span><span class="p">] || </span><span class="s2">"Unknown"</span><span class="p">
    </span><span class="k">}</span><span class="sb">`</span>
  <span class="o">)</span><span class="p">;</span>

  // CORS headers
  res.setHeader<span class="o">(</span><span class="s2">"Access-Control-Allow-Origin"</span>, <span class="s2">"*"</span><span class="o">)</span><span class="p">;</span>
  res.setHeader<span class="o">(</span><span class="s2">"Access-Control-Allow-Methods"</span>, <span class="s2">"GET, POST, PUT, DELETE"</span><span class="o">)</span><span class="p">;</span>
  res.setHeader<span class="o">(</span><span class="s2">"Access-Control-Allow-Headers"</span>, <span class="s2">"Content-Type"</span><span class="o">)</span><span class="p">;</span>

  // security headers
  res.setHeader<span class="o">(</span><span class="s2">"X-Content-Type-Options"</span>, <span class="s2">"nosniff"</span><span class="o">)</span><span class="p">;</span>
  res.setHeader<span class="o">(</span><span class="s2">"X-Frame-Options"</span>, <span class="s2">"DENY"</span><span class="o">)</span><span class="p">;</span>
  res.setHeader<span class="o">(</span><span class="s2">"X-XSS-Protection"</span>, <span class="s2">"1; mode=block"</span><span class="o">)</span><span class="p">;</span>

  // route handling
  switch <span class="o">(</span>pathname<span class="o">)</span> <span class="o">{</span>
    <span class="k">case</span> <span class="s2">"/"</span>:
      sendHTML<span class="o">(</span>
        res,
        200,
        <span class="sb">`</span>
&lt;<span class="o">!</span>DOCTYPE html&gt;
&lt;html&gt;
&lt;<span class="nb">head</span><span class="o">&gt;</span>
  &lt;title&gt;DevOps Lab 2025&lt;/title&gt;
  &lt;style&gt;
    body <span class="o">{</span> font-family: Arial, sans-serif<span class="p">;</span> max-width: 800px<span class="p">;</span> margin: 50px auto<span class="p">;</span> padding: 20px<span class="p">;</span> <span class="o">}</span>
    .header <span class="o">{</span> background: linear-gradient<span class="o">(</span>135deg, <span class="c">#667eea 0%, #764ba2 100%); color: white; padding: 20px; border-radius: 8px; }</span>
    .endpoint <span class="o">{</span> background: <span class="c">#f8f9fa; padding: 15px; margin: 10px 0; border-radius: 5px; border-left: 4px solid #007bff; }</span>
  &lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;div <span class="nv">class</span><span class="o">=</span><span class="s2">"header"</span><span class="o">&gt;</span>
    &lt;h1&gt;Fullstack DevOps Lab 2025&lt;/h1&gt;
    &lt;p&gt;Modern Node.js application with CI/CD pipeline&lt;/p&gt;
    &lt;p&gt;Version: 1.0.0&lt;/p&gt;
  &lt;/div&gt;
  &lt;h2&gt;Available Endpoints:&lt;/h2&gt;
  &lt;div <span class="nv">class</span><span class="o">=</span><span class="s2">"endpoint"</span><span class="o">&gt;</span>&lt;strong&gt;GET /&lt;/strong&gt; - This welcome page&lt;/div&gt;
  &lt;div <span class="nv">class</span><span class="o">=</span><span class="s2">"endpoint"</span><span class="o">&gt;</span>&lt;strong&gt;GET /health&lt;/strong&gt; - Health check <span class="o">(</span>JSON<span class="o">)</span>&lt;/div&gt;
  &lt;div <span class="nv">class</span><span class="o">=</span><span class="s2">"endpoint"</span><span class="o">&gt;</span>&lt;strong&gt;GET /info&lt;/strong&gt; - System information&lt;/div&gt;
  &lt;div <span class="nv">class</span><span class="o">=</span><span class="s2">"endpoint"</span><span class="o">&gt;</span>&lt;strong&gt;GET /metrics&lt;/strong&gt; - Prometheus metrics&lt;/div&gt;
  &lt;p&gt;Environment: &lt;strong&gt;<span class="k">${</span><span class="nv">ENVIRONMENT</span><span class="k">}</span>&lt;/strong&gt;&lt;/p&gt;
  &lt;p&gt;Server <span class="nb">time</span>: &lt;strong&gt;<span class="k">${</span><span class="nv">timestamp</span><span class="k">}</span>&lt;/strong&gt;&lt;/p&gt;
  &lt;p&gt;Requests served: &lt;strong&gt;<span class="k">${</span><span class="nv">requestCount</span><span class="k">}</span>&lt;/strong&gt;&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;<span class="sb">`</span>
      <span class="p">);</span>
      <span class="nb">break</span><span class="p">;</span>

    <span class="k">case</span> <span class="s2">"/health"</span>:
      sendJSON<span class="o">(</span>res, 200, <span class="o">{</span>
        status: <span class="s2">"healthy"</span>,
        timestamp,
        <span class="nb">uptime</span>: process.uptime<span class="o">(</span><span class="p">)</span>,
        environment: ENVIRONMENT,
        version: <span class="s2">"1.0.0"</span>,
        node_version: process.version,
        requests_served: requestCount,
      <span class="o">})</span><span class="p">;</span>
      <span class="nb">break</span><span class="p">;</span>

    <span class="k">case</span> <span class="s2">"/info"</span>:
      sendJSON<span class="o">(</span>res, 200, <span class="o">{</span>
        platform: process.platform,
        architecture: process.arch,
        node_version: process.version,
        memory_usage: process.memoryUsage<span class="o">(</span><span class="p">)</span>,
        environment: ENVIRONMENT,
        pid: process.pid,
        <span class="nb">uptime</span>: process.uptime<span class="o">()</span>,
      <span class="o">})</span><span class="p">;</span>
      <span class="nb">break</span><span class="p">;</span>

    <span class="k">case</span> <span class="s2">"/metrics"</span>:
      sendMetrics<span class="o">(</span>res<span class="p">);</span>
      <span class="nb">break</span><span class="p">;</span>

    default:
      sendJSON<span class="o">(</span>res, 404, <span class="o">{</span>
        error: <span class="s2">"Not Found"</span>,
        message: <span class="sb">`</span>Route <span class="k">${</span><span class="nv">pathname</span><span class="k">}</span> not found<span class="sb">`</span>,
        timestamp,
      <span class="o">})</span><span class="p">;</span>
  <span class="o">}</span>
<span class="o">})</span><span class="p">;</span>

// graceful shutdown
<span class="k">function </span>shutdown<span class="o">(</span>signal<span class="o">)</span> <span class="o">{</span>
  console.log<span class="o">(</span><span class="sb">`</span><span class="se">\n</span>Received <span class="k">${</span><span class="nv">signal</span><span class="k">}</span>, shutting down gracefully...<span class="sb">`</span><span class="o">)</span><span class="p">;</span>
  server.close<span class="o">(()</span> <span class="o">=&gt;</span> <span class="o">{</span>
    console.log<span class="o">(</span><span class="s2">"Server closed"</span><span class="o">)</span><span class="p">;</span>
    process.exit<span class="o">(</span>0<span class="o">)</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>
<span class="o">}</span>
process.on<span class="o">(</span><span class="s2">"SIGTERM"</span>, <span class="o">()</span> <span class="o">=&gt;</span> shutdown<span class="o">(</span><span class="s2">"SIGTERM"</span><span class="o">))</span><span class="p">;</span>
process.on<span class="o">(</span><span class="s2">"SIGINT"</span>, <span class="o">()</span> <span class="o">=&gt;</span> shutdown<span class="o">(</span><span class="s2">"SIGINT"</span><span class="o">))</span><span class="p">;</span>

// start server
server.listen<span class="o">(</span>PORT, <span class="o">()</span> <span class="o">=&gt;</span> <span class="o">{</span>
  console.log<span class="o">(</span><span class="sb">`</span>🚀 Server running at http://localhost:<span class="k">${</span><span class="nv">PORT</span><span class="k">}</span>/<span class="sb">`</span><span class="o">)</span><span class="p">;</span>
  console.log<span class="o">(</span><span class="sb">`</span>Environment: <span class="k">${</span><span class="nv">ENVIRONMENT</span><span class="k">}</span><span class="sb">`</span><span class="o">)</span><span class="p">;</span>
  console.log<span class="o">(</span><span class="sb">`</span>Node.js version: <span class="k">${</span><span class="nv">process</span><span class="p">.version</span><span class="k">}</span><span class="sb">`</span><span class="o">)</span><span class="p">;</span>
<span class="o">})</span><span class="p">;</span>

// <span class="nb">export </span><span class="k">for </span>testing
module.exports <span class="o">=</span> server<span class="p">;</span>

</code></pre>

</div>



<p>Install dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install testing and dev tools</span>
npm <span class="nb">install</span> <span class="nt">--save-dev</span> jest eslint supertest

<span class="c"># Install all dependencies</span>

npm <span class="nb">install</span>
</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzfcbd6x3hhj2wjf9rbng.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzfcbd6x3hhj2wjf9rbng.png" alt="Image 6" width="800" height="230"></a></p>

<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flezhn1rwfk5tbw8r51p4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flezhn1rwfk5tbw8r51p4.png" alt="Image 7" width="800" height="263"></a></p>




<h2>
  
  
  Step 3: Create Proper Tests
</h2>

<p>We are going to setup an automated testing that verify the application works correctly everytime a change is made. </p>

<p>Create a test directory and a test file <code>tests/app.test.js</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create a folder for your tests</span>
<span class="nb">mkdir </span>tests

<span class="c"># Create the main test file</span>
<span class="nb">touch </span>tests/app.test.js

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fii67dj88m58h9me2wr93.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fii67dj88m58h9me2wr93.png" alt="Image 8" width="800" height="186"></a></p>

<p>Copy and paste the code below into <code>tests/app.test.js</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
const request <span class="o">=</span> require<span class="o">(</span><span class="s1">'supertest'</span><span class="o">)</span><span class="p">;</span>
const server <span class="o">=</span> require<span class="o">(</span><span class="s1">'../app'</span><span class="o">)</span><span class="p">;</span>

describe<span class="o">(</span><span class="s1">'App Endpoints'</span>, <span class="o">()</span> <span class="o">=&gt;</span> <span class="o">{</span>
  afterAll<span class="o">(()</span> <span class="o">=&gt;</span> <span class="o">{</span>
    server.close<span class="o">()</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>

  <span class="nb">test</span><span class="o">(</span><span class="s1">'GET / should return welcome page'</span>, async <span class="o">()</span> <span class="o">=&gt;</span> <span class="o">{</span>
    const response <span class="o">=</span> await request<span class="o">(</span>server<span class="o">)</span>.get<span class="o">(</span><span class="s1">'/'</span><span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.status<span class="o">)</span>.toBe<span class="o">(</span>200<span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.text<span class="o">)</span>.toContain<span class="o">(</span><span class="s1">'DevOps Lab 2025'</span><span class="o">)</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>

  <span class="nb">test</span><span class="o">(</span><span class="s1">'GET /health should return health status'</span>, async <span class="o">()</span> <span class="o">=&gt;</span> <span class="o">{</span>
    const response <span class="o">=</span> await request<span class="o">(</span>server<span class="o">)</span>.get<span class="o">(</span><span class="s1">'/health'</span><span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.status<span class="o">)</span>.toBe<span class="o">(</span>200<span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.body.status<span class="o">)</span>.toBe<span class="o">(</span><span class="s1">'healthy'</span><span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.body.timestamp<span class="o">)</span>.toBeDefined<span class="o">()</span><span class="p">;</span>
    expect<span class="o">(</span>typeof response.body.uptime<span class="o">)</span>.toBe<span class="o">(</span><span class="s1">'number'</span><span class="o">)</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>

  <span class="nb">test</span><span class="o">(</span><span class="s1">'GET /info should return system info'</span>, async <span class="o">()</span> <span class="o">=&gt;</span> <span class="o">{</span>
    const response <span class="o">=</span> await request<span class="o">(</span>server<span class="o">)</span>.get<span class="o">(</span><span class="s1">'/info'</span><span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.status<span class="o">)</span>.toBe<span class="o">(</span>200<span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.body.platform<span class="o">)</span>.toBeDefined<span class="o">()</span><span class="p">;</span>
    expect<span class="o">(</span>response.body.node_version<span class="o">)</span>.toBeDefined<span class="o">()</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>

  <span class="nb">test</span><span class="o">(</span><span class="s1">'GET /metrics should return prometheus metrics'</span>, async <span class="o">()</span> <span class="o">=&gt;</span> <span class="o">{</span>
    const response <span class="o">=</span> await request<span class="o">(</span>server<span class="o">)</span>.get<span class="o">(</span><span class="s1">'/metrics'</span><span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.status<span class="o">)</span>.toBe<span class="o">(</span>200<span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.text<span class="o">)</span>.toContain<span class="o">(</span><span class="s1">'http_requests_total'</span><span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.text<span class="o">)</span>.toContain<span class="o">(</span><span class="s1">'app_uptime_seconds'</span><span class="o">)</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>

  <span class="nb">test</span><span class="o">(</span><span class="s1">'GET /nonexistent should return 404'</span>, async <span class="o">()</span> <span class="o">=&gt;</span> <span class="o">{</span>
    const response <span class="o">=</span> await request<span class="o">(</span>server<span class="o">)</span>.get<span class="o">(</span><span class="s1">'/nonexistent'</span><span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.status<span class="o">)</span>.toBe<span class="o">(</span>404<span class="o">)</span><span class="p">;</span>
    expect<span class="o">(</span>response.body.error<span class="o">)</span>.toBe<span class="o">(</span><span class="s1">'Not Found'</span><span class="o">)</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>
<span class="o">})</span><span class="p">;</span>

</code></pre>

</div>



<p><strong>Create Jest Configuration</strong></p>

<p>Jest is a JavaScript testing framework developed by Facebook (now Meta). It’s widely used in Node.js and React projects for testing code.</p>

<p>Create a <code>jest.config.js</code> file in the root folder  for configuration.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="nb">touch </span>jest.config.js

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp29nsrklkt8jj3e1xs4i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp29nsrklkt8jj3e1xs4i.png" alt="Image 9" width="800" height="204"></a></p>

<p>Copy and paste the code below in the <code>jest.config.js</code> file<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
module.exports <span class="o">=</span> <span class="o">{</span>
  testEnvironment: <span class="s1">'node'</span>,
  collectCoverage: <span class="nb">true</span>,
  coverageDirectory: <span class="s1">'coverage'</span>,
  testMatch: <span class="o">[</span><span class="s1">'**/tests/**/*.test.js'</span><span class="o">]</span>,
  verbose: <span class="nb">true</span>
<span class="o">}</span><span class="p">;</span>

</code></pre>

</div>






<h2>
  
  
  Step 4: GitHub Actions CI/CD Pipeline
</h2>

<p>This creates an automated pipeline that runs tests and builds Docker images every time you push code to GitHub.</p>

<p>Create <code>.github/workflows/ci.yml</code> to:</p>

<ul>
<li>Run tests and linting</li>
<li>Build &amp; push Docker images to GitHub Container Registry</li>
<li>Run vulnerability scans</li>
<li>Deploy to staging/production based on branch
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create the GitHub Actions directory structure</span>
<span class="nb">mkdir</span> <span class="nt">-p</span> .github/workflows

<span class="c"># Create a ci.yml in .github/workflows directory</span>
<span class="nb">touch</span> .github/workflows/ci.yml

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgjh5ia2lvxbk23luux96.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgjh5ia2lvxbk23luux96.png" alt="Image 10" width="800" height="187"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>name: CI Pipeline

on:
  push:
    branches: <span class="o">[</span> main, develop <span class="o">]</span>
  pull_request:
    branches: <span class="o">[</span> main <span class="o">]</span>

<span class="nb">env</span>:
  NODE_VERSION: <span class="s1">'20'</span>
  REGISTRY: ghcr.io
  IMAGE_NAME: <span class="k">${</span><span class="p">{ github.repository </span><span class="k">}</span><span class="o">}</span>

<span class="nb">jobs</span>:
  <span class="nb">test</span>:
    name: Run Tests
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: <span class="o">[</span>18, 20]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js <span class="k">${</span><span class="p">{ matrix.node-version </span><span class="k">}</span><span class="o">}</span>
        uses: actions/setup-node@v4
        with:
          node-version: <span class="k">${</span><span class="p">{ matrix.node-version </span><span class="k">}</span><span class="o">}</span>
          cache: <span class="s1">'npm'</span>

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npx eslint <span class="nb">.</span> <span class="nt">--ext</span> .js <span class="nt">--ignore-pattern</span> node_modules/
        <span class="k">continue</span><span class="nt">-on-error</span>: <span class="nb">true</span>

      - name: Run tests
        run: npm <span class="nb">test</span>

      - name: Run security audit
        run: npm audit <span class="nt">--audit-level</span><span class="o">=</span>moderate <span class="o">||</span> <span class="nb">true

  </span>build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: <span class="nb">test
    </span><span class="k">if</span>: github.event_name <span class="o">==</span> <span class="s1">'push'</span>

    permissions:
      contents: <span class="nb">read
      </span>packages: write

    outputs:
      image-digest: <span class="k">${</span><span class="p">{ steps.build.outputs.digest </span><span class="k">}</span><span class="o">}</span>
      image-tag: <span class="k">${</span><span class="p">{ steps.meta.outputs.tags </span><span class="k">}</span><span class="o">}</span>

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log <span class="k">in </span>to Container Registry
        uses: docker/login-action@v3
        with:
          registry: <span class="k">${</span><span class="p">{ env.REGISTRY </span><span class="k">}</span><span class="o">}</span>
          username: <span class="k">${</span><span class="p">{ github.actor </span><span class="k">}</span><span class="o">}</span>
          password: <span class="k">${</span><span class="p">{ secrets.GITHUB_TOKEN </span><span class="k">}</span><span class="o">}</span>

      - name: Extract metadata
        <span class="nb">id</span>: meta
        uses: docker/metadata-action@v5
        with:
          images: <span class="k">${</span><span class="p">{ env.REGISTRY </span><span class="k">}</span><span class="o">}</span>/<span class="k">${</span><span class="p">{ env.IMAGE_NAME </span><span class="k">}</span><span class="o">}</span>
          tags: |
            <span class="nb">type</span><span class="o">=</span>ref,event<span class="o">=</span>branch
            <span class="nb">type</span><span class="o">=</span>sha,prefix<span class="o">={{</span>branch<span class="o">}}</span>-
            <span class="nb">type</span><span class="o">=</span>raw,value<span class="o">=</span>latest,enable<span class="o">={{</span>is_default_branch<span class="o">}}</span>

      - name: Build and push Docker image
        <span class="nb">id</span>: build
        uses: docker/build-push-action@v5
        with:
          context: <span class="nb">.</span>
          platforms: linux/amd64,linux/arm64
          push: <span class="nb">true
          </span>tags: <span class="k">${</span><span class="p">{ steps.meta.outputs.tags </span><span class="k">}</span><span class="o">}</span>
          labels: <span class="k">${</span><span class="p">{ steps.meta.outputs.labels </span><span class="k">}</span><span class="o">}</span>
          cache-from: <span class="nb">type</span><span class="o">=</span>gha
          cache-to: <span class="nb">type</span><span class="o">=</span>gha,mode<span class="o">=</span>max

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    <span class="k">if</span>: github.ref <span class="o">==</span> <span class="s1">'refs/heads/develop'</span>
    environment: staging

    steps:
      - name: Deploy to staging
        run: |
          <span class="nb">echo</span> <span class="s2">"🚀 Deploying to staging environment..."</span>
          <span class="nb">echo</span> <span class="s2">"Image: </span><span class="k">${</span><span class="p">{ needs.build.outputs.image-tag </span><span class="k">}</span><span class="s2">}"</span>
          <span class="nb">echo</span> <span class="s2">"Would deploy to staging server here"</span>
          <span class="c"># In real scenario, you'd use:</span>
          <span class="c"># - kubectl apply -f k8s/staging/</span>
          <span class="c"># - docker-compose -f docker-compose.staging.yml up -d</span>
          <span class="c"># - ssh staging-server "docker pull $IMAGE &amp;&amp; docker-compose up -d"</span>

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    <span class="k">if</span>: github.ref <span class="o">==</span> <span class="s1">'refs/heads/main'</span>
    environment: production

    steps:
      - name: Deploy to production
        run: |
          <span class="nb">echo</span> <span class="s2">"🎯 Deploying to production environment..."</span>
          <span class="nb">echo</span> <span class="s2">"Image: </span><span class="k">${</span><span class="p">{ needs.build.outputs.image-tag </span><span class="k">}</span><span class="s2">}"</span>
          <span class="nb">echo</span> <span class="s2">"Image digest: </span><span class="k">${</span><span class="p">{ needs.build.outputs.image-digest </span><span class="k">}</span><span class="s2">}"</span>
          <span class="nb">echo</span> <span class="s2">"Would deploy to production server here"</span>
          <span class="c"># In real scenario, you'd use:</span>
          <span class="c"># - kubectl apply -f k8s/production/</span>
          <span class="c"># - terraform apply</span>
          <span class="c"># - ansible-playbook deploy.yml</span>

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: build
    <span class="k">if</span>: github.event_name <span class="o">==</span> <span class="s1">'push'</span> <span class="o">&amp;&amp;</span> github.ref <span class="o">==</span> <span class="s1">'refs/heads/main'</span>

    steps:
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: <span class="s1">'ghcr.io/YOU_GITHUB_USERNAME/my-devops-project:latest'</span>  <span class="c">#All in lower case.  </span>
          format: <span class="s1">'sarif'</span>
          output: <span class="s1">'trivy-results.sarif'</span>
    <span class="c"># env:</span>
    <span class="c">#   TRIVY_USERNAME: ${{ github.actor }}</span>
    <span class="c">#   TRIVY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}</span>
</code></pre>

</div>



<p>Note: You'll need to replace <code>YOU_GITHUB_USERNAME</code> with your <strong>GitHub username</strong>.  </p>




<h2>
  
  
  Step 5: Dockerfile
</h2>

<p>This creates instructions for Docker to build a container image of your application that can run on any machine.</p>

<p>Create a <strong>multi-stage Dockerfile</strong> that:</p>

<ul>
<li>Installs dependencies</li>
<li>Uses non-root user for security</li>
<li>Adds health checks</li>
<li>Exposes port 3000
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#Create a Dockerfile</span>

<span class="nb">touch </span>Dockerfile

</code></pre>

</div>



<p>Copy and paste the codes below into <code>Dockerfile</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Multi-stage build for optimized image</span>
FROM node:20-alpine AS dependencies

<span class="c"># Update packages for security</span>
RUN apk update <span class="o">&amp;&amp;</span> apk upgrade <span class="nt">--no-cache</span>

WORKDIR /app

<span class="c"># Copy package files first for better caching</span>
COPY package<span class="k">*</span>.json ./

<span class="c"># Install only production dependencies</span>
RUN npm ci <span class="nt">--only</span><span class="o">=</span>production <span class="o">&amp;&amp;</span> npm cache clean <span class="nt">--force</span>

<span class="c"># Production stage  </span>
FROM node:20-alpine AS production

<span class="c"># Update packages and install necessary tools</span>
RUN apk update <span class="o">&amp;&amp;</span> apk upgrade <span class="nt">--no-cache</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
    apk add <span class="nt">--no-cache</span> curl dumb-init <span class="o">&amp;&amp;</span> <span class="se">\</span>
    <span class="nb">rm</span> <span class="nt">-rf</span> /var/cache/apk/<span class="k">*</span>

<span class="c"># Create non-root user with proper permissions</span>
RUN addgroup <span class="nt">-g</span> 1001 <span class="nt">-S</span> nodejs <span class="o">&amp;&amp;</span> <span class="se">\</span>
    adduser <span class="nt">-S</span> nodeuser <span class="nt">-u</span> 1001 <span class="nt">-G</span> nodejs

WORKDIR /app

<span class="c"># Copy dependencies from previous stage with proper ownership</span>
COPY <span class="nt">--from</span><span class="o">=</span>dependencies <span class="nt">--chown</span><span class="o">=</span>nodeuser:nodejs /app/node_modules ./node_modules

<span class="c"># Copy application code with proper ownership</span>
COPY <span class="nt">--chown</span><span class="o">=</span>nodeuser:nodejs package<span class="k">*</span>.json ./
COPY <span class="nt">--chown</span><span class="o">=</span>nodeuser:nodejs app.js ./

<span class="c"># Switch to non-root user</span>
USER nodeuser

<span class="c"># Expose port</span>
EXPOSE 3000

<span class="c"># Health check with proper timing for Node.js startup</span>
HEALTHCHECK <span class="nt">--interval</span><span class="o">=</span>30s <span class="nt">--timeout</span><span class="o">=</span>10s <span class="nt">--start-period</span><span class="o">=</span>15s <span class="nt">--retries</span><span class="o">=</span>3 <span class="se">\</span>
  CMD curl <span class="nt">-f</span> http://localhost:3000/health <span class="o">||</span> <span class="nb">exit </span>1

<span class="c"># Use dumb-init for proper signal handling in containers</span>
ENTRYPOINT <span class="o">[</span><span class="s2">"dumb-init"</span>, <span class="s2">"--"</span><span class="o">]</span>

<span class="c"># Start application</span>
CMD <span class="o">[</span><span class="s2">"npm"</span>, <span class="s2">"start"</span><span class="o">]</span>

</code></pre>

</div>






<h2>
  
  
  Step 6: Essential Configuration Files
</h2>

<p>Lets creates configuration files that tell various tools what to ignore, how to behave, and what settings to use.</p>

<p>Create the following:</p>

<ul>
<li><code>.dockerignore</code></li>
<li><code>.gitignore</code></li>
<li><code>.env.example</code></li>
<li><code>.eslintrc.js</code></li>
</ul>

<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F73fiozspoisj1ekwir7f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F73fiozspoisj1ekwir7f.png" alt="Image 11" width="800" height="234"></a></p>

<p>Create <code>.dockerignore</code>, copy and paste the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
node_modules 
npm-debug
.log<span class="k">*</span> 
.git 
.github 
.env 
.env.local 
.env.<span class="k">*</span>.local 
logs 
<span class="k">*</span>.log 
coverage 
.nyc_output 
.vscode 
.idea 
<span class="k">*</span>.swp 
<span class="k">*</span>.swo 
.DS_Store 
Thumbs.db 
README.md 
tests/ 
jest.config.js 
.eslintrc<span class="k">*</span>

</code></pre>

</div>



<p>Create <code>.gitignore</code>, copy and paste the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Dependencies</span>
node_modules/
npm-debug.log<span class="k">*</span>

<span class="c"># Runtime data</span>
pids
<span class="k">*</span>.pid
<span class="k">*</span>.seed
<span class="k">*</span>.pid.lock

<span class="c"># Coverage</span>
coverage/
.nyc_output

<span class="c"># Environment variables</span>
.env
.env.local
.env.<span class="k">*</span>.local

<span class="c"># Logs</span>
logs
<span class="k">*</span>.log

<span class="c"># IDE</span>
.vscode/
.idea/
<span class="k">*</span>.swp
<span class="k">*</span>.swo

<span class="c"># OS</span>
.DS_Store
Thumbs.db

</code></pre>

</div>



<p>Create <code>.env.example</code>, copy and paste the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="c"># Server </span>
Configuration <span class="nv">PORT</span><span class="o">=</span>3000 
<span class="nv">NODE_ENV</span><span class="o">=</span>production  
<span class="c"># Logging </span>
<span class="nv">LOG_LEVEL</span><span class="o">=</span>info

</code></pre>

</div>



<p>Create <code>.eslintrc.js</code>, copy and paste the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
module.exports <span class="o">=</span> <span class="o">{</span>
  <span class="nb">env</span>: <span class="o">{</span>
    node: <span class="nb">true</span>,
    es2021: <span class="nb">true</span>,
    jest: <span class="nb">true</span>
  <span class="o">}</span>,
  extends: <span class="o">[</span><span class="s1">'eslint:recommended'</span><span class="o">]</span>,
  parserOptions: <span class="o">{</span>
    ecmaVersion: 12,
    sourceType: <span class="s1">'module'</span>
  <span class="o">}</span>,
  rules: <span class="o">{</span>
    <span class="s1">'no-console'</span>: <span class="s1">'off'</span>,
    <span class="s1">'no-unused-vars'</span>: <span class="o">[</span><span class="s1">'error'</span>, <span class="o">{</span> <span class="s1">'argsIgnorePattern'</span>: <span class="s1">'^_'</span> <span class="o">}]</span>
  <span class="o">}</span>
<span class="o">}</span><span class="p">;</span>

</code></pre>

</div>






<h2>
  
  
  Step 7: Docker Compose for Development
</h2>

<p>Now let's create a Docker Compose file that makes it easy to run your application and any supporting services with a single command.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create a docker-compose.yml file </span>

<span class="nb">touch </span>docker-compose.yml

</code></pre>

</div>



<p>Create <code>docker-compose.yml</code> to run the app with, copy and paste the code below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code>
<span class="na">version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.8'</span>

<span class="na">services</span><span class="pi">:</span>
  <span class="na">app</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span> <span class="s">.</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">3000:3000"</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">NODE_ENV=development</span>
      <span class="pi">-</span> <span class="s">PORT=3000</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">curl"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">-f"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">http://localhost:3000/health"</span><span class="pi">]</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">30s</span>
      <span class="na">timeout</span><span class="pi">:</span> <span class="s">10s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">3</span>
      <span class="na">start_period</span><span class="pi">:</span> <span class="s">10s</span>

</code></pre>

</div>






<h2>
  
  
  Step 8: Test Everything Locally
</h2>

<p>This demonstrates how to run and test your application locally before deploying it.</p>

<p>Run locally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run your test suite to make sure everything works</span>
npm <span class="nb">test</span>

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjh0ai2eywepjpyx51yqg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjh0ai2eywepjpyx51yqg.png" alt="Image 14a" width="800" height="350"></a></p>

<p>You should see all 5 tests pass successfully:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0odfug3jqi3jk8iar0ip.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0odfug3jqi3jk8iar0ip.png" alt="Image 14b" width="800" height="342"></a></p>

<p>This starts the app in your local environment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start the application server</span>
npm start
</code></pre>

</div>



<p>What you'll see:</p>

<ul>
<li>Server starts running at <a href="http://localhost:3000/" rel="noopener noreferrer">http://localhost:3000/</a> </li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp7ct39xopos2qicg51yg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp7ct39xopos2qicg51yg.png" alt="Image 15a" width="800" height="265"></a></p>

<p>Your browser should give this output: </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxsfd3vfx48rokncip0k2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxsfd3vfx48rokncip0k2.png" alt="Image 15b" width="800" height="492"></a></p>

<p><strong>Test endpoints (using terminal windows)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
---

curl http://localhost:3000/         # Homepage
curl http://localhost:3000/health   # Health check JSON
curl http://localhost:3000/info     # System info JSON  
curl http://localhost:3000/metrics  # Prometheus metrics

</code></pre>

</div>






<h2>
  
  
  Docker Commands:
</h2>

<p>Before running the Docker commands, ensure you've launched your Docker Desktop application. </p>

<p>This command builds the Docker image:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="c"># Build image</span>
docker build <span class="nt">-t</span> my-devops-app:latest <span class="nb">.</span>

</code></pre>

</div>



<p>Output of command in VScode terminal:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo2cqgxg0g1goz7w2wpqm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo2cqgxg0g1goz7w2wpqm.png" alt="Image 16a" width="800" height="200"></a></p>

<p>This shows the newly built container image <strong>my-devops-app:latest</strong> in Docker Desktop:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feb7x9qowf5k5j3fxa7o1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feb7x9qowf5k5j3fxa7o1.png" alt="Image 45" width="800" height="479"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run container</span>

docker run  <span class="nt">--name</span> my-devops-container <span class="nt">-d</span> <span class="nt">-p</span> 3000:3000 <span class="nt">--restart</span> unless-stopped my-devops-app:latest

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwu5hqntm3hcgnl239abm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwu5hqntm3hcgnl239abm.png" alt="Image 16b" width="800" height="157"></a></p>

<p>This shows the running container <strong>(my-devops-container)</strong> on Docker Desktop: </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftlbevvw5mcb8g8xaeaag.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftlbevvw5mcb8g8xaeaag.png" alt="Image 46" width="800" height="463"></a></p>

<p>From your browser (<a href="http://localhost:3000" rel="noopener noreferrer">http://localhost:3000</a>), you should see this:  </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvy1xen3xid3ntwmlb0t5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvy1xen3xid3ntwmlb0t5.png" alt="Image 17" width="800" height="478"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check container status</span>
docker ps

<span class="c"># Test health check in terminal</span>
curl http://localhost:3000/health

</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Stop container</span>

docker stop my-devops-container

<span class="c"># Delete Container</span>
docker <span class="nb">rm </span>my-devops-container

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2aqwl4alup5tvafol4xb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2aqwl4alup5tvafol4xb.png" alt="Image 18" width="800" height="203"></a></p>




<h2>
  
  
  Docker Compose Commands
</h2>

<p>Run with Docker Compose:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="c"># Start all services defined in docker-compose.yml</span>
docker-compose up <span class="nt">-d</span>

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fepdsc8lkqgvfh1907c05.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fepdsc8lkqgvfh1907c05.png" alt="Image 19" width="800" height="435"></a></p>

<p>After running the above syntax, it also runs the application and  displays it in the browser via <a href="http://localhost:3000" rel="noopener noreferrer">http://localhost:3000</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Stop all services and clean up</span>
docker-compose down

</code></pre>

</div>



<p>Output of command in VScode terminal:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy510j3p13kcwoaa763gv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy510j3p13kcwoaa763gv.png" alt="Image 20" width="800" height="186"></a></p>


<h2>
  
  
  Step 9: Deploy to GitHub
</h2>

<p>We'll now commit the code and push it to GitHub, allowing the automated CI pipeline to begin processing.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
 <span class="c"># Add all files to Git staging area</span>
git add <span class="nb">.</span>

<span class="c"># Create your first commit with a descriptive message</span>

git commit <span class="nt">-m</span> <span class="s2">"Initial commit: Complete DevOps setup with working CI"</span>

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpc8usw7c9ilppkccs58o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpc8usw7c9ilppkccs58o.png" alt="Image 22" width="800" height="269"></a></p>

<p><strong>IMPORTANT</strong>: Before running these commands:</p>

<ul>
<li>Go to GitHub.com and create a new repository, give it a name of your choice. I called mine <strong>"my-devops-project"</strong>
</li>
<li>
<strong>DO NOT</strong> initialize it with README, .gitignore, or license (we already have these)</li>
<li>Copy the repository URL from GitHub</li>
<li>Replace <strong>YOUR_GITHUB_USERNAME</strong> below with your actual GitHub username
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="c"># Set main as the default branch</span>
git branch <span class="nt">-M</span> main

<span class="c"># Connect to your GitHub repository (UPDATE THIS URL!)</span>
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/my-devops-project.git

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffhghuk5jf5dtqdtloc6q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffhghuk5jf5dtqdtloc6q.png" alt="Image 23" width="800" height="97"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
<span class="c"># Push your code to GitHub for the first time</span>
git push <span class="nt">-u</span> origin main

</code></pre>

</div>



<p>Output of command in VScode terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv2a2r5ripe57kiv10lqs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv2a2r5ripe57kiv10lqs.png" alt="Image 24" width="800" height="248"></a></p>

<p>What you'll see: Your code appears on GitHub, and the CI/CD pipeline starts running automatically.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs7323ymyyg6toqf0k2vh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs7323ymyyg6toqf0k2vh.png" alt="Image 25" width="800" height="523"></a></p>




<h2>
  
  
  Step 10: Deploy to Azure Web Apps via Azure Portal
</h2>

<p>We’ve built our Node.js app, containerized it with Docker, and set up GitHub Actions for CI/CD. Now it’s time to take it live by deploying to Azure Web Apps, a fully managed Platform-as-a-Service (PaaS) that runs your app in the cloud without you managing servers or clusters.</p>

<p><strong>Why Azure Web Apps?</strong></p>

<ul>
<li><p>Zero server management → Microsoft handles scaling, updates, and runtime.</p></li>
<li><p>CI/CD ready → GitHub Actions can publish new versions automatically.</p></li>
<li><p>Scalable → Apps scale up or out based on load.</p></li>
<li><p>Secure &amp; monitored → Logs, metrics, and diagnostics included by default.</p></li>
</ul>

<p><strong>Steps in the Azure Portal</strong></p>

<ol>
<li><strong>Log in to Azure Portal</strong></li>
<li>
<p>Go to <a href="https://portal.azure.com" rel="noopener noreferrer">Azure Portal</a> and sign in with your Azure account.</p>

<ol>
<li><strong>Create a Resource Group</strong></li>
</ol>


</li>

</ol>

<ul>
<li><p>In the left-hand menu, search for Resource Groups.</p></li>
<li><p>Click Create → provide a name (e.g., my-devops-app-rg) and select a region (preferably one close to you).</p></li>
</ul>

<ol>
<li><strong>Create an App Service</strong></li>
</ol>

<ul>
<li>From the menu, search for App Services → click Create and select web app.</li>
</ul>

<p>Fill in:</p>

<ul>
<li><p>Resource Group: my-devops-app-rg</p></li>
<li><p>App Name: my-devops-lab-app (must be unique)</p></li>
<li><p>Publish: Code</p></li>
<li><p>Operating System: Linux</p></li>
<li><p>Region: Choose the same as your resource group</p></li>
<li><p>Pricing Plan: Start with B1 (Basic) for testing</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc6rqny8fnv6tb3yu5y5n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc6rqny8fnv6tb3yu5y5n.png" alt="Image 34" width="800" height="805"></a></p>

<p><strong>Configure Deployment</strong></p>

<p>In the Deployment tab, enable:</p>

<ul>
<li><p>Option: Continuous deployment</p></li>
<li><p>Select Authorize and connect your GitHub account with the Azure App Service to enable access to your repository.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F585wqcyq3d5w8km2hpr2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F585wqcyq3d5w8km2hpr2.png" alt="Image 38" width="800" height="487"></a></p>

<p>Select the following details: </p>

<ul>
<li><p>Organization: your GitHub account username</p></li>
<li><p>Repository: your repository name</p></li>
<li><p>Branch: your branch main (e.g main or master)</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8uf62nwe8zn9ayh45nss.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8uf62nwe8zn9ayh45nss.png" alt="Image 39" width="800" height="800"></a></p>

<p>Review + Create</p>

<p>Click Review + Create and wait for Azure to provision the service.</p>




<p><strong>CI/CD Integration from Portal</strong></p>

<p>Azure Portal can automatically create a GitHub Actions workflow for you:</p>

<ul>
<li><p>Azure will generate a workflow file in .github/workflows/</p></li>
<li><p>This workflow builds your app and redeploys automatically whenever you push to GitHub</p></li>
</ul>

<p>Select <strong>Actions</strong> to see the pipeline running:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fywu1h5jen16dqylyjs6j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fywu1h5jen16dqylyjs6j.png" alt="Image 40" width="800" height="427"></a></p>

<p>Below depicts the <strong>Continuous Integration (CI)</strong> pipeline phase: </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqc3kcjyahej4af1yvb98.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqc3kcjyahej4af1yvb98.png" alt="Image 41b" width="800" height="523"></a></p>

<p>This depicts the <strong>Continuous Deployment (CD)</strong> pipeline phase:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fay176mm0tyfuz9p4vtf4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fay176mm0tyfuz9p4vtf4.png" alt="Image 41a" width="800" height="523"></a></p>

<p><strong>Access Your App</strong></p>

<p>This indicates our web app service has been successfully deployed. Select <strong>Go to Resource</strong> </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F27bro68cgwo5hl52vh8k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F27bro68cgwo5hl52vh8k.png" alt="Image 42" width="800" height="370"></a></p>

<p>Select <strong>Browse</strong> to access the application in your browser<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb6kewu1ufrlv7zoukg0e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb6kewu1ufrlv7zoukg0e.png" alt="Image 44" width="800" height="410"></a></p>

<p>This is the application live in the browser:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjmy3i5dlqq3284o5sz5q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjmy3i5dlqq3284o5sz5q.png" alt="Image 43" width="800" height="531"></a></p>

<h2>
  
  
  Wrapping Up
</h2>

<p>You just built a complete <strong>CI/CD pipeline</strong> for a Node.js application:</p>

<ul>
<li>Automated builds &amp; tests with <strong>GitHub Actions</strong>
</li>
<li>Containerization with <strong>Docker</strong>
</li>
<li>Local orchestration with <strong>Docker Compose</strong>
</li>
<li>Cloud hosting on <strong>Azure Web Apps</strong>
</li>
</ul>

<p>This workflow eliminates manual deployments, ensures code quality, and makes your app production-ready. 🚀</p>

<p>Would you like me to also add <strong>screenshots and diagrams</strong> (pipeline flow, Kubernetes architecture) to make the Dev.to article more visually engaging?</p>

