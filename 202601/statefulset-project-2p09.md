---
Title: StatefulSet project
Description: 
Author: Aisalkyn Aidarova
Date: 2026-01-18T21:05:41.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5j8w2l8hwbnr345awvj4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5j8w2l8hwbnr345awvj4.png" alt="Image" width="800" height="511"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyyy13fx10nytnt4o8gta.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyyy13fx10nytnt4o8gta.png" alt="Image" width="800" height="453"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqfvrea0dkqbwtx9vjax9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqfvrea0dkqbwtx9vjax9.png" alt="Image" width="800" height="533"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feieo5gy1opgyix40ot1o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feieo5gy1opgyix40ot1o.png" alt="Image" width="800" height="533"></a></p>

<p>Below is a <strong>clear, real DevOps way</strong> to create a <strong>StatefulSet</strong> step by step.</p>




<h1>
  
  
  How to create a StatefulSet (step-by-step)
</h1>

<h2>
  
  
  Step 0: Understand the required components
</h2>

<p>A StatefulSet <strong>always needs</strong>:</p>

<ol>
<li>
<strong>Headless Service</strong> (for stable DNS)</li>
<li><strong>StatefulSet manifest</strong></li>
<li>
<strong>PersistentVolume / StorageClass</strong> (usually already exists)</li>
</ol>




<h2>
  
  
  Step 1: Create a Headless Service (MANDATORY)
</h2>

<p>StatefulSet <strong>will not work correctly without this</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Service</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">mysql-headless</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">clusterIP</span><span class="pi">:</span> <span class="s">None</span>        <span class="c1"># &lt;- this makes it headless</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">app</span><span class="pi">:</span> <span class="s">mysql</span>
  <span class="na">ports</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">port</span><span class="pi">:</span> <span class="m">3306</span>
</code></pre>

</div>



<p>Apply:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> service.yaml
</code></pre>

</div>



<p>Why this matters:</p>

<ul>
<li>No load balancing</li>
<li>Each pod gets <strong>its own DNS</strong>
</li>
<li>Required for databases and clustering</li>
</ul>




<h2>
  
  
  Step 2: Create the StatefulSet
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apps/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">StatefulSet</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">mysql</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">serviceName</span><span class="pi">:</span> <span class="s">mysql-headless</span>   <span class="c1"># MUST match service name</span>
  <span class="na">replicas</span><span class="pi">:</span> <span class="m">3</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">matchLabels</span><span class="pi">:</span>
      <span class="na">app</span><span class="pi">:</span> <span class="s">mysql</span>

  <span class="na">template</span><span class="pi">:</span>
    <span class="na">metadata</span><span class="pi">:</span>
      <span class="na">labels</span><span class="pi">:</span>
        <span class="na">app</span><span class="pi">:</span> <span class="s">mysql</span>
    <span class="na">spec</span><span class="pi">:</span>
      <span class="na">containers</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">mysql</span>
          <span class="na">image</span><span class="pi">:</span> <span class="s">mysql:8.0</span>
          <span class="na">ports</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">containerPort</span><span class="pi">:</span> <span class="m">3306</span>
          <span class="na">env</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">MYSQL_ROOT_PASSWORD</span>
              <span class="na">value</span><span class="pi">:</span> <span class="s">root</span>
          <span class="na">volumeMounts</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">data</span>
              <span class="na">mountPath</span><span class="pi">:</span> <span class="s">/var/lib/mysql</span>

  <span class="na">volumeClaimTemplates</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">metadata</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">data</span>
      <span class="na">spec</span><span class="pi">:</span>
        <span class="na">accessModes</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">ReadWriteOnce"</span><span class="pi">]</span>
        <span class="na">resources</span><span class="pi">:</span>
          <span class="na">requests</span><span class="pi">:</span>
            <span class="na">storage</span><span class="pi">:</span> <span class="s">5Gi</span>
</code></pre>

</div>



<p>Apply:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> statefulset.yaml
</code></pre>

</div>






<h2>
  
  
  Step 3: Verify everything
</h2>

<h3>
  
  
  Pods (note the order and names)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get pods
</code></pre>

</div>



<p>You will see:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mysql-0
mysql-1
mysql-2
</code></pre>

</div>



<h3>
  
  
  PVCs (one per pod)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get pvc
</code></pre>

</div>



<p>You will see:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>data-mysql-0
data-mysql-1
data-mysql-2
</code></pre>

</div>






<h2>
  
  
  Step 4: Verify DNS (this is the magic)
</h2>

<p>Exec into any pod:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl <span class="nb">exec</span> <span class="nt">-it</span> mysql-0 <span class="nt">--</span> bash
</code></pre>

</div>



<p>Inside:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ping mysql-1.mysql-headless
ping mysql-2.mysql-headless
</code></pre>

</div>



<p>Each pod has <strong>its own stable hostname</strong>.</p>




<h2>
  
  
  Step 5: Scale safely (ordered)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl scale statefulset mysql <span class="nt">--replicas</span><span class="o">=</span>4
</code></pre>

</div>



<p>What happens:</p>

<ul>
<li>
<code>mysql-3</code> created <strong>after</strong> <code>mysql-2</code>
</li>
<li>New PVC <code>data-mysql-3</code> created</li>
<li>No data loss</li>
</ul>




<h2>
  
  
  VERY IMPORTANT production rules (interview-level)
</h2>

<h3>
  
  
  ✔ StatefulSet guarantees
</h3>

<ul>
<li>Stable pod name</li>
<li>Stable DNS</li>
<li>Stable storage</li>
<li>Ordered start/stop</li>
</ul>

<h3>
  
  
  ❌ StatefulSet does NOT do
</h3>

<ul>
<li>Replication logic (MySQL replication is <strong>your job</strong>)</li>
<li>Backups</li>
<li>Failover automation (unless configured)</li>
</ul>




<h2>
  
  
  When examiners / interviewers expect StatefulSet
</h2>

<p>Use it for:</p>

<ul>
<li>MySQL / PostgreSQL</li>
<li>MongoDB</li>
<li>Kafka</li>
<li>ZooKeeper</li>
<li>Elasticsearch</li>
</ul>




<h2>
  
  
  One-line DevOps explanation
</h2>

<blockquote>
<p><strong>A StatefulSet creates pods with stable identity, DNS, and persistent storage, required for stateful workloads like databases.</strong></p>
</blockquote>

