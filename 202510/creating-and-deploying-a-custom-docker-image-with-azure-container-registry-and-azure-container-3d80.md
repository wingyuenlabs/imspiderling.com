---
Title: Creating and Deploying a Custom Docker Image with Azure Container Registry and Azure Container Instances
Description: 
Author: Theodora
Date: 2025-10-20T21:49:43.000Z
Robots: noindex,nofollow
Template: index
---
<p>Table of Contents</p>

<ul>
<li>Prerequisites</li>
<li>Step 1: Create a Simple Dockerfile and Shell Script</li>
<li>Step 2: Build the Docker Image</li>
<li>Step 3: Set Up Azure Container Registry</li>
<li>Step 4: Push Docker Image to ACR</li>
<li>Step 5: Deploy to Azure Container Instances</li>
<li>Common Errors and Fixes</li>
<li>Final Checks</li>
</ul>

<h2>
  
  
  Prerequisites
</h2>

<p>Make sure you have the following installed:</p>

<ul>
<li>✔️ An <a href="https://portal.azure.com/" rel="noopener noreferrer">Azure account</a> (you can create one for free)</li>
<li>✔️ <a href="https://learn.microsoft.com/en-us/cli/azure/install-azure-cli" rel="noopener noreferrer">Azure CLI</a>
</li>
<li>✔️ <a href="https://www.docker.com/products/docker-desktop" rel="noopener noreferrer">Docker Desktop</a>
</li>
<li>✔️ A terminal or PowerShell window</li>
</ul>

<h2>
  
  
  Step 1: Create a Simple Dockerfile and Shell Script
</h2>

<p>Create a new directory, then inside it:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb1v5ck8ntbyhasjn1u6n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb1v5ck8ntbyhasjn1u6n.png" alt=" " width="563" height="138"></a></p>

<p>This is a minimal Dockerfile using the nginx base image and a custom shell script.</p>

<h3>
  
  
  <strong><code>FROM nginx:latest</code></strong>
</h3>

<p>This tells Docker to <strong>use the latest version of the official <code>nginx</code> image</strong> from Docker Hub as the base image.</p>

<ul>
<li>The base image already includes Nginx installed and configured.</li>
<li>It's lightweight and typically used for serving web content.</li>
</ul>

<h3>
  
  
  <strong><code>ARG VERSION</code></strong>
</h3>

<p>This declares a <strong>build-time argument</strong> named <code>VERSION</code>.</p>

<ul>
<li>
<code>ARG</code> is only available during the image <strong>build process</strong>.</li>
<li>You can pass it using <code>-build-arg</code> like this:</li>
</ul>

<p><strong>docker build --build-arg VERSION=1.2.3</strong><br>
 ### <strong><code>ENV VERSION=$VERSION</code></strong></p>

<p>This sets an <strong>environment variable inside the container</strong> using the value passed during build time.</p>

<ul>
<li>
<code>ENV</code> is available at <strong>runtime</strong> — your script or Nginx can access it.</li>
<li>You can verify it with:
<strong>echo $VERSION</strong>
</li>
</ul>

<h3>
  
  
  <strong><code>COPY hostname.sh .</code></strong>
</h3>

<p>This <strong>copies the <code>hostname.sh</code> script from your local directory into the image's working directory</strong> (which is <code>/</code> by default unless changed).</p>

<ul>
<li>If you want it in a specific directory like <code>/usr/local/bin</code>, you should write:
<strong>COPY hostname.sh /usr/local/bin/</strong>
</li>
</ul>

<h3>
  
  
  <strong><code>CMD ["/hostname.sh"]</code></strong>
</h3>

<p>This sets the <strong>default command that runs when the container starts</strong>.</p>

<ul>
<li>It overrides Nginx’s default command.</li>
<li>So instead of starting Nginx, the container will <strong>run the <code>hostname.sh</code> script</strong>.</li>
</ul>

