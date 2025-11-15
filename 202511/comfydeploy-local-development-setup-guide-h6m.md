---
Title: ComfyDeploy Local Development Setup Guide
Description: 
Author: Umut Tan
Date: 2025-11-15T21:36:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>Getting ComfyDeploy running locally is a bunch of work; but is a dream, and a good helpful one; in which you gotta spend all your weekend just to get things running... </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcjoome7jn1lhcwlg507k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcjoome7jn1lhcwlg507k.png" alt=" " width="800" height="461"></a></p>

<h2>
  
  
  What You'll See
</h2>

<p>By completing this guide, you'll gain hands-on experience with:</p>

<p><strong>Full-Stack Development:</strong></p>

<ul>
<li>🚀 <strong>Modern Web Stack</strong>: React/Vite frontend + Python FastAPI backend</li>
<li>
<strong>Database Management</strong>: PostgreSQL with Drizzle ORM, schema design, and migrations</li>
<li>
<strong>Authentication</strong>: Clerk integration with JWT validation</li>
<li>
<strong>Billing &amp; Subscriptions</strong>: Autumn API for SaaS monetization and feature limits</li>
<li>
<strong>Containerization</strong>: Docker Compose for local development infrastructure</li>
</ul>

<p><strong>Cloud &amp; Serverless Architecture:</strong></p>

<ul>
<li>
<strong>Serverless Computing</strong>: Deploy GPU-accelerated Python functions with Modal</li>
<li>
<strong>Persistent Storage</strong>: Modal volumes for model storage and file management</li>
<li>
<strong>Cloud Storage</strong>: AWS S3 integration for file uploads and assets</li>
<li>
<strong>Webhooks &amp; Tunneling</strong>: ngrok for local development with cloud callbacks</li>
</ul>

<p><strong>DevOps &amp; Infrastructure:</strong></p>

<ul>
<li>
<strong>Monorepo Management</strong>: Turborepo for managing multiple applications</li>
<li>
<strong>Environment Configuration</strong>: Managing secrets and API keys across services</li>
<li>
<strong>CI/CD Concepts</strong>: Understanding deployment pipelines and environments</li>
<li>
<strong>Debugging</strong>: Reading logs, troubleshooting errors, and fixing common issues</li>
</ul>

<p><strong>AI/ML Deployment:</strong></p>

<ul>
<li>
<strong>ComfyUI Integration</strong>: Running AI image generation workflows in production</li>
<li>
<strong>GPU Management</strong>: Serverless GPU allocation and concurrency limits</li>
<li>
<strong>Usage Tracking</strong>: Monitoring GPU credits and resource consumption</li>
</ul>

<p><strong>Free Credits &amp; Resources:</strong></p>

<ul>
<li>💰 <strong>$5 USD in Modal Credits</strong>: Sign up at <a href="https://modal.com/" rel="noopener noreferrer">https://modal.com/</a> to get free credits for GPU compute</li>
<li>🎯 <strong>Run Real Workflows</strong>: Deploy and execute ComfyUI workflows on production-grade infrastructure</li>
<li>🧪 <strong>Experiment with AI Models</strong>: Test different models, custom nodes, and workflow configurations</li>
<li>📚 <strong>Production-Ready Codebase</strong>: Learn from a real SaaS application architecture</li>
</ul>

<p><strong>Skills You'll Develop:</strong></p>

<ul>
<li>Setting up complex development environments with multiple services</li>
<li>Integrating third-party APIs (Clerk, Autumn, AWS, Modal, GitHub)</li>
<li>Managing database schemas and foreign key relationships</li>
<li>Debugging full-stack applications across frontend, backend, and cloud services</li>
<li>Understanding SaaS billing models and feature gating</li>
<li>Working with serverless architectures and GPU compute</li>
</ul>

<p>By the end of this guide, you'll have a <strong>fully functional local development environment</strong> for ComfyDeploy, ready to build features, fix bugs, and deploy AI workflows! 🚀</p>

<h2>
  
  
  Prerequisites
</h2>

<h3>
  
  
  Required Software &amp; Tools
</h3>

<p>Install these dependencies before starting:</p>

<p><strong>Core Development Tools:</strong></p>

<ul>
<li>
<strong>Node.js</strong> (v18 or higher) - JavaScript runtime

<ul>
<li>macOS: <code>brew install node</code>
</li>
<li>Or download from <a href="https://nodejs.org/" rel="noopener noreferrer">https://nodejs.org/</a>
</li>
</ul>


</li>

<li>

<strong>Bun</strong> (v1.0+) - Fast JavaScript package manager and runtime

<ul>
<li>macOS: <code>brew install oven-sh/bun/bun</code>
</li>
<li>Or: <code>curl -fsSL https://bun.sh/install | bash</code>
</li>
</ul>


</li>

<li>

<strong>Python</strong> (3.12.0 exactly) - Required for API server

<ul>
<li>macOS: <code>brew install python@3.12</code>
</li>
<li>Or use pyenv: <code>pyenv install 3.12.0</code>
</li>
</ul>


</li>

<li>

<strong>uv</strong> - Fast Python package manager

<ul>
<li>macOS: <code>brew install uv</code>
</li>
<li>Or: <code>curl -LsSf https://astral.sh/uv/install.sh | sh</code>
</li>
</ul>


</li>

</ul>

<p><strong>Infrastructure:</strong></p>

<ul>
<li>
<strong>Docker Desktop</strong> - For PostgreSQL and Redis

<ul>
<li>macOS: <code>brew install --cask docker</code>
</li>
<li>Or download from <a href="https://www.docker.com/products/docker-desktop/" rel="noopener noreferrer">https://www.docker.com/products/docker-desktop/</a>
</li>
</ul>


</li>

<li>

<strong>PostgreSQL Client</strong> (psql) - For database management

<ul>
<li>macOS: <code>brew install postgresql@16</code>
</li>
<li>Or use the client included with Docker</li>
</ul>


</li>

</ul>

<p><strong>Cloud &amp; Networking:</strong></p>

<ul>
<li>
<strong>AWS CLI</strong> - For S3 bucket management

<ul>
<li>macOS: <code>brew install awscli</code>
</li>
<li>Or: <code>curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg" &amp;&amp; sudo installer -pkg AWSCLIV2.pkg -target /</code>
</li>
</ul>


</li>

<li>

<strong>Modal CLI</strong> - For serverless deployment

<ul>
<li>Installed via Python: <code>pip install modal</code> (or included in uv sync)</li>
</ul>


</li>

<li>

<strong>ngrok</strong> - For local tunneling

<ul>
<li>macOS: <code>brew install ngrok/ngrok/ngrok</code>
</li>
<li>Or download from <a href="https://ngrok.com/download" rel="noopener noreferrer">https://ngrok.com/download</a>
</li>
</ul>


</li>

</ul>

<p><strong>Optional but Recommended:</strong></p>

<ul>
<li>
<strong>Git</strong> (latest version) - Version control

<ul>
<li>macOS: <code>brew install git</code>
</li>
</ul>


</li>

<li>

<strong>jq</strong> - JSON processor for debugging API responses

<ul>
<li>macOS: <code>brew install jq</code>
</li>
</ul>


</li>

</ul>

<h3>
  
  
  Required Service Accounts &amp; API Keys
</h3>

<p>You'll need accounts and API keys for:</p>

<ul>
<li>
<strong>Clerk</strong> (authentication) - get your publishable key, secret key, and JWT public key</li>
<li>
<strong>Autumn</strong> (billing/subscriptions) - get your secret key</li>
<li>
<strong>AWS S3</strong> (file storage) - create a bucket and get your credentials</li>
<li>
<strong>Modal</strong> (serverless compute) - create an account and get your token</li>
<li>
<strong>ngrok</strong> (for local development) - get your authtoken</li>
<li>
<strong>GitHub</strong> (API access) - create a personal access token for fetching ComfyUI version info</li>
</ul>

<h2>
  
  
  Step 1: Clone and Install Dependencies
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/comfy-deploy/comfydeploy.git
<span class="nb">cd </span>comfydeploy

<span class="c"># IMPORTANT: Initialize git submodules to pull in the actual code</span>
<span class="c"># The apps/api and apps/app directories are git submodules</span>
git submodule init
git submodule update

<span class="c"># Install API dependencies (Node.js packages)</span>
<span class="nb">cd </span>apps/api
bun <span class="nb">install</span>

<span class="c"># Install Python dependencies using uv (faster than pip)</span>
<span class="c"># This creates a virtual environment at apps/api/.venv</span>
uv <span class="nb">sync</span>

<span class="c"># Install frontend dependencies</span>
<span class="nb">cd</span> ../app
bun <span class="nb">install</span>
</code></pre>

