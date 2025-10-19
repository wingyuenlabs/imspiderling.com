---
Title: Tu primer sitio web estático con AWS CDK y Python
Description: 
Author: Afu Tse (Chainiz)
Date: 2025-10-19T21:58:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>¿Quieres aprender a desplegar un sitio web estático en AWS sin perderte entre servicios y pantallas?</p>

<p>En este tutorial paso a paso construirás tu primera infraestructura con AWS CDK (Cloud Development Kit) usando Python. Al final tendrás una página lista, distribuida con Amazon S3 y Amazon CloudFront.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frxhhcev3m1f06ts0o32r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frxhhcev3m1f06ts0o32r.png" alt="Diagrama" width="800" height="442"></a></p>




<h2>
  
  
  1. ¿Qué vas a construir?
</h2>

<ul>
<li>Un <em>bucket</em> de Amazon S3 para almacenar archivos estáticos (<code>HTML</code>, <code>CSS</code>, <code>JS</code>, imágenes).</li>
<li>Un <em>distribution</em> de Amazon CloudFront que entrega tu contenido rápidamente en todo el mundo.</li>
<li>Una pila (stack) de AWS CDK que puedes versionar y desplegar con un solo comando.</li>
</ul>

<blockquote>
<p>🧠 Idea clave: AWS CDK te permite describir infraestructura con código. Así obtienes la seguridad de un despliegue reproducible y fácil de mantener.</p>
</blockquote>




<h2>
  
  
  2. Requisitos previos
</h2>

<ol>
<li>
<strong>Cuenta de AWS</strong> activa.</li>
<li>
<strong>AWS CLI</strong> configurada con credenciales (<code>aws configure</code>).</li>
<li>
<strong>Node.js 16+</strong> para instalar CDK.</li>
<li>
<strong>Python 3.9+</strong> y <code>pip</code>.</li>
<li>
<strong>AWS CDK</strong> instalado globalmente:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   npm <span class="nb">install</span> <span class="nt">-g</span> aws-cdk
</code></pre>

</div>



<p>Comprueba que todo funciona:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws <span class="nt">--version</span>
python <span class="nt">--version</span>
cdk <span class="nt">--version</span>
</code></pre>

</div>






<h2>
  
  
  3. Crea el proyecto base
</h2>

<p>El repositorio que acompaña este tutorial ya tiene la estructura mínima. Si quieres comenzar desde cero, estos son los pasos:</p>

<ol>
<li>Inicializa un entorno de trabajo:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   <span class="nb">mkdir </span>cdk-static-site
   <span class="nb">cd </span>cdk-static-site
   cdk init app <span class="nt">--language</span> python
</code></pre>

</div>



<ol>
<li>Instala dependencias de CDK para S3 y CloudFront:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   python <span class="nt">-m</span> pip <span class="nb">install</span> <span class="s2">"aws-cdk-lib&gt;=2.0.0"</span> constructs
</code></pre>

</div>



<ol>
<li>Crea una carpeta <code>public/</code> con tu sitio estático (por ejemplo, un <code>index.html</code> simple). Puede descargar este <a href="https://gist.github.com/r3xakead0/2261d417dc51722fec79fd8d161b8119" rel="noopener noreferrer">ejemplo</a>
</li>
</ol>




<h2>
  
  
  4. Entendamos la pila <code>app.py</code>
</h2>

<p>Este archivo define los recursos de AWS. Un ejemplo básico luce así:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">aws_cdk</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">Stack</span><span class="p">,</span>
    <span class="n">aws_s3</span> <span class="k">as</span> <span class="n">s3</span><span class="p">,</span>
    <span class="n">aws_s3_deployment</span> <span class="k">as</span> <span class="n">s3deploy</span><span class="p">,</span>
    <span class="n">aws_cloudfront</span> <span class="k">as</span> <span class="n">cloudfront</span><span class="p">,</span>
    <span class="n">aws_cloudfront_origins</span> <span class="k">as</span> <span class="n">origins</span><span class="p">,</span>
<span class="p">)</span>
<span class="kn">from</span> <span class="n">constructs</span> <span class="kn">import</span> <span class="n">Construct</span>


