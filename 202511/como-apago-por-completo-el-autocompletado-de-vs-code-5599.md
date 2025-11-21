---
Title: Como apago por completo el autocompletado de VS Code
Description: 
Author: Oscar Pincho
Date: 2025-11-21T21:30:00.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Sí, ese soy yo y se preguntarán...</em></p>

<p>Sé que luego de leer el título lo que en verdad se preguntan es</p>

<p><em><strong>"Y quién quiere hacer eso?"</strong></em></p>

<p>Y más en esta época de IAs escribiendo proyectos completos.<br>
Quien carpinchos querría desactivar las sugerencias de autocompletado en línea.</p>

<p>Bueno, yo, ¿por qué? Por lo que ustedes también saben, me la paso más tiempo borrando y cuidándome de no tocar <code>Tab</code> o <code>Enter</code> o <code>flecha derecha</code> que escribiendo. Así que investigue eso, fin.</p>
<h2>
  
  
  Versión corta:
</h2>

<p>estos son los que me molestaban a mí, al menos.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"editor.inlineSuggest.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.hover.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>pero si quieren saber mas</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxx8m45lpd0xocuzbeysl.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxx8m45lpd0xocuzbeysl.jpg" alt=" " width="450" height="246"></a></p>




<h2>
  
  
  ✅ <strong>Desactivar las sugerencias de autocompletado (IntelliSense)</strong>
</h2>

<ol>
<li>Abre <strong>Settings</strong> (<code>Ctrl + ,</code>).</li>
<li>Busca: <strong>"suggest"</strong>
</li>
<li>Desactiva:

<ul>
<li><strong>Editor: Quick Suggestions</strong></li>
<li><strong>Editor: Suggest On Trigger Characters</strong></li>
<li><strong>Editor: Accept Suggestion On Enter</strong></li>
<li><strong>Editor: Suggest Show Words</strong></li>
<li><strong>Editor: Snippet Suggestions</strong></li>
<li><strong>Editor: Parameter Hints Enabled</strong></li>
<li>
<strong>Editor: Inline Suggest Enabled</strong> (si usas GitHub Copilot u otros)</li>
</ul>
</li>
</ol>

<p>También puedes hacerlo en <code>settings.json</code> a nivel de usuario, pero yo prefiero siempre hacerlo a nivel de proyecto:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"editor.quickSuggestions"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.suggestOnTriggerCharacters"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.acceptSuggestionOnEnter"</span><span class="p">:</span><span class="w"> </span><span class="s2">"off"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.suggest.showWords"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.snippetSuggestions"</span><span class="p">:</span><span class="w"> </span><span class="s2">"none"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.parameterHints.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.inlineSuggest.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  ❌ <strong>Desactivar GitHub Copilot (si lo tienes instalado)</strong>
</h2>

<ol>
<li>Pulsa <code>Ctrl + Shift + P</code>.</li>
<li>Ejecuta: <strong>“Copilot: Disable”</strong>
O directamente desinstálalo desde <em>Extensions</em>.</li>
</ol>




<h2>
  
  
  ❌ <strong>Desactivar extensiones de autocompletado (TypeScript, HTML, etc.)</strong>
</h2>

<p>VS Code usa lenguaje por defecto, pero puedes quitar extensiones adicionales.</p>

<p>Ve a <strong>Extensions</strong> (<code>Ctrl + Shift + X</code>) y desactiva o desinstala las que den sugerencias extra.</p>




<h2>
  
  
  🔥 Desactivar <em>todo</em> lo relacionado al autocompletado (versión extrema) 😱
</h2>

<p>En <code>settings.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"editor.quickSuggestions"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.suggestOnTriggerCharacters"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.wordBasedSuggestions"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.parameterHints.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.inlineSuggest.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.hover.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.codeActionsOnSave"</span><span class="p">:</span><span class="w"> </span><span class="p">{},</span><span class="w">
  </span><span class="nl">"typescript.suggest.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"javascript.suggest.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editor.lightbulb.enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Esto deja VS Code casi sin asistencias automáticas.</p>