</div>



<p><strong>Note:</strong> The repository uses git submodules for <code>apps/api</code> and <code>apps/app</code>. You must run <code>git submodule init &amp;&amp; git submodule update</code> to pull in the actual application code. Without this step, the directories will be empty.</p>

<p><strong>Python Dependencies:</strong> The API uses <code>pyproject.toml</code> instead of <code>requirements.txt</code>. Use <code>uv sync</code> to install Python dependencies - it will automatically create a virtual environment at <code>apps/api/.venv</code> with Python 3.12.0.</p>

<h2>
  
  
  Step 2: Configure Environment Variables
</h2>

<h3>
  
  
  Required Service Accounts &amp; API Keys
</h3>

<p>Before configuring, sign up for these services and get your API keys:</p>

<ol>
<li>
<p><strong>Clerk</strong> (Authentication) - <a href="https://clerk.com/" rel="noopener noreferrer">https://clerk.com/</a></p>

<ul>
<li>Dashboard: <a href="https://dashboard.clerk.com/" rel="noopener noreferrer">https://dashboard.clerk.com/</a>
</li>
<li>Get: <code>CLERK_PUBLISHABLE_KEY</code>, <code>CLERK_SECRET_KEY</code>, <code>CLERK_PUBLIC_JWT_KEY</code>
</li>
<li>Go to: Dashboard → Your App → API Keys → Find "JWKS Public Key" section</li>
</ul>
</li>
<li>
<p><strong>Autumn</strong> (Billing) - <a href="https://useautumn.com/" rel="noopener noreferrer">https://useautumn.com/</a></p>

<ul>
<li>Get: <code>AUTUMN_SECRET_KEY</code>
</li>
<li>Authenticate CLI: <code>npx atmn auth</code>
</li>
<li>Push config: <code>npx atmn push</code> (after configuring <code>autumn.config.ts</code>)</li>
</ul>
</li>
<li>
<p><strong>AWS S3</strong> (File Storage) - <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer">https://aws.amazon.com/s3/</a></p>

<ul>
<li>Create bucket: <code>aws s3 mb s3://comfydeploy-dev-storage --region us-east-1</code>
</li>
<li>Get credentials from AWS IAM or use existing AWS CLI credentials</li>
<li>Get: Access Key ID, Secret Access Key</li>
</ul>
</li>
</ol>

<p><strong>Example S3 Bucket Configuration:</strong></p>

<p>Here's one way to configure your S3 bucket for ComfyDeploy. This is a simple approach that works, but you may have better security methods (e.g., using presigned URLs, CloudFront, etc.):</p>

<p><strong>1. Access Control List (ACL):</strong></p>

<ul>
<li>Public READ access - Grants READ permission to "AllUsers" for public file access</li>
<li>Owner has FULL_CONTROL</li>
</ul>

<p><strong>2. CORS Configuration:</strong></p>

<ul>
<li>Allows all origins (<code>*</code>)</li>
<li>Allows all HTTP methods (GET, PUT, POST, DELETE, HEAD)</li>
<li>Allows all headers (<code>*</code>)</li>
<li>Exposes ETag header</li>
<li>Max age: 3000 seconds</li>
</ul>

<p><strong>3. Bucket Policy:</strong></p>

<ul>
<li>No specific bucket policy required (can be left empty)</li>
</ul>

<p><strong>4. Public Access Block:</strong></p>

<ul>
<li>All public access blocks disabled for public file access</li>
<li>
<code>BlockPublicAcls</code>: false</li>
<li>
<code>IgnorePublicAcls</code>: false</li>
<li>
<code>BlockPublicPolicy</code>: false</li>
<li>
<code>RestrictPublicBuckets</code>: false</li>
</ul>

<p><strong>5. Bucket Structure:</strong></p>

<ul>
<li>
<code>assets/upload/</code> - For uploaded assets</li>
<li>
<code>outputs/runs/</code> - For workflow output files</li>
</ul>

<p><strong>⚠️ Security Note:</strong> This configuration makes the bucket <strong>publicly readable</strong> by anyone on the internet. This is a simple approach that works for development, but you may want to implement more secure methods like presigned URLs, CloudFront distributions, or bucket policies for production use.</p>

<p><strong>To configure CORS via AWS CLI:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="c"># Create a cors.json file</span>
   <span class="nb">cat</span> <span class="o">&gt;</span> cors.json <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
   {
     "CORSRules": [
       {
         "AllowedHeaders": ["*"],
         "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
         "AllowedOrigins": ["*"],
         "ExposeHeaders": ["ETag"],
         "MaxAgeSeconds": 3000
       }
     ]
   }
</span><span class="no">   EOF

</span>   <span class="c"># Apply CORS configuration</span>
   aws s3api put-bucket-cors <span class="nt">--bucket</span> YOUR-BUCKET-NAME <span class="nt">--cors-configuration</span> file://cors.json <span class="nt">--region</span> YOUR-REGION

   <span class="c"># Set public read ACL</span>
   aws s3api put-bucket-acl <span class="nt">--bucket</span> YOUR-BUCKET-NAME <span class="nt">--acl</span> public-read <span class="nt">--region</span> YOUR-REGION

   <span class="c"># Disable public access blocks</span>
   aws s3api delete-public-access-block <span class="nt">--bucket</span> YOUR-BUCKET-NAME <span class="nt">--region</span> YOUR-REGION
</code></pre>

</div>



<p><strong>To verify your bucket configuration:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="c"># Check CORS</span>
   aws s3api get-bucket-cors <span class="nt">--bucket</span> YOUR-BUCKET-NAME <span class="nt">--region</span> YOUR-REGION

   <span class="c"># Check ACL</span>
   aws s3api get-bucket-acl <span class="nt">--bucket</span> YOUR-BUCKET-NAME <span class="nt">--region</span> YOUR-REGION

   <span class="c"># Check public access block</span>
   aws s3api get-public-access-block <span class="nt">--bucket</span> YOUR-BUCKET-NAME <span class="nt">--region</span> YOUR-REGION
</code></pre>

</div>



<ol>
<li>
<p><strong>Modal</strong> (Serverless Compute) - <a href="https://modal.com/" rel="noopener noreferrer">https://modal.com/</a></p>

<ul>
<li>Dashboard: <a href="https://modal.com/home" rel="noopener noreferrer">https://modal.com/home</a> → Settings → API Tokens</li>
<li>Get: <code>MODAL_TOKEN_ID</code>, <code>MODAL_TOKEN_SECRET</code>
</li>
</ul>
</li>
<li>
<p><strong>ngrok</strong> (Local Tunneling) - <a href="https://ngrok.com/" rel="noopener noreferrer">https://ngrok.com/</a></p>

<ul>
<li>Dashboard: <a href="https://dashboard.ngrok.com/" rel="noopener noreferrer">https://dashboard.ngrok.com/</a>
</li>
<li>Get: <code>NGROK_AUTHTOKEN</code> from "Your Authtoken" section</li>
</ul>
</li>
<li>
<p><strong>GitHub</strong> (API Access for ComfyUI Version Info) - <a href="https://github.com/" rel="noopener noreferrer">https://github.com/</a></p>

<ul>
<li>Settings → Developer settings → Personal access tokens → Tokens (classic)</li>
<li>Generate new token (classic) with <code>public_repo</code> scope</li>
<li>Get: <code>GITHUB_TOKEN</code> (starts with <code>ghp_</code>)</li>
</ul>
</li>
</ol>

<h3>
  
  
  Configure Environment Files
</h3>

<p>Copy the example env files and fill in your credentials:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cp </span>apps/api/.env.example apps/api/.env
<span class="nb">cp </span>apps/app/.env.example apps/app/.env
</code></pre>

</div>



<h3>
  
  
  Generate Encryption Key
</h3>

<p>Generate a secret encryption key for the API:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
<span class="nb">source</span> .venv/bin/activate
python3 <span class="nt">-c</span> <span class="s2">"from cryptography.fernet import Fernet; print(Fernet.generate_key().decode('utf-8'))"</span>
</code></pre>

</div>



<p>Copy the output and add it to <code>apps/api/.env</code> as <code>SECRET_ENCRYPTION_KEY</code>.</p>

<h3>
  
  
  Get Clerk JWT Public Key
</h3>

<ol>
<li>Go to Clerk Dashboard → Your App → API Keys</li>
<li>Scroll down to the "JWKS Public Key" section</li>
<li>Copy the entire key (starts with <code>-----BEGIN PUBLIC KEY-----</code>)</li>
<li>Add it to <code>apps/api/.env</code> as <code>CLERK_PUBLIC_JWT_KEY</code>
</li>
</ol>

