---
Title: Configurar un CDN gratuito con GitHub y jsDelivr
Description: 
Author: Ciro
Date: 2025-10-21T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>¿Necesitas servir archivos CSS o JavaScript desde un CDN pero no quieres pagar por servicios de terceros? Existe una solución simple y gratuita que aprovecha GitHub y jsDelivr.</p>

<h2>
  
  
  El problema
</h2>

<p>Cuando desarrollas aplicaciones web, es común necesitar:</p>

<ul>
<li>Compartir hojas de estilo CSS entre múltiples proyectos</li>
<li>Servir archivos estáticos con baja latencia global</li>
<li>Cachear recursos de forma eficiente</li>
<li>Versionar tus assets de manera confiable</li>
</ul>

<p>Los CDN tradicionales pueden ser costosos o requerir configuración compleja. Pero hay una alternativa mejor.</p>

<h2>
  
  
  La solución: GitHub + jsDelivr
</h2>

<p><a href="https://www.jsdelivr.com/" rel="noopener noreferrer">jsDelivr</a> es un CDN gratuito que puede servir archivos directamente desde repositorios de GitHub. No requiere registro, configuración ni costo alguno.</p>

<h3>
  
  
  Ventajas
</h3>

<ul>
<li>
<strong>Gratis y sin límites</strong> para proyectos open source</li>
<li>
<strong>Red global de servidores</strong> con baja latencia</li>
<li>
<strong>Caché automático</strong> para mejorar el rendimiento</li>
<li>
<strong>Control de versiones</strong> mediante tags de Git</li>
<li>
<strong>Minificación automática</strong> disponible</li>
</ul>

<h2>
  
  
  Implementación paso a paso
</h2>

<p>Vamos a crear un CDN para servir archivos CSS, similar a <a href="https://github.com/Gekyzo/stylesheet-cdn" rel="noopener noreferrer">este ejemplo real</a>.</p>

<h3>
  
  
  1. Crea un repositorio en GitHub
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>stylesheet-cdn
<span class="nb">cd </span>stylesheet-cdn
git init
</code></pre>

</div>



<h3>
  
  
  2. Agrega tus archivos CSS
</h3>

<p>Crea una carpeta <code>styles/</code> y añade tu archivo CSS:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>styles
</code></pre>

</div>



<p>Por ejemplo, <code>styles/mis-estilos.css</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="c">/* Tu CSS aquí */</span>
<span class="nt">body</span> <span class="p">{</span>
  <span class="nl">font-family</span><span class="p">:</span> <span class="n">-apple-system</span><span class="p">,</span> <span class="n">BlinkMacSystemFont</span><span class="p">,</span> <span class="s1">"Segoe UI"</span><span class="p">,</span> <span class="n">Roboto</span><span class="p">,</span> <span class="n">Oxygen</span><span class="p">,</span>
    <span class="n">Ubuntu</span><span class="p">,</span> <span class="n">Cantarell</span><span class="p">,</span> <span class="nb">sans-serif</span><span class="p">;</span>
  <span class="nl">line-height</span><span class="p">:</span> <span class="m">1.6</span><span class="p">;</span>
  <span class="nl">color</span><span class="p">:</span> <span class="m">#333</span><span class="p">;</span>
<span class="p">}</span>

<span class="nt">h1</span><span class="o">,</span>
<span class="nt">h2</span><span class="o">,</span>
<span class="nt">h3</span> <span class="p">{</span>
  <span class="nl">color</span><span class="p">:</span> <span class="m">#2c3e50</span><span class="p">;</span>
  <span class="nl">margin-top</span><span class="p">:</span> <span class="m">1.5em</span><span class="p">;</span>
<span class="p">}</span>

