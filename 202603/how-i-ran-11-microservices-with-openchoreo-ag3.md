---
Title: 🚀 How I Ran 11 Microservices with OpenChoreo
Description: 
Author: Limani Ndou
Date: 2026-03-25T22:05:07.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>I used OpenChoreo to deploy and manage the Google Cloud microservices demo (11 services) on my local machine. The result was a cleaner developer workflow, better service visibility, and a production-like platform experience without building custom platform tooling from scratch.</p>
</blockquote>

<p>1️⃣ <strong>What OpenChoreo?</strong><br>
<strong>OpenChoreo</strong> is a developer platform for <strong>Kubernetes</strong> offering development and architecture abstractions, a Backstage-powered developer portal, application CI/CD, GitOps, and observability. It composes <strong>CNCF</strong> and complementary projects into a modular, extensible framework that provides both strong governance for platform engineers and simplified abstractions for developers.</p>

<p><em>OpenChoreo was developed by WSO2 and is now part of the Cloud Native Computing Foundation (CNCF) ecosystem.</em></p>

<p>2️⃣ <strong>Why OpenChoreo?</strong><br>
OpenChoreo it provides all essential building blocks of an Internal Developer Platform (IDP) :</p>

<p>High-level APIs for modeling cloud-native applications<br>
A Control Plane that understands and enforces these APIs with GitOps support<br>
A built-in workflow engine for CI/CD pipelines and automation<br>
An opinionated Data Plane with runtime enforcement of design-time semantics<br>
Built-in security, networking, and observability integrations<br>
This is the key reason I used it for this 11-microservice demo.</p>

<p>3️⃣ <strong>GCP Microservices Demo: What It Includes</strong><br>
Repository: <a href="https://github.com/GoogleCloudPlatform/microservices-demo" rel="noopener noreferrer">https://github.com/GoogleCloudPlatform/microservices-demo</a></p>

<p>🌐 frontend (Go): Serves the website and auto-generates user session IDs.<br>
🛒 cartservice (C#): Stores/retrieves cart items in Redis.<br>
📚 productcatalogservice (Go): Product listing, search, and product details from JSON.<br>
💱 currencyservice (Node.js): Converts money values using ECB exchange rates (highest QPS).<br>
💳 paymentservice (Node.js): Mock credit card charge and transaction ID generation.<br>
🚚 shippingservice (Go): Shipping cost estimates and mock shipping execution.<br>
✉️ emailservice (Python): Mock order confirmation emails.<br>
🎯 checkoutservice (Go): Orchestrates cart, payment, shipping, and email flow.<br>
💡 recommendationservice (Python): Suggests products from cart context.<br>
📢 adservice (Java): Returns text ads based on context words.<br>
🔄 loadgenerator (Python/Locust): Simulates realistic shopping traffic.<br>
4️⃣ What I Did (Step by Step)<br>
Step 1: Start OpenChoreo Quick Start<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">--rm</span> <span class="nt">-it</span> <span class="nt">--name</span> openchoreo-quick-start <span class="sb">`</span>
  <span class="nt">--pull</span> always <span class="sb">`</span>
  <span class="nt">-v</span> //var/run/docker.sock:/var/run/docker.sock <span class="sb">`</span>
  <span class="nt">-p</span> 8080:8080 <span class="nt">-p</span> 9443:9443 <span class="sb">`</span>
  ghcr.io/openchoreo/quick-start:v1.0.0
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fug1zsmtasstswm3p1vq2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fug1zsmtasstswm3p1vq2.png" alt=" " width="800" height="387"></a></p>

<p>Step 2: Install OpenChoreo(two options):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./install.sh

</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmg20cvhsh9dqjy2ylfg0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmg20cvhsh9dqjy2ylfg0.png" alt=" " width="800" height="394"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F33ht2p8jm6efx3icd1oy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F33ht2p8jm6efx3icd1oy.png" alt=" " width="800" height="400"></a></p>

<p>With observability<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./install.sh <span class="nt">--with-build</span> <span class="nt">--with-observability</span>
</code></pre>

</div>



<p>Step 3: Choose a Deployment Option<br>
There are two ways to deploy the GCP Microservices Demo:</p>

<p>Option A: Use the built-in OpenChoreo deploy script for GCP Microservices Demo (11 services: frontend, cart, checkout, payment, and more)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./deploy-gcp-demo.sh
</code></pre>

</div>



<p>Option B: Clone and deploy manually<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone <span class="nt">--depth</span> 1 <span class="nt">--branch</span> v0 https://github.com/GoogleCloudPlatform/microservices-demo.git
<span class="nb">cd </span>microservices-demo
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> ./release/kubernetes-manifests.yaml
I used Option B <span class="o">(</span>clone and deploy<span class="o">)</span><span class="nb">.</span>
</code></pre>

</div>



<p>Step 4: Verify in OpenChoreo UI<br>
Access the UI at: <code>http://openchoreo.localhost:8080/</code></p>

<p>Default login credentials:</p>

<p>Username: <code>admin@openchoreo.dev</code><br>
Password: <code>Admin@123</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1frup8664uxz6h9fmj7e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1frup8664uxz6h9fmj7e.png" alt=" " width="800" height="400"></a></p>

<p>Check:</p>

<p>Project and component catalog<br>
Deployment pipeline view<br>
Environment progression</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkzq4d2vsaa9gtkfn6hug.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkzq4d2vsaa9gtkfn6hug.png" alt=" " width="800" height="398"></a></p>

<p>Step 5: Validate End-to-End App Behavior<br>
I opened the storefront and validated core user flow (browse, cart, checkout).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ec04p8gys9vauqgobu8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ec04p8gys9vauqgobu8.png" alt=" " width="800" height="401"></a></p>

<p>📚 <strong>References</strong><br>
Documentation: <a href="https://openchoreo.dev/docs/" rel="noopener noreferrer">https://openchoreo.dev/docs/</a><br>
Documentation: <a href="https://insights.linuxfoundation.org/project/openchore" rel="noopener noreferrer">https://insights.linuxfoundation.org/project/openchore</a><br>
OpenChoreo Quick Start Guide:<br>
<a href="https://openchoreo.dev/docs/getting-started/quick-start-guide/" rel="noopener noreferrer">https://openchoreo.dev/docs/getting-started/quick-start-guide/</a><br>
GCP Microservices Demo Repository:<br>
<a href="https://github.com/GoogleCloudPlatform/microservices-demo" rel="noopener noreferrer">https://github.com/GoogleCloudPlatform/microservices-demo</a></p>