<h3>
  
  
  Example Configuration
</h3>

<p><strong>apps/api/.env</strong> (key variables):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ENV=development
DATABASE_URL="postgresql://postgres:postgres@localhost:5480/verceldb"
CLERK_PUBLIC_JWT_KEY="-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----"
CLERK_SECRET_KEY="sk_test_..."
AUTUMN_SECRET_KEY="am_sk_test_..."
SPACES_BUCKET_V2="comfydeploy-dev-storage"
SPACES_ENDPOINT_V2="https://s3.amazonaws.com"
SPACES_REGION_V2="us-east-1"
SPACES_KEY_V2="AKIA..."
SPACES_SECRET_V2="..."
MODAL_ENVIRONMENT="dev"
MODAL_TOKEN_ID="ak-..."
MODAL_TOKEN_SECRET="as-..."
SHARED_MODEL_VOLUME_NAME="comfydeploy-dev-public-models"  # Check with: modal volume list
PUBLIC_MODEL_VOLUME_NAME="comfydeploy-dev-public-models"  # Should match your Modal volume name
NGROK_AUTHTOKEN="..."
GITHUB_TOKEN="ghp_..."
SECRET_ENCRYPTION_KEY="..."
CURRENT_API_URL="https://your-ngrok-url.ngrok-free.dev"  # ⚠️ MUST be ngrok URL, not localhost!
</code></pre>

</div>



<p><strong>⚠️ CRITICAL:</strong> <code>CURRENT_API_URL</code> must be set to your ngrok URL (from Step 5.5), NOT <code>http://localhost:3011</code>. Modal functions run in the cloud and cannot reach localhost. See Step 5.5 for how to get your ngrok URL.</p>

<p><strong>Note:</strong> To verify your Modal volume name, run <code>modal volume list</code> and use the exact name that appears in the output. See Step 5.6 for details.</p>

<p><strong>apps/app/.env</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="pk_test_..."
NEXT_PUBLIC_CD_API_URL="http://localhost:3011"
</code></pre>

</div>



<h2>
  
  
  Step 2.5: Set Up Autumn Billing Integration
</h2>

<p><strong>What is Autumn?</strong></p>

<p>Autumn is a billing and subscription management API for SaaS applications. It handles product definitions, pricing plans, feature limits, and usage tracking. ComfyDeploy uses Autumn to manage different subscription tiers (free, business, enterprise) and enforce feature limits like machine count, workflow limits, and GPU concurrency.</p>

<p><strong>Understanding autumn.config.ts</strong></p>

<p>The file <code>apps/api/autumn.config.ts</code> defines your entire billing structure. Here's what you need to know:</p>

<ul>
<li>
<strong>Features</strong>: Define what users can do (e.g., <code>machineLimit</code>, <code>gpuConcurrencyLimit</code>, <code>workflowLimit</code>)</li>
<li>
<strong>Products</strong>: Combine features into subscription tiers (e.g., <code>free</code>, <code>business</code>, <code>enterprise</code>)</li>
<li>
<strong>Pricing</strong>: Set monthly/yearly prices and included usage amounts</li>
</ul>

<p><strong>Example Configuration</strong></p>

<p>Here's a simplified version of what's in the config:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Define features</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">machineLimit</span> <span class="o">=</span> <span class="nf">feature</span><span class="p">({</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">machine_limit</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Machine Limit</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">continuous_use</span><span class="dl">"</span><span class="p">,</span>
<span class="p">});</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">gpuConcurrencyLimit</span> <span class="o">=</span> <span class="nf">feature</span><span class="p">({</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">gpu_concurrency_limit</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GPU Concurrency Limit</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">continuous_use</span><span class="dl">"</span><span class="p">,</span>
<span class="p">});</span>

<span class="c1">// Define products (subscription tiers)</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">free</span> <span class="o">=</span> <span class="nf">product</span><span class="p">({</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">free</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Free</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">is_default</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">items</span><span class="p">:</span> <span class="p">[</span>
    <span class="nf">featureItem</span><span class="p">({</span>
      <span class="na">feature_id</span><span class="p">:</span> <span class="nx">machineLimit</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
      <span class="na">included_usage</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span>  <span class="c1">// Free tier gets 3 machines</span>
    <span class="p">}),</span>
    <span class="nf">featureItem</span><span class="p">({</span>
      <span class="na">feature_id</span><span class="p">:</span> <span class="nx">gpuConcurrencyLimit</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
      <span class="na">included_usage</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>  <span class="c1">// Can run 1 GPU job at a time</span>
    <span class="p">}),</span>
  <span class="p">],</span>
<span class="p">});</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">business</span> <span class="o">=</span> <span class="nf">product</span><span class="p">({</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">business</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Business</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">items</span><span class="p">:</span> <span class="p">[</span>
    <span class="nf">pricedFeatureItem</span><span class="p">({</span>
      <span class="na">feature_id</span><span class="p">:</span> <span class="nx">gpuCredit</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
      <span class="na">price</span><span class="p">:</span> <span class="mf">0.01</span><span class="p">,</span>
      <span class="na">included_usage</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
      <span class="na">usage_model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">pay_per_use</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">}),</span>
    <span class="nf">featureItem</span><span class="p">({</span>
      <span class="na">feature_id</span><span class="p">:</span> <span class="nx">machineLimit</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
      <span class="na">included_usage</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span>  <span class="c1">// Business tier gets 100 machines</span>
    <span class="p">}),</span>
    <span class="nf">featureItem</span><span class="p">({</span>
      <span class="na">feature_id</span><span class="p">:</span> <span class="nx">gpuConcurrencyLimit</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
      <span class="na">included_usage</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>  <span class="c1">// Can run 10 GPU jobs at a time</span>
    <span class="p">}),</span>
  <span class="p">],</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Pushing Configuration to Autumn</strong></p>

<p>After updating <code>autumn.config.ts</code>, push your changes to Autumn:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
npx atmn push
</code></pre>

</div>



<p>This command validates your config and syncs it with the Autumn API. You'll see output confirming which products and features were created/updated.</p>

<p><strong>Verifying Your Setup</strong></p>

<p>Check that plans are active in the Autumn dashboard:</p>

<ol>
<li>Go to <a href="https://dashboard.autumn.dev" rel="noopener noreferrer">https://dashboard.autumn.dev</a> (or your Autumn instance)</li>
<li>Navigate to Products → verify your products are listed</li>
<li>Check that features have the correct limits</li>
<li>Test by creating a subscription in the dashboard</li>
</ol>

<p><strong>Common Gotcha: Missing Feature Definitions</strong></p>

<p>If you see "Max Machines Exceeded" errors even though users have paid plans, it's usually because:</p>

<ol>
<li>The <code>machineLimit</code> feature isn't defined in the product</li>
<li>The <code>included_usage</code> is set too low</li>
<li>The config wasn't pushed to Autumn after changes</li>
</ol>

<p><strong>Fix</strong>: Add the missing feature to your product and run <code>npx atmn push</code> again.</p>

<p><strong>Push Configuration to Autumn:</strong></p>

<p>After setting up your <code>.env</code> file with <code>AUTUMN_SECRET_KEY</code>, push the billing configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
npx atmn push
</code></pre>

</div>



<p>This will push all features and products defined in <code>autumn.config.ts</code> to your Autumn sandbox environment. You should see:</p>

<ul>
<li>✅ 20 features pushed (GPU types, credits, limits, etc.)</li>
<li>✅ 10 products pushed (Free, Creator, Deployment, Business plans + add-ons)</li>
</ul>

<p>View your products at: <a href="http://app.useautumn.com/sandbox/products" rel="noopener noreferrer">http://app.useautumn.com/sandbox/products</a></p>

<h3>
  
  
  🔑 GitHub Token Setup
</h3>

<p>The API uses GitHub's API to fetch the latest ComfyUI and ComfyUI-Deploy version information. Without a GitHub token, the <code>/api/latest-hashes</code> endpoint will fail.</p>

<p><strong>Create a GitHub Classic Token:</strong></p>

<ol>
<li>Go to <a href="https://github.com/settings/tokens" rel="noopener noreferrer">https://github.com/settings/tokens</a>
</li>
<li>Click "Generate new token" → "Generate new token (classic)"</li>
<li>Give it a descriptive name (e.g., "ComfyDeploy Local Dev")</li>
<li>Select scopes:

<ul>
<li>✅ <code>public_repo</code> (Access public repositories)</li>
</ul>
</li>
<li>Click "Generate token"</li>
<li>Copy the token (starts with <code>ghp_</code>)</li>
<li>Add to <code>apps/api/.env</code>:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nv">GITHUB_TOKEN</span><span class="o">=</span><span class="s2">"ghp_your_token_here"</span>
</code></pre>

