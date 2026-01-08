---
Title: Valkey vs Redis – Explicación simple y practica
Description: 
Author: Afu Tse (Chainiz)
Date: 2026-01-08T21:59:36.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🔑 ¿Qué es Valkey?
</h2>

<p><strong>Valkey</strong> es un <strong>fork open-source de Redis</strong>, creado en <strong>2024</strong>, totalmente compatible, creado tras el cambio de licencia de Redis para mantener una alternativa 100% open source y <strong>respaldado por la Linux Foundation</strong>.</p>

<p>Surge como respuesta al <strong>cambio de licencia de Redis</strong> (Redis ≥ 7.4), que dejó de ser open source bajo los estándares de la OSI.</p>

<p>👉 <strong>Objetivo de Valkey</strong></p>

<ul>
<li>Mantener un Redis <strong>100% open source</strong>
</li>
<li>Compatibilidad total con Redis (comandos, protocolo, clientes)</li>
<li>Gobernanza comunitaria y neutral</li>
<li>Sin vendor lock-in</li>
</ul>

<p>En la práctica:</p>

<blockquote>
<p><strong>Valkey es Redis, pero con licencia abierta y desarrollo comunitario.</strong></p>
</blockquote>




<h2>
  
  
  🧠 ¿Para qué sirve Valkey?
</h2>

<p>Exactamente los mismos casos de uso que Redis:</p>

<ul>
<li>Cache in-memory</li>
<li>Session store</li>
<li>Pub/Sub</li>
<li>Rate limiting</li>
<li>Distributed locks</li>
<li>Streams</li>
<li>Leaderboards</li>
<li>Replication</li>
<li>Persistence (RDB / AOF)</li>
<li>Redis Cluster</li>
</ul>




<h2>
  
  
  ⚖️ Valkey vs Redis (comparación rápida)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Característica</th>
<th>Valkey</th>
<th>Redis (≥ 7.4)</th>
</tr>
</thead>
<tbody>
<tr>
<td>Licencia</td>
<td>✅ Open Source (BSD/Apache-style)</td>
<td>❌ RSAL / SSPL</td>
</tr>
<tr>
<td>Gobernanza</td>
<td>Linux Foundation</td>
<td>Redis Inc</td>
</tr>
<tr>
<td>Compatibilidad</td>
<td>100% Redis</td>
<td>Nativa</td>
</tr>
<tr>
<td>Uso comercial</td>
<td>✅ Libre</td>
<td>⚠️ Restricciones</td>
</tr>
<tr>
<td>Vendor lock-in</td>
<td>❌ Bajo</td>
<td>⚠️ Mayor</td>
</tr>
<tr>
<td>Cloud friendly</td>
<td>✅ Sí</td>
<td>⚠️ Limitado</td>
</tr>
<tr>
<td>Soporte K8s</td>
<td>✅ Excelente</td>
<td>✅</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  🧾 La licencia: el punto clave
</h2>

<h3>
  
  
  Redis hoy
</h3>

<ul>
<li>Licencia RSAL / SSPL</li>
<li>No considerada open source por la OSI</li>
<li>Restricciones para SaaS y cloud providers</li>
</ul>

<h3>
  
  
  Valkey
</h3>

<ul>
<li>Open source real</li>
<li>Uso comercial sin restricciones</li>
<li>Ideal para empresas, educación y gobiernos</li>
</ul>

<p>Por eso <strong>AWS, Google, Oracle y otros proveedores apoyan Valkey</strong>.</p>




<h2>
  
  
  🧪 Demo 1 – Valkey local con Docker
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">-d</span> <span class="nt">--name</span> valkey <span class="nt">-p</span> 6379:6379 valkey/valkey:latest
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsv4ubt3ek2u9tlgrmu4m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsv4ubt3ek2u9tlgrmu4m.png" alt="docker run" width="800" height="316"></a></p>

<p>Prueba rápida:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>redis-cli ping
<span class="c"># PONG</span>

redis-cli <span class="nb">set </span>demo <span class="s2">"hola valkey"</span>
redis-cli get demo
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foi7szg2kyzmph654qev2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foi7szg2kyzmph654qev2.png" alt="redis-cli" width="696" height="214"></a></p>




<h2>
  
  
  🧪 Demo 2 – Pub/Sub
</h2>

<p>Terminal A:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>redis-cli subscribe events
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekf2dsemgylimldveovh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekf2dsemgylimldveovh.png" alt="redis-cli subscribe" width="626" height="184"></a></p>

<p>Terminal B:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>redis-cli publish events <span class="s2">"mensaje desde valkey"</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd0mxmc7mia608oqrvk9g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd0mxmc7mia608oqrvk9g.png" alt="redis-cli publish" width="800" height="69"></a></p>

<p>Terminal A:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fze1a4mrplzsnypa3bjwn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fze1a4mrplzsnypa3bjwn.png" alt="redis-cli subscribe result" width="630" height="288"></a></p>