<span class="nt">code</span> <span class="p">{</span>
  <span class="nl">background</span><span class="p">:</span> <span class="m">#f4f4f4</span><span class="p">;</span>
  <span class="nl">padding</span><span class="p">:</span> <span class="m">2px</span> <span class="m">6px</span><span class="p">;</span>
  <span class="nl">border-radius</span><span class="p">:</span> <span class="m">3px</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  3. Sube el repositorio a GitHub
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git add <span class="nb">.</span>
git commit <span class="nt">-m</span> <span class="s2">"Add stylesheet"</span>
git push origin main
</code></pre>

</div>



<h3>
  
  
  4. Usa tu archivo desde el CDN
</h3>

<p>Ahora puedes incluir tu CSS en cualquier página web:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;link</span>
      <span class="na">rel=</span><span class="s">"stylesheet"</span>
      <span class="na">href=</span><span class="s">"https://cdn.jsdelivr.net/gh/TuUsuario/stylesheet-cdn@main/styles/mahonnaise.vs-code-markdown-theme.css"</span>
    <span class="nt">/&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;h1&gt;</span>¡Hola mundo!<span class="nt">&lt;/h1&gt;</span>
    <span class="nt">&lt;p&gt;</span>Este CSS se sirve desde un CDN global gratuito.<span class="nt">&lt;/p&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</div>



<h3>
  
  
  Patrón de URL
</h3>

<p>El formato de URL de jsDelivr es:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://cdn.jsdelivr.net/gh/{usuario}/{repositorio}@{branch}/ruta/al/archivo
</code></pre>

</div>



<h2>
  
  
  Mejores prácticas
</h2>

<h3>
  
  
  Usa versiones en producción
</h3>

<p>Para proyectos en producción, es recomendable usar tags de Git en lugar de <code>@main</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Crea un tag</span>
git tag v1.0.0
git push origin v1.0.0
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="c">&lt;!-- URL con versión específica --&gt;</span>
<span class="nt">&lt;link</span>
  <span class="na">rel=</span><span class="s">"stylesheet"</span>
  <span class="na">href=</span><span class="s">"https://cdn.jsdelivr.net/gh/TuUsuario/stylesheet-cdn@v1.0.0/styles/mahonnaise.vs-code-markdown-theme.css"</span>
<span class="nt">/&gt;</span>
</code></pre>

</div>



<p>Esto asegura que tu aplicación no se rompa si actualizas el archivo en el futuro.</p>

<h3>
  
  
  Minificación automática
</h3>

<p>jsDelivr puede minificar tus archivos automáticamente añadiendo <code>.min</code> antes de la extensión:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;link</span>
  <span class="na">rel=</span><span class="s">"stylesheet"</span>
  <span class="na">href=</span><span class="s">"https://cdn.jsdelivr.net/gh/TuUsuario/stylesheet-cdn@v1.0.0/styles/mahonnaise.vs-code-markdown-theme.min.css"</span>
<span class="nt">/&gt;</span>
</code></pre>

</div>



<h3>
  
  
  Considera el caché
</h3>

<p>Los archivos se cachean en el CDN, por lo que los cambios pueden tardar 5-10 minutos en propagarse. Para desarrollo, usa <code>@main</code>. Para producción, usa tags versionados.</p>

<h2>
  
  
  Casos de uso
</h2>

<p>Esta técnica es perfecta para:</p>

<ul>
<li>
<strong>Librerías CSS compartidas</strong> entre múltiples proyectos</li>
<li>
<strong>Themes y plantillas</strong> de código abierto</li>
<li>
<strong>Archivos JavaScript</strong> de utilidades comunes</li>
<li>
<strong>Recursos estáticos</strong> como fuentes o iconos</li>
<li>
<strong>Prototipos rápidos</strong> sin necesidad de configurar hosting</li>
</ul>

<h2>
  
  
  Limitaciones
</h2>

<ul>
<li>El repositorio debe ser <strong>público</strong>
</li>
<li>No es ideal para archivos muy grandes (&gt;50MB)</li>
<li>El caché puede dificultar el desarrollo activo (usa <code>@main</code> para desarrollo)</li>
</ul>

<h2>
  
  
  Conclusión
</h2>

<p>Con GitHub y jsDelivr puedes tener un CDN global de nivel empresarial completamente gratis. Esta solución es ideal para proyectos open source, bibliotecas compartidas, y cualquier recurso estático que quieras distribuir eficientemente.</p>

<p>El código completo de ejemplo está disponible en <a href="https://github.com/Gekyzo/stylesheet-cdn" rel="noopener noreferrer">github.com/Gekyzo/stylesheet-cdn</a>.</p>