</div>



<p><strong>Why is this needed?</strong></p>

<p>The API fetches:</p>

<ul>
<li>Latest ComfyUI commit hash from <code>comfyanonymous/ComfyUI</code>
</li>
<li>Latest ComfyUI release information</li>
<li>Latest ComfyUI-Deploy commit hash from <code>bennykok/comfyui-deploy</code>
</li>
</ul>

<p>Without the token, you'll see errors like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ERROR:api.routes.comfy_node:Error fetching repo info: Illegal header value b'Bearer '
INFO:     127.0.0.1:xxxxx - "GET /api/latest-hashes HTTP/1.1" 404 Not Found
</code></pre>

</div>



<h3>
  
  
  📦 Modal Volume Configuration
</h3>

<p><strong>What are Modal Volumes?</strong></p>

<p>Modal volumes are persistent storage for your serverless ComfyUI deployments. They store:</p>

<ul>
<li>Public models (shared across all users)</li>
<li>Private models (user-specific or organization-specific)</li>
<li>Workflow files and custom nodes</li>
</ul>

<p><strong>Configure Volume Names:</strong></p>

<p>For local development, you need to set the public model volume name in <code>apps/api/.env</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">SHARED_MODEL_VOLUME_NAME</span><span class="o">=</span><span class="s2">"comfydeploy-dev-public-models"</span>
<span class="nv">PUBLIC_MODEL_VOLUME_NAME</span><span class="o">=</span><span class="s2">"comfydeploy-dev-public-models"</span>
</code></pre>

</div>



<p><strong>Why is this needed?</strong></p>

<p>Without these environment variables, Modal will fail to create volumes with the error:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>InvalidError: Invalid Volume name: ''.
Names may contain only alphanumeric characters, dashes, periods, and underscores,
must be shorter than 64 characters, and cannot conflict with App ID strings.
</code></pre>

</div>



<p><strong>Volume Naming Convention:</strong></p>

<ul>
<li>
<strong>Local Development</strong>: Use a simple name like <code>comfydeploy-dev-public-models</code>
</li>
<li>
<strong>Production</strong>: Use organization-specific names like <code>models_org_&lt;org_id&gt;</code>
</li>
<li>
<strong>Private Volumes</strong>: Automatically named as <code>models_&lt;user_id&gt;</code> or <code>models_org_&lt;org_id&gt;</code>
</li>
</ul>

<p><strong>How Volumes Work:</strong></p>

<ol>
<li>When you deploy a serverless machine, Modal creates volumes with <code>create_if_missing=True</code>
</li>
<li>The public volume is shared across all deployments for common models</li>
<li>Private volumes are created per-user or per-organization for custom models</li>
<li>Volumes persist between deployments and are mounted at runtime</li>
</ol>

<p><strong>Verifying Volume Creation:</strong></p>

<p>After starting the API server, check the Modal dashboard:</p>

<ol>
<li>Go to <a href="https://modal.com/home" rel="noopener noreferrer">https://modal.com/home</a>
</li>
<li>Navigate to "Volumes" in the sidebar</li>
<li>You should see <code>comfydeploy-dev-public-models</code> listed</li>
<li>The volume will be created automatically on first use</li>
</ol>

<h2>
  
  
  Step 3: Start Docker Services
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
docker compose up <span class="nt">-d</span>
</code></pre>

</div>



<p>This starts PostgreSQL, Redis, pg_proxy, and serverless-redis-http containers.</p>

<p>✓ <strong>Verify</strong>: Check that all services are running:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker compose ps

<span class="c"># Expected output should show:</span>
<span class="c"># - api-postgres-1 (port 5480)</span>
<span class="c"># - api-redis-1 (port 6379)</span>
<span class="c"># - api-pg_proxy-1 (port 5481)</span>
<span class="c"># - api-serverless-redis-http-1 (port 8079)</span>
</code></pre>

</div>



<p>If a service failed to start, check logs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker compose logs postgres  <span class="c"># or redis, etc.</span>
</code></pre>

</div>



<h2>
  
  
  Step 4: Run Database Migrations
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
bun run migrate-local
</code></pre>

</div>



<p><strong>⚠️ Troubleshooting Connection Timeout:</strong></p>

<p>If migrations fail with <code>CONNECT_TIMEOUT localhost:5480</code>, VS Code's port forwarding may be conflicting:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check what's listening on port 5480</span>
lsof <span class="nt">-i</span> :5480

<span class="c"># If you see "Code Helper" process, kill it:</span>
<span class="nb">kill</span> <span class="nt">-9</span> &lt;PID&gt;

<span class="c"># Then retry migrations</span>
bun run migrate-local
</code></pre>

</div>



<p>✓ <strong>Verify</strong>: Check that migrations completed without errors. You should see output like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Database is live
Migrating... DB 1
Done!
</code></pre>

</div>



<h2>
  
  
  Step 4.5: Understanding the Database Architecture
</h2>

<p><strong>Schema Structure</strong></p>

<p>All ComfyDeploy tables live in the <code>comfyui_deploy</code> schema (not the default <code>public</code> schema). This keeps your data organized and separate from other PostgreSQL objects.</p>

<p><strong>Key Tables and Relationships</strong></p>

<p>Here's the core data model:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>users (root table)
├── machines (user_id → users.id)
│   ├── machine_versions (machine_id → machines.id)
│   └── workflow_runs (machine_id → machines.id)
├── workflows (user_id → users.id)
│   ├── workflow_versions (workflow_id → workflows.id)
│   └── workflow_runs (workflow_id → workflows.id)
├── user_volume (user_id → users.id)
│   └── models (user_volume_id → user_volume.id)
└── deployments (user_id → users.id)
</code></pre>

</div>



<p><strong>Critical Foreign Key Relationships</strong></p>

<ul>
<li>
<code>machines.user_id</code> → <code>users.id</code> (cascade delete)</li>
<li>
<code>user_volume.user_id</code> → <code>users.id</code> (no cascade)</li>
<li>
<code>workflow_runs.machine_id</code> → <code>machines.id</code> (set null on delete)</li>
<li>
<code>models.user_volume_id</code> → <code>user_volume.id</code> (cascade delete)</li>
</ul>

<p><strong>Verify Your Schema</strong></p>

<p>Check that tables exist and have the right structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Connect to the database</span>
psql postgresql://postgres:postgres@localhost:5480/verceldb

<span class="c"># List all tables in the comfyui_deploy schema</span>
<span class="se">\d</span>t comfyui_deploy.<span class="k">*</span>

<span class="c"># Check the users table structure</span>
<span class="se">\d</span> comfyui_deploy.users

<span class="c"># Count rows in key tables</span>
SELECT COUNT<span class="o">(</span><span class="k">*</span><span class="o">)</span> FROM comfyui_deploy.users<span class="p">;</span>
SELECT COUNT<span class="o">(</span><span class="k">*</span><span class="o">)</span> FROM comfyui_deploy.machines<span class="p">;</span>
SELECT COUNT<span class="o">(</span><span class="k">*</span><span class="o">)</span> FROM comfyui_deploy.workflows<span class="p">;</span>
</code></pre>

</div>



<p><strong>Why Fresh Docker Restarts Wipe Data</strong></p>

<p>When you run <code>docker-compose down</code>, the PostgreSQL container is removed along with its data volume. This is why you need to:</p>

<ol>
<li>Re-run migrations (<code>bun run migrate-local</code>)</li>
<li>Recreate user records (see Step 6)</li>
</ol>

<p>In production, you'd use persistent volumes to prevent data loss.</p>

<h2>
  
  
  Step 5: Start the Development Servers
</h2>

<p>Start both servers (they will run in the background):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start API server (runs on port 3011)</span>
<span class="nb">cd </span>apps/api
bun run dev

<span class="c"># Start Frontend server (runs on port 3001)</span>
<span class="nb">cd </span>apps/app
bun run dev
</code></pre>

</div>



<p>✓ <strong>Verify API Server</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:3011/
<span class="c"># Expected: {"detail":"Not Found"} (this is normal - root path returns 404)</span>

<span class="c"># Check if server is responding</span>
curl <span class="nt">-I</span> http://localhost:3011/
<span class="c"># Expected: HTTP/1.1 404 Not Found (server is running)</span>
</code></pre>

</div>



<p>✓ <strong>Verify Frontend</strong>: Open <a href="http://localhost:3001" rel="noopener noreferrer">http://localhost:3001</a> in your browser</p>

<ul>
<li>You should see the Comfy Deploy login page</li>
<li>Try logging in with your Clerk credentials</li>
</ul>

