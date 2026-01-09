---
Title: Kubernetes Services & Networking
Description: 
Author: Aisalkyn Aidarova
Date: 2026-01-09T21:51:06.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Architecture Overview (Mental Model)
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuo2y2uuhxceq88x8yuhu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuo2y2uuhxceq88x8yuhu.png" alt="Image" width="761" height="461"></a></p>

<p><strong>Traffic flow:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Browser
  ↓
Ingress
  ↓
Service
  ↓
Pod
  ↓
Container
</code></pre>

</div>



<p>Everything in this material builds around this flow.</p>




<h1>
  
  
  MODULE 1 — Kubernetes Services &amp; Networking
</h1>

<h2>
  
  
  Why Services Exist
</h2>

<p>Pods:</p>

<ul>
<li>Have dynamic IPs</li>
<li>Can be recreated at any time</li>
<li>Must never be accessed directly</li>
</ul>

<p>A <strong>Service</strong> provides:</p>

<ul>
<li>Stable IP</li>
<li>Load balancing</li>
<li>Pod discovery</li>
</ul>




<h2>
  
  
  Service Types
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>Purpose</th>
<th>Production Usage</th>
</tr>
</thead>
<tbody>
<tr>
<td>ClusterIP</td>
<td>Internal access</td>
<td>Most common</td>
</tr>
<tr>
<td>NodePort</td>
<td>Direct node access</td>
<td>Debug / learning</td>
</tr>
<tr>
<td>LoadBalancer</td>
<td>Cloud LB</td>
<td>External traffic</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Project 1 — Service Traffic Flow
</h2>

<h3>
  
  
  Step 1 — Deployment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apps/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Deployment</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">web</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">replicas</span><span class="pi">:</span> <span class="m">2</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">matchLabels</span><span class="pi">:</span>
      <span class="na">app</span><span class="pi">:</span> <span class="s">web</span>
  <span class="na">template</span><span class="pi">:</span>
    <span class="na">metadata</span><span class="pi">:</span>
      <span class="na">labels</span><span class="pi">:</span>
        <span class="na">app</span><span class="pi">:</span> <span class="s">web</span>
    <span class="na">spec</span><span class="pi">:</span>
      <span class="na">containers</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">app</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">hashicorp/http-echo:0.2.3</span>
        <span class="na">args</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s2">"</span><span class="s">-listen=:8080"</span>
          <span class="pi">-</span> <span class="s2">"</span><span class="s">-text=SERVICE</span><span class="nv"> </span><span class="s">WORKS"</span>
        <span class="na">ports</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">containerPort</span><span class="pi">:</span> <span class="m">8080</span>
</code></pre>

</div>



<p>Apply:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> deployment.yaml
</code></pre>

</div>






<h3>
  
  
  Step 2 — ClusterIP Service
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Service</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">web-svc</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">app</span><span class="pi">:</span> <span class="s">web</span>
  <span class="na">ports</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">port</span><span class="pi">:</span> <span class="m">80</span>
    <span class="na">targetPort</span><span class="pi">:</span> <span class="m">8080</span>
</code></pre>

</div>



<p>Apply:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> service.yaml
</code></pre>

</div>



<p>Verify:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get svc
kubectl get endpoints web-svc
</code></pre>

</div>






<h3>
  
  
  Step 3 — Access Inside Cluster
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl run tmp <span class="nt">--rm</span> <span class="nt">-it</span> <span class="nt">--image</span><span class="o">=</span>busybox <span class="nt">--</span> sh
wget <span class="nt">-qO-</span> http://web-svc
</code></pre>

</div>






<h3>
  
  
  Key Concepts Learned
</h3>

<ul>
<li>Services select Pods using labels</li>
<li>Endpoints show real traffic targets</li>
<li>Service failure usually means selector mismatch</li>
</ul>




<h1>
  
  
  MODULE 2 — Ingress (Real Production Entry)
