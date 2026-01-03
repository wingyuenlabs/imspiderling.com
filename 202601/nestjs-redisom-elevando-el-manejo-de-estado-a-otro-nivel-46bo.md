---
Title: 🚀 NestJS + RedisOM: Elevando el manejo de Estado a otro nivel
Description: 
Author: Tomás Alegre Sepúlveda
Date: 2026-01-03T21:19:40.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>🇺🇸 <a href="https://dev.to/alpha018/nestjs-redisom-taking-state-management-to-the-next-level-5gli">Read this post in English</a></p>
</blockquote>

<p>Si vienes del mundo de <strong>NestJS</strong>, amas los decoradores, la inyección de dependencias y el tipado fuerte. Pero cuando toca integrar <strong>Redis</strong>, a menudo sentimos que retrocedemos 10 años: <em>magic strings</em>, parsers manuales de JSON y comandos <em>low-level</em> que ensucian nuestra lógica de negocio.</p>

<p>Hoy quiero presentarles <strong><code>nestjs-redisom</code></strong>, una librería open-source diseñada para cerrar esa brecha y tratar a Redis con el respeto que se merece en una arquitectura empresarial moderna.</p>




<h2>
  
  
  🤔 ¿Por qué creé esto?
</h2>

<p>Redis ha evolucionado. Ya no es solo un caché key-value. Con <strong>Redis Stack</strong> (RedisJSON + RediSearch), se ha convertido en una base de datos de documentos en tiempo real increíblemente potente.</p>

<p>Sin embargo, usar estas features desde Node.js suele ser verboso. <strong>Redis OM</strong> (la librería oficial) ayuda, pero integrarla en NestJS requería mucho <em>boilerplate</em>. <code>nestjs-redisom</code> elimina esa fricción, dándote una experiencia nativa.</p>

<h3>
  
  
  Instalación
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>nestjs-redisom redis-om redis
</code></pre>

</div>






<h2>
  
  
  🛠️ Profundizando en el Código
</h2>

<p>Veamos ejemplos más realistas de lo que puedes lograr.</p>

<h3>
  
  
  1. Modelado de Datos Complejos (Anidados)
</h3>

