---
Title: El error de seguridad más común es “Dale Admin y Ya”
Description: 
Author: Augusto Valdivia
Date: 2025-12-22T21:28:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>Cuando estamos bajo presión, casi siempre gana la solución más rápida. Algo falla, alguien necesita acceso, hay una entrega cerca. Entonces hacemos lo típico: damos permisos amplios “por ahora”.</p>

<p>El problema es que lo temporal suele quedarse para siempre.</p>

<p>Menor privilegio no es paranoia. Es intención. Damos solo lo necesario para que los errores tengan un impacto pequeño y la seguridad sea más predecible.</p>

<h2>
  
  
  Qué significa menor privilegio de verdad
</h2>

<p>Menor privilegio significa:</p>

<ul>
<li>Solo las acciones necesarias</li>
<li>Solo los recursos necesarios</li>
<li>Solo cuando se necesita</li>
<li>Solo para la identidad correcta</li>
</ul>

<p>Una buena política responde:</p>

<ul>
<li>Qué necesita hacer este sistema</li>
<li>En qué recursos lo hará</li>
<li>Qué cosas nunca debería poder hacer</li>
</ul>

<p>IAM no es solo seguridad. IAM también es estabilidad. Un rol con demasiado poder puede romper más cosas más rápido.</p>

<h2>
  
  
  Por Qué Importa a Gran Escala
</h2>

<p>En entornos pequeños, los permisos amplios tal vez no exploten de inmediato. En entornos grandes, tarde o temprano sí.</p>

<p>Menor privilegio te protege de:</p>

<ul>
<li>Impacto masivo si una credencial se compromete</li>
<li>Borrados accidentales en producción</li>
<li>Roles antiguos que nadie recuerda</li>
<li>Auditorías difíciles de explicar</li>
</ul>

<p>Además, ayuda a depurar. Si algo falla, sabemos que los límites de acceso son reales.</p>

<h2>
  
  
  Dónde Fallamos Normalmente
</h2>

<p>Los patrones más comunes son:</p>

<ol>
<li>Wildcards como <code>*:*</code>
</li>
<li>Políticas copiadas sin limpieza</li>
<li>Un rol para todo</li>
<li>Permisos temporales que nunca se quitan</li>
<li>No separar permisos de despliegue y ejecución</li>
</ol>

<p>Esto les pasa a equipos buenos también. La solución es un patrón claro.</p>

<h2>
  
  
  Ejemplos: Mala Política vs Buena Política
</h2>

<h3>
  
  
  Ejemplo 1: Acceso a S3
</h3>

<h4>
  
  
  ❌ Mala política (demasiado amplia)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2012-10-17"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Statement"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Allow"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"s3:*"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h4>
  
  
  ✅ Buena política (limitada y práctica)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2012-10-17"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Statement"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ListBucketInPrefix"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Allow"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"s3:ListBucket"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:s3:::my-app-data"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Condition"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"StringLike"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"s3:prefix"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"public/*"</span><span class="p">]</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ReadObjectsInPrefix"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Allow"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"s3:GetObject"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:s3:::my-app-data/public/*"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Un Sistema Simple para Diseñar IAM Bien
</h2>

<ol>
<li>Separar roles</li>
<li>Empezar mínimo y crecer cuando sea necesario</li>
<li>Usar guardrails como SCPs y boundaries</li>
<li>Revisar y limpiar permisos regularmente</li>
</ol>

<h2>
  
  
  🧪 Mini Proyecto: Rol de Menor Privilegio para Lambda + S3 usando Terraform.
</h2>

<h3>
  
  
  Objetivo
</h3>

<p>Crear:</p>

<ul>
<li>Un bucket S3</li>
<li>Un rol de ejecución de Lambda</li>
<li>Una política de menor privilegio</li>
<li>Adjuntar la política al rol</li>
</ul>

<p>La Lambda podrá:</p>

<ul>
<li>Leer solo de <code>public/</code>
</li>
<li>Escribir solo en <code>results/</code>
</li>
<li>Escribir logs en CloudWatch</li>
</ul>

<p>Si este artículo te ayudó, aquí está lo que puedes hacer después:</p>

<p><strong>Sígueme</strong> en <a href="https://x.com/TerraSpacio" rel="noopener noreferrer">X</a> y <a href="https://www.youtube.com/@TerraSpacio" rel="noopener noreferrer">YouTube</a> para más contenido de AWS, DevOps y Terraform,para principiantes o expertos. También comparto miniproyectos en nuestro newsletter ☕ Cloud Café <a href="https://www.linkedin.com/build-relation/newsletter-follow?entityUrn=7373717160319299584" rel="noopener noreferrer">Subscríbete en LinkedIn</a>.</p>

<p><strong>Pronto crearé un repositorio en GitHub.</strong> Donde compartiré recursos y ejemplos de práctica.</p>