</h1>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fllpch47cmicx8mbphxgd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fllpch47cmicx8mbphxgd.png" alt="Image" width="800" height="544"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Ftetrate.io%2F.netlify%2Fimages%3Fh%3D549%26q%3D90%26url%3D_astro%252Fimage-1024x549.Dst0COpw.png%26w%3D1024" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Ftetrate.io%2F.netlify%2Fimages%3Fh%3D549%26q%3D90%26url%3D_astro%252Fimage-1024x549.Dst0COpw.png%26w%3D1024" alt="Image" width="1024" height="549"></a></p>

<p>Ingress provides:</p>

<ul>
<li>Single entry point</li>
<li>Path-based routing</li>
<li>Host-based routing</li>
<li>SSL termination</li>
</ul>




<h2>
  
  
  Project 2 — Ingress Routing
</h2>

<h3>
  
  
  Step 1 — Deploy Two Versions
</h3>

<p><strong>Stable Deployment</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apps/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Deployment</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">stable</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">replicas</span><span class="pi">:</span> <span class="m">2</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">matchLabels</span><span class="pi">:</span>
      <span class="na">app</span><span class="pi">:</span> <span class="s">echo</span>
      <span class="na">version</span><span class="pi">:</span> <span class="s">stable</span>
  <span class="na">template</span><span class="pi">:</span>
    <span class="na">metadata</span><span class="pi">:</span>
      <span class="na">labels</span><span class="pi">:</span>
        <span class="na">app</span><span class="pi">:</span> <span class="s">echo</span>
        <span class="na">version</span><span class="pi">:</span> <span class="s">stable</span>
    <span class="na">spec</span><span class="pi">:</span>
      <span class="na">containers</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">app</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">hashicorp/http-echo:0.2.3</span>
        <span class="na">args</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s2">"</span><span class="s">-listen=:8080"</span>
          <span class="pi">-</span> <span class="s2">"</span><span class="s">-text=STABLE</span><span class="nv"> </span><span class="s">VERSION"</span>
        <span class="na">ports</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">containerPort</span><span class="pi">:</span> <span class="m">8080</span>
</code></pre>

</div>



<p><strong>Canary Deployment</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apps/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Deployment</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">canary</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">replicas</span><span class="pi">:</span> <span class="m">1</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">matchLabels</span><span class="pi">:</span>
      <span class="na">app</span><span class="pi">:</span> <span class="s">echo</span>
      <span class="na">version</span><span class="pi">:</span> <span class="s">canary</span>
  <span class="na">template</span><span class="pi">:</span>
    <span class="na">metadata</span><span class="pi">:</span>
      <span class="na">labels</span><span class="pi">:</span>
        <span class="na">app</span><span class="pi">:</span> <span class="s">echo</span>
        <span class="na">version</span><span class="pi">:</span> <span class="s">canary</span>
    <span class="na">spec</span><span class="pi">:</span>
      <span class="na">containers</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">app</span>
        <span class="na">image</span><span class="pi">:</span> <span class="s">hashicorp/http-echo:0.2.3</span>
        <span class="na">args</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s2">"</span><span class="s">-listen=:8080"</span>
          <span class="pi">-</span> <span class="s2">"</span><span class="s">-text=CANARY</span><span class="nv"> </span><span class="s">VERSION"</span>
        <span class="na">ports</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">containerPort</span><span class="pi">:</span> <span class="m">8080</span>
</code></pre>

</div>






<h3>
  
  
  Step 2 — Services
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Service</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">stable-svc</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">app</span><span class="pi">:</span> <span class="s">echo</span>
    <span class="na">version</span><span class="pi">:</span> <span class="s">stable</span>
  <span class="na">ports</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">port</span><span class="pi">:</span> <span class="m">80</span>
    <span class="na">targetPort</span><span class="pi">:</span> <span class="m">8080</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Service</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">canary-svc</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">app</span><span class="pi">:</span> <span class="s">echo</span>
    <span class="na">version</span><span class="pi">:</span> <span class="s">canary</span>
  <span class="na">ports</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">port</span><span class="pi">:</span> <span class="m">80</span>
    <span class="na">targetPort</span><span class="pi">:</span> <span class="m">8080</span>
</code></pre>

</div>






