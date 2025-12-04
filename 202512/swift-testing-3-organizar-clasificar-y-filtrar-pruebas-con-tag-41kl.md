---
Title: Swift Testing #3: Organizar, Clasificar y Filtrar pruebas con @Tag
Description: 
Author: David Goyes
Date: 2025-12-04T22:03:35.000Z
Robots: noindex,nofollow
Template: index
---
<p>Un proyecto puede tener cientos o miles de pruebas y suites. Cierto subconjunto de estas pruebas puede compartir alguna característica común como ser "crítico" o "inestable". La biblioteca de pruebas de Swift incluye un trait llamado <code>tags</code> para agrugar y categorizar pruebas.</p>

<p>Las etiquetas (Tags) son distintas de las suites de pruebas: Mientras que las suites organizan las pruebas a nivel de código, los tags dan cierta información semántica a una prueba, que puede compartirse con otras entre las distintas suites, archivos o incluso targets de prueba.</p>

<p>Una etiqueta es una instancia de <a href="https://developer.apple.com/documentation/testing/tag" rel="noopener noreferrer"><code>Tag</code></a> que se declara como una variable estática dentro de <a href="https://developer.apple.com/documentation/testing/tag" rel="noopener noreferrer"><code>Tag</code></a> con el macro <a href="https://developer.apple.com/documentation/testing/tag()" rel="noopener noreferrer"><code>Tag()</code></a> que obligatoriamente debe existir dentro de una extensión de <code>Tag</code> para que pueda ser usada en las pruebas.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">extension</span> <span class="kt">Tag</span> <span class="p">{</span>
  <span class="kd">@Tag</span> <span class="kd">static</span> <span class="k">var</span> <span class="nv">legallyRequired</span><span class="p">:</span> <span class="k">Self</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Si se usa <code>let</code> en lugar de <code>var</code> en la declaración del <code>Tag</code> se tiene la siguiente advertencia, que falla al usar Swift 6:</p>

<blockquote>
<p>Cannot expand accessor macro on variable declared with 'let'; this is an error in the Swift 6 language mode</p>
</blockquote>

<p>Puedo usar una o varias etiquetas para marcar una prueba, pasándolas como argumento variádico al "trait" <a href="https://developer.apple.com/documentation/testing/trait/tags(_:)" rel="noopener noreferrer"><code>.tags()</code></a>, después de la descripción de la prueba.</p>

<p>En caso de que use varias etiquetas, esto significa que la prueba haría parte de todos los grupos etiquetados al mismo tiempo. Sin embargo, esto no significa que la prueba se ejecute varias veces, sino que solo ejecuta una vez, pero el resultado aparece en dos grupos.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">extension</span> <span class="kt">Tag</span> <span class="p">{</span>
  <span class="kd">@Tag</span> <span class="kd">static</span> <span class="k">var</span> <span class="nv">tag1</span><span class="p">:</span> <span class="k">Self</span>
  <span class="kd">@Tag</span> <span class="kd">static</span> <span class="k">var</span> <span class="nv">tag2</span><span class="p">:</span> <span class="k">Self</span>
<span class="p">}</span>
<span class="kd">@Test</span><span class="p">(</span><span class="s">"Descripción de la prueba"</span><span class="p">,</span> <span class="o">.</span><span class="nf">tags</span><span class="p">(</span><span class="o">.</span><span class="n">tag1</span><span class="p">,</span> <span class="o">.</span><span class="n">tag2</span><span class="p">))</span>
<span class="kd">func</span> <span class="nf">prueba</span><span class="p">()</span> <span class="p">{</span> <span class="o">...</span> <span class="p">}</span>
</code></pre>

</div>



<p>Un <code>Tag</code> también puede ser usado para etiquetar una <code>Suite</code> entera de pruebas, lo que implicaría que todas las pruebas de esa <code>Suite</code> tendrían la misma etiqueta.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">extension</span> <span class="kt">Tag</span> <span class="p">{</span>
  <span class="kd">@Tag</span> <span class="kd">static</span> <span class="k">var</span> <span class="nv">tag1</span><span class="p">:</span> <span class="k">Self</span>
  <span class="kd">@Tag</span> <span class="kd">static</span> <span class="k">var</span> <span class="nv">tag2</span><span class="p">:</span> <span class="k">Self</span>