<span class="k">class</span> <span class="nc">StaticSiteStack</span><span class="p">(</span><span class="n">Stack</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">scope</span><span class="p">:</span> <span class="n">Construct</span><span class="p">,</span> <span class="n">construct_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">scope</span><span class="p">,</span> <span class="n">construct_id</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>

        <span class="n">bucket</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="nc">Bucket</span><span class="p">(</span>
            <span class="n">self</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">SiteBucket</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">website_index_document</span><span class="o">=</span><span class="sh">"</span><span class="s">index.html</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">block_public_access</span><span class="o">=</span><span class="n">s3</span><span class="p">.</span><span class="n">BlockPublicAccess</span><span class="p">.</span><span class="n">BLOCK_ALL</span><span class="p">,</span>
            <span class="n">enforce_ssl</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
            <span class="n">auto_delete_objects</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
            <span class="n">removal_policy</span><span class="o">=</span><span class="n">s3</span><span class="p">.</span><span class="n">RemovalPolicy</span><span class="p">.</span><span class="n">DESTROY</span><span class="p">,</span>
        <span class="p">)</span>

        <span class="n">distribution</span> <span class="o">=</span> <span class="n">cloudfront</span><span class="p">.</span><span class="nc">Distribution</span><span class="p">(</span>
            <span class="n">self</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">SiteDistribution</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">default_behavior</span><span class="o">=</span><span class="n">cloudfront</span><span class="p">.</span><span class="nc">BehaviorOptions</span><span class="p">(</span>
                <span class="n">origin</span><span class="o">=</span><span class="n">origins</span><span class="p">.</span><span class="nc">S3Origin</span><span class="p">(</span><span class="n">bucket</span><span class="p">),</span>
                <span class="n">viewer_protocol_policy</span><span class="o">=</span><span class="n">cloudfront</span><span class="p">.</span><span class="n">ViewerProtocolPolicy</span><span class="p">.</span><span class="n">REDIRECT_TO_HTTPS</span><span class="p">,</span>
            <span class="p">),</span>
            <span class="n">default_root_object</span><span class="o">=</span><span class="sh">"</span><span class="s">index.html</span><span class="sh">"</span><span class="p">,</span>
        <span class="p">)</span>

        <span class="n">s3deploy</span><span class="p">.</span><span class="nc">BucketDeployment</span><span class="p">(</span>
            <span class="n">self</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">DeployWithInvalidation</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">sources</span><span class="o">=</span><span class="p">[</span><span class="n">s3deploy</span><span class="p">.</span><span class="n">Source</span><span class="p">.</span><span class="nf">asset</span><span class="p">(</span><span class="sh">"</span><span class="s">./public</span><span class="sh">"</span><span class="p">)],</span>
            <span class="n">destination_bucket</span><span class="o">=</span><span class="n">bucket</span><span class="p">,</span>
            <span class="n">distribution</span><span class="o">=</span><span class="n">distribution</span><span class="p">,</span>
            <span class="n">distribution_paths</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">/*</span><span class="sh">"</span><span class="p">],</span>
        <span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  ¿Qué hace cada bloque?
</h3>

<ul>
<li>
<code>Bucket</code>: almacena los archivos del sitio. Bloqueamos acceso directo público y dejamos que CloudFront sea la puerta de entrada.</li>
<li>
<code>Distribution</code>: CDN que entrega el contenido con HTTPS automático.</li>
<li>
<code>BucketDeployment</code>: sube los archivos de <code>./public</code> y limpia la caché de CloudFront tras cada despliegue.</li>
</ul>




<h2>
  
  
  5. Configura el entorno de CDK
</h2>

<p>Antes de desplegar, usa <code>cdk bootstrap</code> para preparar tu cuenta de AWS. Solo necesitas hacerlo una vez por región:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cdk bootstrap aws://&lt;tu-account-id&gt;/&lt;region&gt;
</code></pre>

</div>



<p>Sustituye <code>&lt;tu-account-id&gt;</code> y <code>&lt;region&gt;</code> (ejemplo: <code>aws://123456789012/us-east-1</code>).</p>




<h2>
  
  
  6. Despliega tu sitio
</h2>

<ol>
<li>
<strong>Sintetiza</strong> la plantilla de CloudFormation:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   cdk synth
</code></pre>

</div>



<p>Verás un archivo YAML grande: es la definición de infraestructura que CDK generó.</p>

<ol>
<li>
<strong>Despliega</strong>:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   cdk deploy
</code></pre>

</div>



<p>Confirma el despliegue cuando se solicite. CDK subirá tu sitio y creará los recursos.</p>

<ol>
<li>Al finalizar, copia la URL que aparece en la salida (el dominio de CloudFront). ¡Ese es tu sitio en producción!</li>
</ol>




<h2>
  
  
  7. Actualiza tu contenido
</h2>

<p>Cada vez que modifiques archivos dentro de <code>public/</code>, solo debes volver a ejecutar:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cdk deploy
</code></pre>

</div>



<p>CDK calculará qué cambió, subirá nuevos archivos y limpiará la caché de CloudFront.</p>




<h2>
  
  
  8. Limpia recursos (opcional)
</h2>

<p>Si ya no necesitas el sitio, destruye la pila para evitar costos:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cdk destroy
</code></pre>

</div>



<p>Esto eliminará el bucket, la distribución y cualquier recurso creado por la pila.</p>




<h2>
  
  
  9. Próximos pasos
</h2>

<ul>
<li>Personaliza un dominio propio con Amazon Route 53 y certificados SSL (AWS Certificate Manager).</li>
<li>Agrega automatización con GitHub Actions o AWS CodeBuild para desplegar en cada push.</li>
<li>Integra herramientas de observabilidad como CloudWatch Logs para registrar accesos.</li>
</ul>




<p>🎉 ¡Listo! Acabas de desplegar un sitio estático moderno usando infraestructura como código. Explora más constructos de CDK y adapta este esqueleto a tus próximos proyectos.</p>