<p><strong>Note:</strong> The API server may show some <code>SyntaxWarning</code> messages about invalid escape sequences - these are harmless and don't affect functionality.</p>

<h2>
  
  
  Step 5.5: Set Up ngrok Tunnel (CRITICAL - Required for Local Development)
</h2>

<p><strong>⚠️ CRITICAL STEP:</strong> ngrok is <strong>required</strong> for local development with Modal. Without it, Modal functions running in the cloud cannot send status updates back to your local API server, and workflows will get stuck in "not-started" status.</p>

<h3>
  
  
  Install ngrok
</h3>

<p>If you haven't installed ngrok yet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># macOS</span>
brew <span class="nb">install </span>ngrok

<span class="c"># Or download from https://ngrok.com/download</span>
</code></pre>

</div>



<h3>
  
  
  Configure ngrok Authentication
</h3>

<p>Get your authtoken from the ngrok dashboard:</p>

<ul>
<li>Dashboard: <a href="https://dashboard.ngrok.com/get-started/your-authtoken" rel="noopener noreferrer">https://dashboard.ngrok.com/get-started/your-authtoken</a>
</li>
<li>Copy your authtoken</li>
</ul>

<p>Then configure ngrok:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ngrok config add-authtoken YOUR_AUTHTOKEN
</code></pre>

</div>



<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ngrok config add-authtoken YOUR_AUTHTOKEN
</code></pre>

</div>



<h3>
  
  
  Start ngrok Tunnel
</h3>

<p>Launch ngrok to create a public tunnel to your local API:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ngrok http 3011
</code></pre>

</div>



<p>You'll see output like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Session Status                online
Account                       Your Name (Plan: Free)
Version                       3.x.x
Region                        United States (us)
Latency                       -
Web Interface                 http://127.0.0.1:4040
Forwarding                    https://ngrok_generated_url.dev -&gt; http://localhost:3011
</code></pre>

</div>