<h2>
  
  
  ☸️ Demo 3 – Valkey en Kubernetes
</h2>

<p>Crear el archivo "valkey.yaml"<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Namespace</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">demo</span>
<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Service</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">valkey</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">demo</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">app</span><span class="pi">:</span> <span class="s">valkey</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="c1"># Headless Service required by StatefulSet</span>
  <span class="na">clusterIP</span><span class="pi">:</span> <span class="s">None</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">app</span><span class="pi">:</span> <span class="s">valkey</span>
  <span class="na">ports</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">redis</span>
      <span class="na">port</span><span class="pi">:</span> <span class="m">6379</span>
      <span class="na">targetPort</span><span class="pi">:</span> <span class="m">6379</span>
<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apps/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">StatefulSet</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">valkey</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">demo</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">serviceName</span><span class="pi">:</span> <span class="s">valkey</span>
  <span class="na">replicas</span><span class="pi">:</span> <span class="m">1</span>
  <span class="na">selector</span><span class="pi">:</span>
    <span class="na">matchLabels</span><span class="pi">:</span>
      <span class="na">app</span><span class="pi">:</span> <span class="s">valkey</span>
  <span class="na">template</span><span class="pi">:</span>
    <span class="na">metadata</span><span class="pi">:</span>
      <span class="na">labels</span><span class="pi">:</span>
        <span class="na">app</span><span class="pi">:</span> <span class="s">valkey</span>
    <span class="na">spec</span><span class="pi">:</span>
      <span class="na">containers</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">valkey</span>
          <span class="na">image</span><span class="pi">:</span> <span class="s">valkey/valkey:latest</span>
          <span class="na">ports</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">redis</span>
              <span class="na">containerPort</span><span class="pi">:</span> <span class="m">6379</span>
          <span class="na">volumeMounts</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">data</span>
              <span class="na">mountPath</span><span class="pi">:</span> <span class="s">/data</span>
          <span class="na">readinessProbe</span><span class="pi">:</span>
            <span class="na">tcpSocket</span><span class="pi">:</span>
              <span class="na">port</span><span class="pi">:</span> <span class="m">6379</span>
            <span class="na">initialDelaySeconds</span><span class="pi">:</span> <span class="m">5</span>
            <span class="na">periodSeconds</span><span class="pi">:</span> <span class="m">5</span>
          <span class="na">livenessProbe</span><span class="pi">:</span>
            <span class="na">tcpSocket</span><span class="pi">:</span>
              <span class="na">port</span><span class="pi">:</span> <span class="m">6379</span>
            <span class="na">initialDelaySeconds</span><span class="pi">:</span> <span class="m">15</span>
            <span class="na">periodSeconds</span><span class="pi">:</span> <span class="m">10</span>
  <span class="na">volumeClaimTemplates</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">metadata</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">data</span>
      <span class="na">spec</span><span class="pi">:</span>
        <span class="na">accessModes</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">ReadWriteOnce</span>
        <span class="na">resources</span><span class="pi">:</span>
          <span class="na">requests</span><span class="pi">:</span>
            <span class="na">storage</span><span class="pi">:</span> <span class="s">1Gi</span>
</code></pre>

</div>



<p>Desplegar:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> valkey.yaml
</code></pre>

</div>



<p>Acceso desde un pod:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl run <span class="nt">-it</span> <span class="nt">--rm</span> vcli <span class="nt">-n</span> demo <span class="nt">--image</span><span class="o">=</span>redis:7 <span class="nt">--</span> bash
</code></pre>

</div>



<p>Ejecuta el ping para validar conexion<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>redis-cli <span class="nt">-h</span> valkey.demo.svc.cluster.local ping
PONG
</code></pre>

</div>



<blockquote>
<p>Código fuente : <a href="https://github.com/r3xakead0/k8s-valkey-demo" rel="noopener noreferrer">k8s-valkey-demo</a></p>
</blockquote>




<h2>
  
  
  🔄 ¿Tengo que cambiar código si vengo de Redis?
</h2>

<p>👉 <strong>No.</strong></p>

<ul>
<li>Mismos comandos</li>
<li>Mismo protocolo RESP</li>
<li>Mismos clientes (<code>redis-cli</code>, <code>node-redis</code>, <code>ioredis</code>, etc.)</li>
</ul>

<p>Solo cambia la imagen:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">image</span><span class="pi">:</span> <span class="s">valkey/valkey</span>
</code></pre>

</div>






<h2>
  
  
  🚀 Conclusión
</h2>

<p>Si estás construyendo:</p>

<ul>
<li>Kubernetes</li>
<li>SaaS</li>
<li>Plataformas cloud</li>
<li>Proyectos open source</li>
</ul>

<p>👉 <strong>Valkey es la opción recomendada hoy</strong>.</p>