<h3>
  
  
  Step 3 — Ingress
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">networking.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Ingress</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">app-ingress</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">rules</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">http</span><span class="pi">:</span>
      <span class="na">paths</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s">/</span>
        <span class="na">pathType</span><span class="pi">:</span> <span class="s">Prefix</span>
        <span class="na">backend</span><span class="pi">:</span>
          <span class="na">service</span><span class="pi">:</span>
            <span class="na">name</span><span class="pi">:</span> <span class="s">stable-svc</span>
            <span class="na">port</span><span class="pi">:</span>
              <span class="na">number</span><span class="pi">:</span> <span class="m">80</span>
      <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s">/canary</span>
        <span class="na">pathType</span><span class="pi">:</span> <span class="s">Prefix</span>
        <span class="na">backend</span><span class="pi">:</span>
          <span class="na">service</span><span class="pi">:</span>
            <span class="na">name</span><span class="pi">:</span> <span class="s">canary-svc</span>
            <span class="na">port</span><span class="pi">:</span>
              <span class="na">number</span><span class="pi">:</span> <span class="m">80</span>
</code></pre>

</div>






<h3>
  
  
  Test
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://&lt;INGRESS-IP&gt;/
curl http://&lt;INGRESS-IP&gt;/canary
</code></pre>

</div>






<h1>
  
  
  MODULE 3 — ConfigMaps &amp; Secrets
</h1>

<h2>
  
  
  Why Configuration Is External
</h2>

<p>Images must:</p>

<ul>
<li>Be immutable</li>
<li>Work in all environments</li>
</ul>

<p>Configuration must:</p>

<ul>
<li>Change without rebuilding images</li>
<li>Be environment-specific</li>
</ul>




<h2>
  
  
  Project 3 — ConfigMap Injection
</h2>

<h3>
  
  
  Step 1 — ConfigMap
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">ConfigMap</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">app-config</span>
<span class="na">data</span><span class="pi">:</span>
  <span class="na">MESSAGE</span><span class="pi">:</span> <span class="s2">"</span><span class="s">CONFIGMAP</span><span class="nv"> </span><span class="s">VALUE"</span>
</code></pre>

</div>






<h3>
  
  
  Step 2 — Deployment Using ConfigMap
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">containers</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">app</span>
  <span class="na">image</span><span class="pi">:</span> <span class="s">hashicorp/http-echo:0.2.3</span>
  <span class="na">args</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">-listen=:8080"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">-text=$(MESSAGE)"</span>
  <span class="na">env</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">MESSAGE</span>
    <span class="na">valueFrom</span><span class="pi">:</span>
      <span class="na">configMapKeyRef</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">app-config</span>
        <span class="na">key</span><span class="pi">:</span> <span class="s">MESSAGE</span>
</code></pre>

</div>






<h3>
  
  
  Update Config Live
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl edit configmap app-config
kubectl rollout restart deployment web
</code></pre>

</div>






<h1>
  
  
  MODULE 4 — Resource Management
</h1>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmedia.licdn.com%2Fdms%2Fimage%2FD5612AQHE2ObtjTAz7A%2Farticle-cover_image-shrink_720_1280%2F0%2F1719982339960%3Fe%3D2147483647%26t%3DGmjmEghOXWJQNG6PNOm-LmRX0-whZ3REcox5KmfU3QY%26v%3Dbeta" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmedia.licdn.com%2Fdms%2Fimage%2FD5612AQHE2ObtjTAz7A%2Farticle-cover_image-shrink_720_1280%2F0%2F1719982339960%3Fe%3D2147483647%26t%3DGmjmEghOXWJQNG6PNOm-LmRX0-whZ3REcox5KmfU3QY%26v%3Dbeta" alt="Image" width="1200" height="675"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj4vpsvt6pan58hwna6hc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj4vpsvt6pan58hwna6hc.png" alt="Image" width="800" height="600"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fcdn.prod.website-files.com%2F626a25d633b1b99aa0e1afa7%2F66c3581c6807ab2dd826addc_What%2520is%2520Kubernetes%2520CPU%2520throttling_%2520%25281%2529.svg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fcdn.prod.website-files.com%2F626a25d633b1b99aa0e1afa7%2F66c3581c6807ab2dd826addc_What%2520is%2520Kubernetes%2520CPU%2520throttling_%2520%25281%2529.svg" alt="Image" width="720" height="446"></a></p>