<p>No estamos limitados a strings planos. Podemos modelar objetos complejos con validación implícita.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// 1. Definimos una clase anidada</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">Address</span> <span class="p">{</span>
  <span class="p">@</span><span class="nd">Prop</span><span class="p">({</span> <span class="na">indexed</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
  <span class="nx">city</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="c1">// Indexado para buscar "Usuarios en Madrid"</span>

  <span class="p">@</span><span class="nd">Prop</span><span class="p">()</span>
  <span class="nx">zipCode</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// 2. Definimos la Entidad Principal</span>
<span class="p">@</span><span class="nd">Schema</span><span class="p">()</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">UserProfile</span> <span class="kd">extends</span> <span class="nc">BaseEntity</span> <span class="p">{</span>
  <span class="p">@</span><span class="nd">Prop</span><span class="p">({</span> <span class="na">textSearch</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
  <span class="nx">bio</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>

  <span class="p">@</span><span class="nd">Prop</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">Address</span> <span class="p">})</span> <span class="c1">// Mapeo anidado</span>
  <span class="nx">address</span><span class="p">:</span> <span class="nx">Address</span><span class="p">;</span>

  <span class="p">@</span><span class="nd">Prop</span><span class="p">()</span>
  <span class="nx">roles</span><span class="p">:</span> <span class="kr">string</span><span class="p">[];</span> <span class="c1">// Arrays de strings</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Búsquedas Avanzadas (Fluent API)
</h3>

<p>Aquí es donde <code>nestjs-redisom</code> brilla. Olvídate de aprender la sintaxis de comandos de RediSearch (<code>FT.SEARCH index "@field:value ..."</code>). Usa esta API fluida y tipada.</p>

<blockquote>
<p><em>"Encuentra usuarios que vivan en 'New York', tengan el rol 'admin' y cuya bio mencione 'typescript'"</em><br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">users</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">userRepo</span><span class="p">.</span><span class="nf">search</span><span class="p">()</span>
  <span class="p">.</span><span class="nf">where</span><span class="p">(</span><span class="dl">'</span><span class="s1">address_city</span><span class="dl">'</span><span class="p">).</span><span class="nf">equals</span><span class="p">(</span><span class="dl">'</span><span class="s1">New York</span><span class="dl">'</span><span class="p">)</span> <span class="c1">// Busca en objeto anidado</span>
  <span class="p">.</span><span class="nf">and</span><span class="p">(</span><span class="dl">'</span><span class="s1">roles</span><span class="dl">'</span><span class="p">).</span><span class="nf">contain</span><span class="p">(</span><span class="dl">'</span><span class="s1">admin</span><span class="dl">'</span><span class="p">)</span>            <span class="c1">// Busca en Array</span>
  <span class="p">.</span><span class="nf">and</span><span class="p">(</span><span class="dl">'</span><span class="s1">bio</span><span class="dl">'</span><span class="p">).</span><span class="nf">matches</span><span class="p">(</span><span class="dl">'</span><span class="s1">typescript</span><span class="dl">'</span><span class="p">)</span>         <span class="c1">// Full-Text Search</span>
  <span class="p">.</span><span class="k">return</span><span class="p">.</span><span class="nf">page</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span>                      <span class="c1">// Paginación nativa</span>
</code></pre>

</div>



<h3>
  
  
  3. Expiración de Documentos (TTL)
</h3>

<p>Manejar sesiones o caché temporal es trivial. No necesitas scripts de Lua ni nada por el estilo.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Crea una sesión y haz que expire en 1 hora</span>
<span class="k">async</span> <span class="nf">createSession</span><span class="p">(</span><span class="nx">user</span><span class="p">:</span> <span class="nx">User</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">sessionRepo</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="nx">user</span><span class="p">);</span>
  <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">sessionRepo</span><span class="p">.</span><span class="nf">expire</span><span class="p">(</span><span class="nx">session</span><span class="p">[</span><span class="k">this</span><span class="p">.</span><span class="nx">sessionRepo</span><span class="p">.</span><span class="nx">schema</span><span class="p">.</span><span class="nx">idField</span><span class="p">],</span> <span class="mi">3600</span><span class="p">);</span>
  <span class="k">return</span> <span class="nx">session</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  🎯 Mejores Casos de Uso
</h2>

<ol>
<li>
<strong>High-Speed Objects</strong>: Carritos de compra, Perfiles de Jugador, Sesiones. Datos que lees/escribes 1000 veces por segundo.</li>
<li>
<strong>Caché con "Poderes"</strong>: A veces necesitas invalidar caché por criterio (<em>"borrar caché de todos los productos de la categoría X"</em>). Con Redis normal es imposible (o lento con <code>KEYS *</code>). Con RedisOM es una <em>query</em> de O(1) o O(N log N).</li>
<li>
<strong>Geo-Spatial Features</strong>: "Encuentra conductores cerca de mi", usando la potencia nativa de Redis para geo-hashing.</li>
</ol>




<h2>
  
  
  ⚠️ Trade-offs (Honestidad ante todo)
</h2>

<ul>
<li>
<strong>RAM es Dinero</strong>: Redis guarda todo en memoria. No uses esto para guardar logs de auditoría de 5 años.</li>
<li>
<strong>Eventual Consistency</strong>: Los índices de búsqueda se actualizan asíncronamente (milisegundos). No es ACID estricto para búsquedas inmediatas post-escritura.</li>
</ul>




<h2>
  
  
  💌 Agradecimientos Especiales
</h2>

<p>Este proyecto ha sido construido con muchas horas de dedicación y amor por el código.</p>

<blockquote>
<p><em>Para una chica que siempre me inspira en silencio, y su sola existencia me incentiva a seguir programando en mis tiempos libres, gracias "YLP".</em></p>
</blockquote>




<h2>
  
  
  ☕ Apoya el Proyecto
</h2>

<p>Si este proyecto te ha sido útil o te parece interesante, considera apoyarlo:</p>

<ul>
<li>⭐️ <strong>Star en GitHub</strong>: Es gratis y ayuda a que más gente lo encuentre.</li>
<li>☕ <strong>Invítame un Café</strong>: Si quieres ir un paso más allá, puedes hacerlo en <a href="https://buymeacoffee.com/alpha018" rel="noopener noreferrer">buymeacoffee.com/alpha018</a>.</li>
</ul>




<h2>
  
  
  🔗 Pruébalo y Contribuye
</h2>

<ul>
<li>
<strong>Repo</strong>: <a href="https://github.com/Alpha018/nestjs-redisom" rel="noopener noreferrer">github.com/Alpha018/nestjs-redisom</a>
</li>
<li>
<strong>Autor</strong>: <a href="https://dev.to/alpha018">dev.to/alpha018</a>
</li>
</ul>

<p>¿Qué opinan? ¿Le darán una oportunidad a Redis como base de datos de documentos? 👇</p>