<p><strong>Note</strong> If the script isn't executable or doesn't start with a shebang (#!/bin/bash or similar), the container will fail.</p>

<p>This Dockerfile creates a container that:</p>

<ol>
<li>Uses the latest Nginx image.</li>
<li>Accepts a version number at build time.</li>
<li>Sets that version as an environment variable.</li>
<li>Copies a script called <code>hostname.sh</code> into the container.</li>
<li>Runs that script instead of Nginx when the container starts</li>
</ol>

<h3>
  
  
  Script Breakdown: hostname.sh
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3x5t5py0cho3rbe9l8n1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3x5t5py0cho3rbe9l8n1.png" alt=" " width="800" height="438"></a></p>

<h3>
  
  
  <code>#!/bin/sh</code>
</h3>

<ul>
<li>This line tells the system to use the <strong><code>sh</code> shell</strong> to run the script.</li>
</ul>

<h3>
  
  
  <code>HOSTNAME=\</code>hostname``
</h3>

<ul>
<li>Runs the <code>hostname</code> command, capturing the <strong>machine/container name</strong>.</li>
<li>Stores it in the variable <code>HOSTNAME</code>.</li>
</ul>

<h3>
  
  
  <code>VERSION=${VERSION:-v1}</code>
</h3>

<ul>
<li>Reads the <code>VERSION</code> environment variable if it’s set.</li>
<li>If not, it defaults to <code>v1</code>.</li>
</ul>

<p>Creates a landing HTML page for the root of the Nginx server:</p>

<p><strong>cat &gt; /usr/share/nginx/html/index.html &lt;&lt;EOF<br>
...<br>
EOF</strong></p>

<ul>
<li>This file will be served when someone visits <code>/</code> in the browser.</li>
<li>It includes:

<ul>
<li>Writes an HTML file directly to the default Nginx directory.</li>
<li>The hostname (of the container or VM)</li>
<li>The version number</li>
</ul>


</li>

</ul>

<h3>
  
  
  <code>mkdir /usr/share/nginx/html/healthz /usr/share/nginx/html/hostname /usr/share/nginx/html/version</code>
</h3>

<ul>
<li>Creates <strong>3 subdirectories</strong> inside Nginx's web root:

<ul>
<li>
<code>/healthz</code> – for liveness/readiness checks</li>
<li>
<code>/hostname</code> – for showing the container’s hostname and version</li>
<li>
<code>/version</code> – for showing the version only</li>
</ul>


</li>

</ul>

<p>Writes mini HTML/text responses to each endpoint:<br>
<strong>cat &gt; /usr/share/nginx/html/hostname/index.html &lt;&lt;EOF<br>
$HOSTNAME -- $VERSION<br>
EOF</strong></p>

<ul>
<li>Shows: <code>hostname -- version</code>
</li>
<li>Endpoint: <code>/hostname</code>
</li>
</ul>

<p><strong>cat &gt; /usr/share/nginx/html/version/index.html &lt;&lt;EOF<br>
$VERSION<br>
EOF</strong></p>

<ul>
<li>Endpoint: <code>/version</code>
</li>
<li>Shows: version only</li>
</ul>

<p><strong>chmod 777 /usr/share/nginx/html/healthz<br>
cat &gt; /usr/share/nginx/html/healthz/index.html &lt;&lt;EOF<br>
healthy<br>
EOF</strong></p>

<ul>
<li>Grants all permissions to <code>/healthz</code> directory (not ideal for production, but okay for dev/demo).</li>
<li>Writes a file that says <code>healthy</code> to <code>/healthz/index.html</code>
</li>
<li>This is useful for Kubernetes or Docker health checks.</li>
</ul>

<h3>
  
  
  <code>nginx -g "daemon off;"</code>
</h3>

<ul>
<li>Starts Nginx in <strong>foreground mode</strong> (so the container doesn't exit).</li>
<li>This is required in Docker — if the main process exits, the container stops.</li>
</ul>

<p>When this script runs inside a Docker container, it:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpukud4mdguboshooiknj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpukud4mdguboshooiknj.png" alt=" " width="484" height="262"></a></p>

<p>If you access the container in a browser (or curl):</p>

<ul>
<li>
<code>/</code> → HTML page showing the hostname and version</li>
<li>
<code>/healthz</code> → returns <code>healthy</code>
</li>
<li>
<code>/hostname</code> → returns <code>hostname -- version</code>
</li>
<li>
<code>/version</code> → returns the version string</li>
</ul>

<h2>
  
  
  Step 2: Build the Docker Image
</h2>

<p>Open PowerShell in the folder and run:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftl8ogssurg5yhd732lpd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftl8ogssurg5yhd732lpd.png" alt=" " width="517" height="103"></a></p>

<p>Output</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi00g4fvj6585cyuirxaq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi00g4fvj6585cyuirxaq.png" alt=" " width="526" height="150"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Femdz5bnuuuipf0vr52kj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Femdz5bnuuuipf0vr52kj.png" alt=" " width="800" height="280"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdtwpsdki2v2awvqa05a.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdtwpsdki2v2awvqa05a.png" alt=" " width="587" height="569"></a></p>

<h2>
  
  
  Step 3: Set Up Azure Container Registry
</h2>

<p>In the <a href="https://portal.azure.com/" rel="noopener noreferrer">Azure Portal</a>:</p>

<ol>
<li>Search for <strong>Container Registry</strong>
</li>
<li>Click <strong>Create</strong>
</li>
<li>Choose a name (mine was <code>hostcr</code>)</li>
<li>Set the resource group, region, SKU, etc.</li>
<li>Enable <strong>Admin User</strong> under Access Keys after creation</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbdqhf1iph3xrfktpngsf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbdqhf1iph3xrfktpngsf.png" alt=" " width="800" height="739"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Focnv7z4a5hg1paztnehl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Focnv7z4a5hg1paztnehl.png" alt=" " width="463" height="840"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5nobkk2kknmny91rqh0t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5nobkk2kknmny91rqh0t.png" alt=" " width="483" height="321"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn8phn9l1lwwjw89cqi0a.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn8phn9l1lwwjw89cqi0a.png" alt=" " width="707" height="372"></a></p>

<h2>
  
  
  Step 4: Push Docker Image to ACR
</h2>

<p>1.Authenticate with Azure:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fci53fmfeya5iur71p7sb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fci53fmfeya5iur71p7sb.png" alt=" " width="521" height="98"></a></p>

<ol>
<li>Tag Your Docker Image</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxvsvffbfpxhnh4t01u9s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxvsvffbfpxhnh4t01u9s.png" alt=" " width="463" height="100"></a></p>

<ol>
<li>Log in to ACR</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3j8w4qeqht0w4mpt2eg8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3j8w4qeqht0w4mpt2eg8.png" alt=" " width="401" height="110"></a></p>

<p>Output</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu9dki566614pr43l786p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu9dki566614pr43l786p.png" alt=" " width="302" height="93"></a></p>

<ol>
<li>Push the Image to ACR</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4j4roirkk6noiq08tzho.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4j4roirkk6noiq08tzho.png" alt=" " width="421" height="102"></a></p>

<p>Output</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpg77mcr4odv427zsocbp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpg77mcr4odv427zsocbp.png" alt=" " width="700" height="143"></a><br>
 <strong>boops</strong>-I lost my screenshot of the Azure intercae for this</p>

<h2>
  
  
  Step 5: Deploy to Azure Container Instances
</h2>

<ol>
<li>In the Azure portal, search for <strong>Container Instances</strong>
</li>
<li>Click <strong>Create</strong>
</li>
<li>Fill in:

<ul>
<li>Name</li>
<li>Resource Group</li>
<li>Image source: Azure Container Registry</li>
<li>Image: <code>hostcr.azurecr.io/hostname:v1</code>
</li>
</ul>
</li>
<li>Review and create</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0w0eqn6bbw6a8xewox2h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0w0eqn6bbw6a8xewox2h.png" alt=" " width="663" height="744"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjs8tmg81r5p92fopw2de.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjs8tmg81r5p92fopw2de.png" alt=" " width="601" height="762"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcorlbozehra90udq6cfy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcorlbozehra90udq6cfy.png" alt=" " width="675" height="295"></a></p>

<h2>
  
  
  Common Errors and Fixes
</h2>

<h3>
  
  
  ❌ <code>docker push</code> returns "authentication required"
</h3>

<p><strong>Fix:</strong> Make sure you're logged into ACR:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmf1a6qmyx84vrggun0q0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmf1a6qmyx84vrggun0q0.png" alt=" " width="366" height="101"></a></p>

<h3>
  
  
  ❌ <code>az login</code> fails due to tenant mismatch
</h3>

<p><strong>Fix:</strong> Use the right tenant ID explicitly:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu06yqozgxyxw67ndr2n3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu06yqozgxyxw67ndr2n3.png" alt=" " width="405" height="110"></a></p>

<h2>
  
  
  ✅ Final Checks
</h2>

<ol>
<li>Go to your deployed <strong>Container Instance</strong>
</li>
<li>Copy the <strong>public IP</strong>
</li>
<li>Visit in browser:</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7hcmoktfrmpx8bellxn2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7hcmoktfrmpx8bellxn2.png" alt=" " width="440" height="157"></a></p>

<p>You should see HTML responses with hostname, version, and health check status.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpu9hkvr6gk62l8w69a8w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpu9hkvr6gk62l8w69a8w.png" alt=" " width="694" height="267"></a></p>