<h2>
  
  
  Requests vs Limits
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Setting</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td>requests</td>
<td>Guaranteed</td>
</tr>
<tr>
<td>limits</td>
<td>Maximum allowed</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Project 4 — OOM Kill Demo
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">resources</span><span class="pi">:</span>
  <span class="na">requests</span><span class="pi">:</span>
    <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">32Mi"</span>
    <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">50m"</span>
  <span class="na">limits</span><span class="pi">:</span>
    <span class="na">memory</span><span class="pi">:</span> <span class="s2">"</span><span class="s">64Mi"</span>
    <span class="na">cpu</span><span class="pi">:</span> <span class="s2">"</span><span class="s">100m"</span>
</code></pre>

</div>



<p>Observe:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl describe pod
</code></pre>

</div>






<h1>
  
  
  MODULE 5 — Autoscaling (HPA)
</h1>




<h2>
  
  
  Project 5 — CPU-Based Scaling
</h2>

<h3>
  
  
  Step 1 — Enable Metrics
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get apiservices | <span class="nb">grep </span>metrics
</code></pre>

</div>






<h3>
  
  
  Step 2 — HPA
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">autoscaling/v2</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">HorizontalPodAutoscaler</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">web-hpa</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">scaleTargetRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apps/v1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">Deployment</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">web</span>
  <span class="na">minReplicas</span><span class="pi">:</span> <span class="m">2</span>
  <span class="na">maxReplicas</span><span class="pi">:</span> <span class="m">5</span>
  <span class="na">metrics</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">Resource</span>
    <span class="na">resource</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">cpu</span>
      <span class="na">target</span><span class="pi">:</span>
        <span class="na">type</span><span class="pi">:</span> <span class="s">Utilization</span>
        <span class="na">averageUtilization</span><span class="pi">:</span> <span class="m">50</span>
</code></pre>

</div>






<h3>
  
  
  Generate Load
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">while </span><span class="nb">true</span><span class="p">;</span> <span class="k">do </span>wget <span class="nt">-qO-</span> http://web-svc<span class="p">;</span> <span class="k">done</span>
</code></pre>

</div>



<p>Watch:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get hpa
kubectl get pods
</code></pre>

</div>






<h1>
  
  
  MODULE 6 — Logs &amp; Troubleshooting
</h1>




<h2>
  
  
  Debug Order
</h2>

<ol>
<li>Pod status</li>
<li>Events</li>
<li>Logs</li>
<li>Resource usage</li>
<li>Service endpoints</li>
</ol>




<h2>
  
  
  Commands
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get pods
kubectl describe pod &lt;pod&gt;
kubectl logs &lt;pod&gt;
kubectl get events <span class="nt">--sort-by</span><span class="o">=</span>.metadata.creationTimestamp
</code></pre>

</div>






<h2>
  
  
  Incident Simulation
</h2>

<ul>
<li>Pod is Running</li>
<li>Browser shows nothing</li>
<li>Endpoint list is empty</li>
<li>Fix selector</li>
</ul>




<h1>
  
  
  MODULE 7 — Security Basics
</h1>




<h2>
  
  
  Minimal SecurityContext
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">securityContext</span><span class="pi">:</span>
  <span class="na">runAsNonRoot</span><span class="pi">:</span> <span class="kc">true</span>
  <span class="na">allowPrivilegeEscalation</span><span class="pi">:</span> <span class="kc">false</span>
</code></pre>

</div>






<h2>
  
  
  Image Best Practices
</h2>

<ul>
<li>Never use <code>latest</code>
</li>
<li>Use fixed versions</li>
<li>Use trusted registries</li>
</ul>




<h1>
  
  
  Final Integrated Project
</h1>

<p><strong>Production Application Includes:</strong></p>

<ul>
<li>Deployment with readiness probe</li>
<li>ClusterIP Service</li>
<li>Ingress routing</li>
<li>ConfigMap</li>
<li>Resource limits</li>
<li>HPA</li>
<li>Logs &amp; events</li>
<li>Secure container settings</li>
</ul>

<p>This mirrors how Kubernetes is used in real companies.</p>