<p><strong>Copy the <code>Forwarding</code> URL</strong> (the https one, e.g., <code>https://shelia-nondedicative-kaidence.ngrok-free.dev</code>)</p>

<h3>
  
  
  Update Environment Variables
</h3>

<p>Add the ngrok URL to your <code>apps/api/.env</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">NGROK_DOMAIN</span><span class="o">=</span><span class="s2">"https://your-ngrok-url.ngrok-free.dev"</span>
</code></pre>

</div>



<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">NGROK_DOMAIN</span><span class="o">=</span><span class="s2">"https://ngrok_generated_url.dev"</span>
</code></pre>

</div>



<p><strong>Important:</strong> Keep ngrok running in a separate terminal window. If ngrok stops, Modal functions won't be able to communicate with your API.</p>

<p>✓ <strong>Verify ngrok is working:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Test that your ngrok URL reaches your local API</span>
curl https://your-ngrok-url.ngrok-free.dev/
<span class="c"># Expected: {"detail":"Not Found"} (this is normal - root path returns 404)</span>
</code></pre>

</div>



<h2>
  
  
  Step 5.6: Set Up Modal for Serverless Compute
</h2>

<p><strong>What is Modal?</strong></p>

<p>Modal is a serverless platform for running Python code with GPU access. ComfyDeploy uses Modal to run ComfyUI workflows in the cloud without needing to manage servers. When you create a "serverless machine" in ComfyDeploy, it's actually a Modal app that gets deployed and runs your workflows.</p>

<p><strong>Creating a Modal Environment</strong></p>

<p>Modal uses "environments" to organize deployments (dev, staging, production). You've already configured this in your <code>.env</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>MODAL_ENVIRONMENT="dev"
MODAL_TOKEN_ID="your-modal-token-id"
MODAL_TOKEN_SECRET="your-modal-token-secret"
</code></pre>

</div>



<p><strong>⚠️ CRITICAL: Deploying the volume-operations App</strong></p>

<blockquote>
<p><strong>This step is REQUIRED!</strong> Without deploying the <code>volume-operations</code> app, model downloads will fail and workflows won't be able to access models from Modal volumes.</p>
</blockquote>

<p>The <code>volume-operations</code> Modal app handles downloading models to Modal volumes. Deploy it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
modal deploy src/modal_apps/modal_downloader.py::modal_downloader_app
</code></pre>

</div>



<p><strong>What this does:</strong></p>

<ul>
<li>Creates a Modal app called "volume-operations" in your Modal workspace</li>
<li>Enables downloading models from HuggingFace, CivitAI, and arbitrary URLs</li>
<li>Uploads models to Modal volumes for use in workflows</li>
<li><strong>Required for model management functionality to work</strong></li>
</ul>

<p>✓ <strong>Verify Modal Deployment</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># List all Modal apps in your workspace</span>
modal app list

<span class="c"># Expected output should include "volume-operations"</span>
</code></pre>

</div>



<p>Check the Modal dashboard at <a href="https://modal.com/apps" rel="noopener noreferrer">https://modal.com/apps</a> to see your deployed apps.</p>

<p><strong>Common Issues:</strong></p>

<ul>
<li>If you see "Function not found" errors when downloading models, you likely forgot this step</li>
<li>The app must be deployed to the same Modal environment specified in <code>MODAL_ENVIRONMENT</code>
</li>
<li>Re-run the deploy command if you change Modal environments</li>
</ul>

<p><strong>Configuring Modal Volumes</strong></p>

<p>Modal volumes are persistent storage that Modal functions can access. ComfyDeploy creates volumes for:</p>

<ul>
<li>
<code>comfydeploy-dev-public-models</code>: Shared models available to all workflows</li>
<li>
<code>models_org_&lt;org_id&gt;</code>: Organization-specific models</li>
<li>
<code>models_&lt;user_id&gt;</code>: User-specific models</li>
</ul>

<p><strong>Check your existing Modal volumes:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>modal volume list
</code></pre>

</div>



<p>You should see output like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Volumes in environment 'dev'
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━┓
┃ Name                                    ┃ Created at           ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━┩
│ comfydeploy-dev-public-models           │ 2025-11-15 22:23 +03 │
│ models_org_35Wmd12wjmWmYICCWpxZykn4YVq  │ 2025-11-15 22:18 +03 │
│ public-models                           │ 2025-10-18 19:05 +03 │
└─────────────────────────────────────────┴──────────────────────┘
</code></pre>

</div>



<p><strong>Update your <code>.env</code> with the correct volume name:</strong></p>

<p>Look for the volume that matches your environment (e.g., <code>comfydeploy-dev-public-models</code> for dev environment), then update <code>apps/api/.env</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Modal Volume Configuration</span>
<span class="nv">SHARED_MODEL_VOLUME_NAME</span><span class="o">=</span>comfydeploy-dev-public-models
<span class="nv">PUBLIC_MODEL_VOLUME_NAME</span><span class="o">=</span>comfydeploy-dev-public-models
</code></pre>

</div>



<p><strong>Important Notes:</strong></p>

<ul>
<li>The volume name must match exactly what appears in <code>modal volume list</code>
</li>
<li>If the volume doesn't exist, Modal will create it automatically on first use</li>
<li>Use environment-specific names (e.g., <code>comfydeploy-dev-*</code> for dev, <code>comfydeploy-prod-*</code> for production)</li>
<li>Both <code>SHARED_MODEL_VOLUME_NAME</code> and <code>PUBLIC_MODEL_VOLUME_NAME</code> can point to the same volume for local development</li>
</ul>

<p>When you download a model through the UI, it gets uploaded to the appropriate volume, and then your workflows can access it.</p>

<h2>
  
  
  The ngrok Part (Important!)
</h2>

<p>Here's the thing: Modal functions run in the cloud and need to send status updates back to your local API. They can't reach <code>http://localhost:3011</code> directly. That's where ngrok comes in.</p>

<p>ngrok creates a public tunnel to your local API. When you start it, you'll get a URL like <code>https://abc123.ngrok-free.app</code>. Update your <code>.env</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CURRENT_API_URL="https://ngrok_generated_url.dev"
</code></pre>

</div>



<p>Then restart your API server. Now Modal functions can reach your API and send status updates properly.</p>

<h2>
  
  
  Step 6: Create a User Record in the Database (CRITICAL - Must Do After First Login)
</h2>

<p><strong>⚠️ CRITICAL STEP:</strong> After logging in with Clerk for the first time, you <strong>must</strong> create a user record in your local database. Without this, you'll get <code>ForeignKeyViolationError</code> errors when trying to use the application.</p>

<p><strong>Why is this needed?</strong></p>

<p>Clerk handles authentication and creates a user in their system, but your local PostgreSQL database doesn't know about this user yet. Most API endpoints require a user record to exist in the <code>comfyui_deploy.users</code> table due to foreign key constraints.</p>

<p><strong>When to do this:</strong></p>

<ol>
<li>After your first login via the frontend (<a href="http://localhost:3001" rel="noopener noreferrer">http://localhost:3001</a>)</li>
<li>After restarting Docker (which wipes the database)</li>
<li>Whenever you see errors like: <code>Key (user_id)=(user_xxx) is not present in table "users"</code>
</li>
</ol>

<h3>
  
  
  Get Your Clerk User ID
</h3>

<p>First, log in to the frontend at <a href="http://localhost:3001" rel="noopener noreferrer">http://localhost:3001</a>. Then get your Clerk user ID:</p>

<p><strong>Option 1: From Browser DevTools</strong></p>

<ol>
<li>Open DevTools (F12)</li>
<li>Go to Console tab</li>
<li>Type: <code>window.Clerk.user.id</code>
</li>
<li>Copy the user ID (starts with <code>user_</code>)</li>
</ol>

<p><strong>Option 2: From API Logs</strong><br>
Check your API server logs - you'll see the user ID in authentication errors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ERROR: Key (user_id)=(user_123456789) is not present in table "users"
</code></pre>

</div>



<h3>
  
  
  Create the User Record
</h3>

<p>Replace <code>your-clerk-user-id</code>, <code>your-username</code>, and <code>Your Name</code> with your actual values:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
<span class="nb">source</span> .venv/bin/activate
python3 <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
import asyncio
from sqlalchemy import text
from sqlalchemy.ext.asyncio import create_async_engine

DATABASE_URL = "postgresql+asyncpg://postgres:postgres@localhost:5480/verceldb"
engine = create_async_engine(DATABASE_URL)

async def main():
    async with engine.begin() as conn:
        await conn.execute(text("""
            INSERT INTO comfyui_deploy.users (id, username, name, created_at, updated_at)
            VALUES (:id, :username, :name, now(), now())
            ON CONFLICT (id) DO NOTHING
        """), {
            "id": "your-clerk-user-id",  # e.g., "user_35WjS9hWy1iZFJ6WbeThk92qcgf"
            "username": "your-username",  # e.g., "umut"
            "name": "Your Name"           # e.g., "Umut"
        })
    await engine.dispose()
    print("✅ User created successfully!")

asyncio.run(main())
</span><span class="no">EOF
</span></code></pre>

</div>



<p><strong>Example with actual values:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python3 <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
import asyncio
from sqlalchemy import text
from sqlalchemy.ext.asyncio import create_async_engine

DATABASE_URL = "postgresql+asyncpg://postgres:postgres@localhost:5480/verceldb"
engine = create_async_engine(DATABASE_URL)

async def main():
    async with engine.begin() as conn:
        await conn.execute(text("""
            INSERT INTO comfyui_deploy.users (id, username, name, created_at, updated_at)
            VALUES (:id, :username, :name, now(), now())
            ON CONFLICT (id) DO NOTHING
        """), {
            "id": "user_12345679",
            "username": "umut",
            "name": "Umut"
        })
    await engine.dispose()
    print("✅ User created successfully!")

asyncio.run(main())
</span><span class="no">EOF
</span></code></pre>

</div>



<p>✓ <strong>Verify User Creation</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>psql postgresql://postgres:postgres@localhost:5480/verceldb

<span class="c"># In psql:</span>
SELECT <span class="nb">id</span>, username, name FROM comfyui_deploy.users<span class="p">;</span>

<span class="c"># You should see your user record</span>
<span class="c"># Exit psql with: \q</span>
</code></pre>

</div>



<p><strong>After creating the user</strong>, refresh your browser at <a href="http://localhost:3001" rel="noopener noreferrer">http://localhost:3001</a>. The foreign key errors should be gone and you should be able to use the application normally.</p>

<h2>
  
  
  Step 6.5: Attach an Autumn Plan to Your User or Organization (CRITICAL)
</h2>

<p><strong>⚠️ CRITICAL STEP:</strong> After creating your user record, you <strong>must</strong> attach a subscription plan in the Autumn dashboard. Without a plan, you'll hit feature limits and won't be able to create machines or run workflows.</p>

<p><strong>Why is this needed?</strong></p>

<p>ComfyDeploy uses Autumn for billing and feature limits. Even in local development, the API checks Autumn to determine:</p>

<ul>
<li>How many machines you can create (<code>machine_limit</code>)</li>
<li>How many concurrent GPU jobs you can run (<code>gpu_concurrency_limit</code>)</li>
<li>How many workflows you can have (<code>workflow_limit</code>)</li>
<li>How many GPU credits you have available (<code>gpu-credit</code>)</li>
</ul>

<p>Without a plan attached, you'll be on the default "Free" tier with very limited features.</p>

<h3>
  
  
  Access the Autumn Dashboard
</h3>

<ol>
<li>Go to <a href="https://dashboard.autumn.dev/" rel="noopener noreferrer">https://dashboard.autumn.dev/</a> (or <a href="https://app.useautumn.com/" rel="noopener noreferrer">https://app.useautumn.com/</a>)</li>
<li>Log in with your Autumn account</li>
<li>Navigate to <strong>Customers</strong> in the sidebar</li>
</ol>

<h3>
  
  
  Attach a Plan to Your Organization
</h3>

<p><strong>For Organization-based Development</strong> (Recommended):</p>

<ol>
<li>In the Autumn dashboard, go to <strong>Customers</strong>
</li>
<li>Find or create a customer for your Clerk organization ID (e.g., <code>org_123456789</code>)

<ul>
<li>If it doesn't exist, click <strong>"Create Customer"</strong>
</li>
<li>Enter your Clerk organization ID as the customer ID</li>
</ul>
</li>
<li>Click on the customer to view details</li>
<li>Click <strong>"Attach Plan"</strong> or <strong>"Add Subscription"</strong>
</li>
<li>Select <strong>"Business (Monthly)"</strong> plan</li>
<li>Verify the plan includes:

<ul>
<li>
<strong>GPU Concurrency Limit</strong>: 10</li>
<li>
<strong>Machine Limit</strong>: 25</li>
<li>
<strong>Workflow Limit</strong>: 300</li>
<li>
<strong>Seats</strong>: 10</li>
<li>
<strong>GPU Credit</strong>: $10,000 per month (100,000 credits)</li>
<li>
<strong>Price</strong>: $998 per month</li>
</ul>
</li>
<li>Click <strong>"Create"</strong> or <strong>"Attach"</strong>
</li>
</ol>

<p><strong>For User-based Development</strong>:</p>

<ol>
<li>In the Autumn dashboard, go to <strong>Customers</strong>
</li>
<li>Find or create a customer for your Clerk user ID (e.g., <code>user_123456789</code>)</li>
<li>Follow the same steps as above to attach the Business plan</li>
</ol>

<h3>
  
  
  Example Plan Configuration
</h3>

<p>Here's what the Business (Monthly) plan should look like in Autumn:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Business (Monthly)
├── GPU Concurrency Limit: 10
├── Machine Limit: 25
├── Workflow Limit: 300
├── Seats: 10
├── GPU Credit: $10,000 per month (100,000 credits)
└── Price: $998 per month
</code></pre>

</div>



<p><strong>Feature Items Breakdown:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Included Usage</th>
<th>Price</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>gpu_concurrency_limit</code></td>
<td>10</td>
<td>Feature</td>
</tr>
<tr>
<td><code>machine_limit</code></td>
<td>25</td>
<td>Feature</td>
</tr>
<tr>
<td><code>workflow_limit</code></td>
<td>300</td>
<td>Feature</td>
</tr>
<tr>
<td><code>seats</code></td>
<td>10</td>
<td>Feature</td>
</tr>
<tr>
<td><code>gpu-credit</code></td>
<td>$10,000 per GPU Credit (cents) per month</td>
<td>Feature</td>
</tr>
<tr>
<td><strong>Total</strong></td>
<td></td>
<td><strong>$998 per month</strong></td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Verify the Plan is Active
</h3>

<ol>
<li>Refresh your browser at <a href="http://localhost:3001" rel="noopener noreferrer">http://localhost:3001</a>
</li>
<li>Navigate to <strong>Settings</strong> → <strong>Billing</strong> (or <strong>Usage</strong>)</li>
<li>You should see:

<ul>
<li>Current plan: <strong>Business</strong>
</li>
<li>Machine limit: <strong>25</strong>
</li>
<li>GPU concurrency: <strong>10</strong>
</li>
<li>Available credits: <strong>100,000 credits</strong> ($10,000)</li>
</ul>
</li>
</ol>

<p><strong>Check via API:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-H</span> <span class="s2">"Authorization: Bearer YOUR_CLERK_TOKEN"</span> <span class="se">\</span>
  http://localhost:3011/api/platform/plan
</code></pre>

</div>



<p>You should see a response with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"plan"</span><span class="p">:</span><span class="w"> </span><span class="s2">"business"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"features"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"machine_limit"</span><span class="p">:</span><span class="w"> </span><span class="mi">25</span><span class="p">,</span><span class="w">
    </span><span class="nl">"gpu_concurrency_limit"</span><span class="p">:</span><span class="w"> </span><span class="mi">10</span><span class="p">,</span><span class="w">
    </span><span class="nl">"workflow_limit"</span><span class="p">:</span><span class="w"> </span><span class="mi">300</span><span class="p">,</span><span class="w">
    </span><span class="nl">"gpu_credit"</span><span class="p">:</span><span class="w"> </span><span class="mi">100000</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Common Issues
</h3>

<p><strong>Error: "Max Machines Exceeded" despite attaching a plan</strong></p>

<p>This usually means:</p>

<ol>
<li>The plan wasn't attached to the correct customer ID (check if you're using user ID vs organization ID)</li>
<li>The Autumn cache needs to be refreshed - wait 30 seconds and try again</li>
<li>The <code>autumn.config.ts</code> wasn't pushed - run <code>npx atmn push</code> again</li>
</ol>

<p><strong>Error: "Insufficient GPU credits"</strong></p>

<p>The Business plan includes $10,000 in GPU credits (100,000 credits). If you see this error:</p>

<ol>
<li>Check the Autumn dashboard to verify credits were added</li>
<li>Make sure the <code>gpu-credit</code> feature is included in the plan</li>
<li>Try refreshing the page or logging out and back in</li>
</ol>

<p><strong>Plan not showing in the frontend</strong></p>

<ol>
<li>Clear browser cache and hard refresh (Cmd+Shift+R or Ctrl+Shift+R)</li>
<li>Check that <code>AUTUMN_SECRET_KEY</code> is correct in <code>apps/api/.env</code>
</li>
<li>Restart the API server to pick up the latest Autumn data</li>
</ol>

<h2>
  
  
  Step 7: Access the App
</h2>

<p>Open <a href="http://localhost:3001" rel="noopener noreferrer">http://localhost:3001</a> in your browser. You should be able to log in with Clerk and start creating workflows.</p>

<p>✓ <strong>Verify Full Stack</strong>:</p>

<ol>
<li>Log in with your Clerk credentials</li>
<li>Navigate to "Machines" → you should see an empty list</li>
<li>Try creating a new workflow</li>
<li>Check that the API is receiving requests (watch the API server logs)</li>
</ol>

<h2>
  
  
  Troubleshooting Guide
</h2>

<h3>
  
  
  Database Issues
</h3>

<p><strong>Error: <code>relation "comfyui_deploy.machines" does not exist</code></strong></p>

<p>This means migrations didn't run or failed. Fix it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>apps/api
bun run migrate-local
</code></pre>

</div>



<p>Check the output for any errors. If migrations are stuck, you may need to reset:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># WARNING: This deletes all data!</span>
docker-compose down <span class="nt">-v</span>
docker-compose up <span class="nt">-d</span>
bun run migrate-local
</code></pre>

</div>



<p><strong>Error: <code>ForeignKeyViolationError: Key (user_id)=(...) is not present in table "users"</code></strong></p>

<p><strong>This is the most common error in local development.</strong> The user record doesn't exist in the database. This happens when:</p>

<ul>
<li>You restarted Docker (which wipes the database)</li>
<li>You're using a new Clerk user ID that hasn't been added to the database</li>
<li>You skipped Step 6 after first login</li>
</ul>

<p><strong>Fix:</strong> Run Step 6 to create the user record. This is a <strong>required step</strong> after every Docker restart or first login.</p>

<p><strong>Error: <code>500 Internal Server Error on /api/volume/private-models</code></strong></p>

<p>Usually caused by missing user record. Check:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Get your Clerk user ID from the browser console:</span>
<span class="c"># Open DevTools → Application → Cookies → find __clerk_db_jwt</span>
<span class="c"># Decode it to see your user_id</span>

<span class="c"># Then verify the user exists:</span>
psql postgresql://postgres:postgres@localhost:5480/verceldb
SELECT <span class="k">*</span> FROM comfyui_deploy.users WHERE <span class="nb">id</span> <span class="o">=</span> <span class="s1">'your-user-id'</span><span class="p">;</span>
</code></pre>

</div>



<p>If the user doesn't exist, create it (Step 6).</p>

<h3>
  
  
  Port and Process Issues
</h3>

<p><strong>Error: <code>Address already in use</code> on port 3011</strong></p>

<p>Something is already running on that port. Kill it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>lsof <span class="nt">-ti</span>:3011 | xargs <span class="nb">kill</span> <span class="nt">-9</span>
</code></pre>

</div>



<p>Or find what's using it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>lsof <span class="nt">-i</span> :3011
</code></pre>

</div>



<p><strong>Error: <code>Address already in use</code> on port 3001 (frontend)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>lsof <span class="nt">-ti</span>:3001 | xargs <span class="nb">kill</span> <span class="nt">-9</span>
</code></pre>

</div>



<p><strong>Error: ngrok tunnel keeps disconnecting</strong></p>

<p>ngrok free tier has connection limits. If you're testing heavily:</p>

<ul>
<li>Upgrade to ngrok pro</li>
<li>Or restart ngrok: <code>ngrok http 3011</code>
</li>
</ul>

<h3>
  
  
  Modal and Workflow Issues
</h3>

<p><strong>Runs stuck in "not-started" status</strong></p>

<p>This means Modal functions can't send status updates back to your API. Check:</p>

<ol>
<li>Is ngrok running?
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   ps aux | <span class="nb">grep </span>ngrok
</code></pre>

</div>



<ol>
<li>Is the ngrok URL in your <code>.env</code>?
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">grep </span>CURRENT_API_URL apps/api/.env
</code></pre>

</div>



<ol>
<li>Did you restart the API server after updating <code>.env</code>?
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="c"># Kill the old server and restart it</span>
   lsof <span class="nt">-ti</span>:3011 | xargs <span class="nb">kill</span> <span class="nt">-9</span>
   <span class="nv">PORT</span><span class="o">=</span>3011 uv run src/api/server.py
</code></pre>

</div>



<ol>
<li>Test the ngrok tunnel:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   curl https://your-ngrok-url.ngrok-free.app/api/health
</code></pre>

</div>



<p><strong>Error: "Max Machines Exceeded" despite having a paid plan</strong></p>

<p>This is an Autumn billing configuration issue. The most common cause is <strong>not attaching a plan to your user or organization in the Autumn dashboard</strong>.</p>

<p><strong>Fix:</strong></p>

<ol>
<li>
<p><strong>🔴 MOST COMMON:</strong> Did you attach a plan in the Autumn dashboard? (See Step 6.5)</p>

<ul>
<li>Go to <a href="https://dashboard.autumn.dev/" rel="noopener noreferrer">https://dashboard.autumn.dev/</a>
</li>
<li>Navigate to <strong>Customers</strong>
</li>
<li>Find your user ID or organization ID</li>
<li>Click <strong>"Attach Plan"</strong> → Select <strong>"Business (Monthly)"</strong>
</li>
<li>Verify the plan includes <code>machine_limit: 25</code>
</li>
</ul>
</li>
<li><p>Did you push your config to Autumn?<br>
</p></li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">cd </span>apps/api
   npx atmn push
</code></pre>

</div>



<ol>
<li>Is the <code>machineLimit</code> feature defined in your product?
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">grep</span> <span class="nt">-A</span> 5 <span class="s2">"machineLimit"</span> autumn.config.ts
</code></pre>

</div>



<ol>
<li>
<p>Are you using the correct customer ID?</p>

<ul>
<li>Check if you're logged in as a user or organization</li>
<li>The plan must be attached to the correct customer ID (user ID vs org ID)</li>
<li>Check browser console: <code>window.Clerk.user.id</code> or <code>window.Clerk.organization.id</code>
</li>
</ul>
</li>
<li><p>Wait 30 seconds for Autumn cache to refresh, then try again</p></li>
</ol>

<p><strong>Error: "GET /api/latest-hashes 404 Not Found" or "Illegal header value b'Bearer '"</strong></p>

<p>This means the <code>GITHUB_TOKEN</code> is missing or empty in your <code>.env</code> file. The API needs this token to fetch ComfyUI version information from GitHub.</p>

<p><strong>Fix:</strong></p>

<ol>
<li>Create a GitHub classic token at <a href="https://github.com/settings/tokens" rel="noopener noreferrer">https://github.com/settings/tokens</a>
</li>
<li>Select <code>public_repo</code> scope</li>
<li>Add to <code>apps/api/.env</code>:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nv">GITHUB_TOKEN</span><span class="o">=</span><span class="s2">"ghp_your_token_here"</span>
</code></pre>

</div>



<ol>
<li>Restart the API server</li>
</ol>

<p><strong>Error: "Invalid Volume name: ''" or "Can't find Volume: Invalid Volume name: ''"</strong></p>

<p>This means the Modal volume environment variables are missing or empty. Modal requires valid volume names to create persistent storage for models.</p>

<p><strong>Symptoms:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ERROR:api.middleware.authMiddleware:Can't find Volume: Invalid Volume name: ''.
Names may contain only alphanumeric characters, dashes, periods, and underscores,
must be shorter than 64 characters, and cannot conflict with App ID strings.
</code></pre>

</div>



<p><strong>Fix:</strong></p>

<ol>
<li>Add volume names to <code>apps/api/.env</code>:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nv">SHARED_MODEL_VOLUME_NAME</span><span class="o">=</span><span class="s2">"comfydeploy-dev-public-models"</span>
   <span class="nv">PUBLIC_MODEL_VOLUME_NAME</span><span class="o">=</span><span class="s2">"comfydeploy-dev-public-models"</span>
</code></pre>

</div>



<ol>
<li>Restart the API server</li>
<li>The volume will be automatically created in Modal on first use</li>
</ol>

<p><strong>Why this happens:</strong></p>

<ul>
<li>The code looks for <code>PUBLIC_MODEL_VOLUME_NAME</code> or <code>SHARED_MODEL_VOLUME_NAME</code> environment variables</li>
<li>If both are empty, it returns an empty string <code>""</code>
</li>
<li>Modal rejects empty volume names with a validation error</li>
<li>This affects both the <code>/api/volume/public-models</code> endpoint and serverless deployments</li>
</ul>

<p><strong>Error: Model downloads fail or "Function not found" errors</strong></p>

<p>This usually means you forgot to deploy the <code>volume-operations</code> Modal app, which is <strong>required</strong> for model management.</p>

<p><strong>Symptoms:</strong></p>

<ul>
<li>Model downloads fail silently or with errors</li>
<li>"Function not found" errors when trying to download models</li>
<li>Models don't appear in Modal volumes</li>
</ul>

<p><strong>Fix:</strong></p>

<ol>
<li>Deploy the volume-operations app:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">cd </span>apps/api
   modal deploy src/modal_apps/modal_downloader.py::modal_downloader_app
</code></pre>

</div>



<ol>
<li>Verify deployment:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   modal app list
   <span class="c"># Should show "volume-operations" in the list</span>
</code></pre>

</div>



<ol>
<li>Check the Modal dashboard at <a href="https://modal.com/apps" rel="noopener noreferrer">https://modal.com/apps</a>
</li>
</ol>

<p><strong>Why this is required:</strong></p>

<ul>
<li>The <code>volume-operations</code> app contains Modal functions for downloading models</li>
<li>Without it, the API can't download models to Modal volumes</li>
<li>This is a one-time deployment per Modal environment</li>
<li>See Step 5.4 in the setup guide for details</li>
</ul>

<h3>
  
  
  Service Health Checks
</h3>

<p><strong>Check if PostgreSQL is running:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>psql postgresql://postgres:postgres@localhost:5480/verceldb <span class="nt">-c</span> <span class="s2">"SELECT 1"</span>
<span class="c"># Expected: 1</span>
</code></pre>

</div>



<p><strong>Check if Redis is running:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>redis-cli <span class="nt">-p</span> 6379 ping
<span class="c"># Expected: PONG</span>
</code></pre>

</div>



<p><strong>Check if API server is responding:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:3011/api/health
<span class="c"># Expected: 200 OK</span>
</code></pre>

</div>



<p><strong>Check if frontend is running:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:3001
<span class="c"># Expected: HTML response</span>
</code></pre>

</div>



<p><strong>Check API server logs for errors:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># The API server logs to stdout, so check your terminal</span>
<span class="c"># Look for ERROR or EXCEPTION messages</span>
</code></pre>

</div>



<p><strong>Check Docker container logs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker-compose logs postgres    <span class="c"># PostgreSQL logs</span>
docker-compose logs redis       <span class="c"># Redis logs</span>
docker-compose logs             <span class="c"># All services</span>
</code></pre>

</div>



<h3>
  
  
  React Query Caching Issues
</h3>

<p><strong>Settings page shows "disabled for free tier" despite having a paid plan</strong></p>

<p>This is a React Query cache issue. The frontend cached stale plan data. Fix:</p>

<ol>
<li>Clear browser cache: DevTools → Application → Clear Storage</li>
<li>Or hard refresh: Ctrl+Shift+R (Windows/Linux) or Cmd+Shift+R (Mac)</li>
<li>Or log out and log back in</li>
</ol>

<p>If this keeps happening, check that your <code>.env</code> has the correct <code>AUTUMN_SECRET_KEY</code>.</p>

<h2>
  
  
  That's It!
</h2>

<p>You're now ready to develop. The key insights:</p>

<ol>
<li><p><strong>🔴 CRITICAL: Create user record after first login (Step 6)</strong> - Without this, you'll get foreign key errors on every API call. This must be done after every Docker restart.</p></li>
<li><p><strong>🔴 CRITICAL: Attach Autumn plan (Step 6.5)</strong> - Without a subscription plan attached in the Autumn dashboard, you'll hit feature limits and won't be able to create machines or run workflows. Attach the "Business (Monthly)" plan to your user or organization.</p></li>
<li><p><strong>🔴 CRITICAL: ngrok must be running (Step 5.5)</strong> - Modal functions need to communicate back to your local API. Without ngrok, workflows will get stuck in "not-started" status.</p></li>
<li>
<p><strong>Fresh Docker restarts wipe data</strong> - You'll need to:</p>

<ul>
<li>Re-run migrations (<code>bun run migrate-local</code>)</li>
<li>Recreate user records (Step 6)</li>
<li>Verify Autumn plan is still attached (Step 6.5)</li>
</ul>
</li>
<li><p><strong>Autumn config must be pushed</strong> - Changes to <code>autumn.config.ts</code> don't take effect until you run <code>npx atmn push</code></p></li>
<li><p><strong>Check logs first</strong> - Most issues are visible in the API server logs or Docker logs</p></li>
</ol>

<h2>
  
  
  Quick Start Checklist
</h2>

<p>After cloning the repo, here's the minimal path to get running:</p>

<p><strong>Prerequisites:</strong></p>

<ul>
<li>[ ] Install all required software (Node.js, Bun, Python 3.12, uv, Docker, AWS CLI, ngrok, psql)</li>
<li>[ ] Create accounts and get API keys (Clerk, Autumn, AWS S3, Modal, ngrok, GitHub)</li>
</ul>

<p><strong>Setup:</strong></p>

<ul>
<li>[ ] Initialize git submodules: <code>git submodule init &amp;&amp; git submodule update</code>
</li>
<li>[ ] Install dependencies (API: <code>bun install</code> + <code>uv sync</code>, Frontend: <code>bun install</code>)</li>
<li>[ ] Configure <code>.env</code> files with all API keys (Clerk, Autumn, AWS S3, Modal, ngrok, <strong>GitHub</strong>, <strong>Modal Volumes</strong>)</li>
<li>[ ] Push Autumn config: <code>npx atmn push</code>
</li>
<li>[ ] Start Docker: <code>docker compose up -d</code>
</li>
<li>[ ] Run migrations: <code>bun run migrate-local</code>
</li>
<li>[ ] <strong>🔴 Install and configure ngrok: <code>brew install ngrok/ngrok/ngrok</code> + <code>ngrok config add-authtoken YOUR_TOKEN</code></strong>
</li>
<li>[ ] <strong>🔴 Start ngrok: <code>ngrok http 3011</code></strong> (keep running)</li>
<li>[ ] <strong>🔴 Update <code>.env</code> with ngrok URL and Modal volume names</strong>
</li>
<li>[ ] Start API server: <code>bun run dev</code>
</li>
<li>[ ] Start frontend: <code>bun run dev</code>
</li>
<li>[ ] <strong>🔴 Log in via frontend, then create user record in database (Step 6)</strong>
</li>
<li>[ ] <strong>🔴 Attach Business plan to your user/org in Autumn dashboard (Step 6.5)</strong>
</li>
<li>[ ] Verify everything works by creating a test workflow</li>
</ul>

<p>Happy coding! 🚀</p>