<span class="p">}</span>
<span class="kd">@Suite</span><span class="p">(</span><span class="s">"FEATURE: Alguna funcionalidad"</span><span class="p">,</span> <span class="o">.</span><span class="nf">tags</span><span class="p">(</span><span class="o">.</span><span class="n">tag1</span><span class="p">,</span> <span class="o">.</span><span class="n">tag2</span><span class="p">))</span>
<span class="kd">struct</span> <span class="kt">FeatureX</span> <span class="p">{</span>
  <span class="kd">@Test</span><span class="p">(</span><span class="s">"Descripción de la prueba 1"</span><span class="p">)</span>
  <span class="kd">func</span> <span class="nf">prueba1</span><span class="p">()</span> <span class="p">{</span> <span class="o">...</span> <span class="p">}</span>
  <span class="kd">@Test</span><span class="p">(</span><span class="s">"Descripción de la prueba 2"</span><span class="p">)</span>
  <span class="kd">func</span> <span class="nf">prueba2</span><span class="p">()</span> <span class="p">{</span> <span class="o">...</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Si se declaran dos etiquetas con el mismo nombre en dos lugares, Swift las considera como equivalentes y no presenta problema.<br>
Si se requiere distinguir dos etiquetas con el mismo nombre, se puede declarar un namespace dentro de Tags para diferenciarlas.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">extension</span> <span class="kt">Tag</span> <span class="p">{</span>
  <span class="kd">enum</span> <span class="kt">SomeNamespace</span> <span class="p">{}</span>
<span class="p">}</span>

<span class="kd">extension</span> <span class="kt">Tag</span><span class="o">.</span><span class="kt">SomeNamespace</span> <span class="p">{</span>
  <span class="kd">@Tag</span> <span class="kd">static</span> <span class="k">var</span> <span class="nv">extraSpecial</span><span class="p">:</span> <span class="kt">Tag</span>
<span class="p">}</span>

<span class="kd">@Test</span><span class="p">(</span>
  <span class="s">"Extra Special Sauce recipe is secret"</span><span class="p">,</span>
  <span class="o">.</span><span class="nf">tags</span><span class="p">(</span><span class="o">.</span><span class="kt">SomeNamespace</span><span class="o">.</span><span class="n">extraSpecial</span><span class="p">)</span>
<span class="p">)</span>
<span class="kd">func</span> <span class="nf">secretSauce</span><span class="p">()</span> <span class="p">{</span> <span class="o">...</span> <span class="p">}</span>
</code></pre>

</div>



<p>En el navegador de pruebas, se pueden agrupar por jerarquía o por etiquetas.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb730g26ilywxb91wei63.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb730g26ilywxb91wei63.jpg" alt=" " width="800" height="490"></a></p>

<p>En el reporte de las pruebas aparece una columna adicional con las etiquetas de las pruebas listadas.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fskvge32qkf3x92theekk.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fskvge32qkf3x92theekk.jpg" alt=" " width="800" height="282"></a></p>

<p>Se puede usar las etiquetas para filtrar el resultado. Inicialmente se muestran las pruebas de todas las etiquetas.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fahl6f42poinc4fpbh3f2.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fahl6f42poinc4fpbh3f2.jpg" alt=" " width="800" height="263"></a></p>

<h2>
  
  
  Bibliografía
</h2>

<ul>
<li>Video "Mastering Swift Testing: Organize and Filter Tests with @Tag" (Swift and Tips), <a href="https://www.youtube.com/watch?v=7LOBNeAmm44" rel="noopener noreferrer">aquí</a>.</li>
<li>Lista de reproducción "Swift Testing" (Swift and Tips), <a href="https://www.youtube.com/playlist?list=PLHWvYoDHvsOV67md_mU5nMN_HDZK7rEKn" rel="noopener noreferrer">aquí</a>.</li>
<li>Documentación sobre Swift Testing, <a href="https://developer.apple.com/documentation/testing" rel="noopener noreferrer">aquí</a>.</li>
<li>Documentación: Adding tags to tests, <a href="https://developer.apple.com/documentation/testing/addingtags" rel="noopener noreferrer">aquí</a>.</li>
</ul>

