---
Title: Resolviendo errores al conformar protocolos "actor-isolated".
Description: 
Author: GoyesDev
Date: 2026-01-16T21:19:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Swift 6 introdujo varios cambios para promover un entorno concurrente seguro. Entre ellos está el ejecutar el código en el hilo principal por defecto y también conformar protocolos de forma aislada por concurrencia ("actor-isolated").</p>




<h2>
  
  
  Contexto
</h2>

<ul>
<li>En Swift, un actor garantiza exclusión mutua: su estado interno solo puede ser accedido desde dentro de su propio aislamiento de concurrencia.</li>
<li>Cuando un protocolo define una propiedad o método, por defecto se interpreta como aislada al contexto del tipo que la implementa. Eso quiere decir que si un actor conforma ese protocolo, esa propiedad o método estaría aislada al actor y para accederla desde fuera tendría que usar await.</li>
</ul>




<h2>
  
  
  ¿Qué pasa con <code>nonisolated</code>?
</h2>

<p>Cuando cierto código está aislado por actor (actor-isolated) y va a ser invocado por otro actor, se necesita envolver el llamado con async/await. Si, en cambio, el código no está aislado por el actor (está marcado como nonisolated), estoy diciendo al compilador que no necesito garantizar que el cliente del código esté en el mismo actor.</p>

<p>Cuando marco una propiedad o método de un protocolo como <code>nonisolated</code>, le estoy diciendo al compilador:</p>

<blockquote>
<p>"Este requisito no depende del aislamiento de concurrencia del tipo que lo implemente. Puede ser accedido de manera síncrona y sin <code>await</code>, incluso si la conformidad la da un actor".</p>
</blockquote>

<p>Por ejemplo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">protocol</span> <span class="kt">IdentifiableEntity</span> <span class="p">{</span>
    <span class="k">var</span> <span class="nv">id</span><span class="p">:</span> <span class="kt">String</span> <span class="p">{</span> <span class="k">get</span> <span class="p">}</span> <span class="c1">// aislada si lo implementa un actor</span>
    <span class="kd">nonisolated</span> <span class="k">var</span> <span class="nv">typeName</span><span class="p">:</span> <span class="kt">String</span> <span class="p">{</span> <span class="k">get</span> <span class="p">}</span> <span class="c1">// no depende del aislamiento</span>
<span class="p">}</span>
<span class="kd">actor</span> <span class="kt">User</span><span class="p">:</span> <span class="kt">IdentifiableEntity</span> <span class="p">{</span>
    <span class="k">let</span> <span class="nv">id</span><span class="p">:</span> <span class="kt">String</span>
    <span class="nf">init</span><span class="p">(</span><span class="nv">id</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="p">{</span> <span class="k">self</span><span class="o">.</span><span class="n">id</span> <span class="o">=</span> <span class="n">id</span> <span class="p">}</span>
    <span class="kd">nonisolated</span> <span class="k">var</span> <span class="nv">typeName</span><span class="p">:</span> <span class="kt">String</span> <span class="p">{</span> <span class="s">"User"</span> <span class="p">}</span>
<span class="p">}</span>
<span class="k">let</span> <span class="nv">u</span> <span class="o">=</span> <span class="kt">User</span><span class="p">(</span><span class="nv">id</span><span class="p">:</span> <span class="s">"123"</span><span class="p">)</span>
<span class="c1">// Para acceder al id necesito await porque está aislado al actor:</span>
<span class="k">let</span> <span class="nv">id</span> <span class="o">=</span> <span class="k">await</span> <span class="n">u</span><span class="o">.</span><span class="n">id</span>  
<span class="c1">// Pero el typeName es nonisolated, así que puedo llamarlo directamente:</span>
<span class="nf">print</span><span class="p">(</span><span class="n">u</span><span class="o">.</span><span class="n">typeName</span><span class="p">)</span>
</code></pre>

</div>






<h2>
  
  
  ¿Qué pasa cuando conformamos un protocolo en contextos aislados (actor-isolated)?
</h2>

<p>Podríamos crear un protocolo con un método o propiedad de tipo <code>nonisolated</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">protocol</span> <span class="kt">SomeProtocol</span> <span class="p">{</span>
  <span class="kd">nonisolated</span> <span class="k">var</span> <span class="nv">name</span><span class="p">:</span> <span class="kt">String</span> <span class="p">{</span> <span class="k">get</span> <span class="p">}</span>
<span class="p">}</span>
<span class="kd">class</span> <span class="kt">SomeClass</span><span class="p">:</span> <span class="kt">SomeProtocol</span> <span class="p">{</span>
  <span class="k">var</span> <span class="nv">name</span><span class="p">:</span> <span class="kt">String</span>

  <span class="nf">init</span><span class="p">(</span><span class="nv">name</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">self</span><span class="o">.</span><span class="n">name</span> <span class="o">=</span> <span class="n">name</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Sin embargo, al tratar de conformar directamente este protocolo obtenemos el error:</p>

<blockquote>
<p>Conformance of 'SomeClass' to protocol 'SomeProtocol' crosses into main actor-isolated code and can cause data races</p>
</blockquote>

<p>Como en Swift 6 el código se ejecuta por defecto en el actor principal ("main-actor"), se asume que <code>SomeClass</code> está en el hilo principal. No obstante, la propiedad <code>name</code> del protocolo está en un contexto <code>nonisolated</code>, lo cual significa que podría ser accedido sin usar <code>await</code> desde otros hilos. Esto puede provocar una condición de carrera.</p>




<h2>
  
  
  Solución 1: Que todo sea <code>nonisolated</code>
</h2>

<p>En este escenario, la clase <code>SomeClass</code> va a ser declarada como <code>nonisolated</code>. Se supone que originalmente la clase estaba aislada en un actor: ¿por qué querríamos alterar a TODA la clase cuando solo una pequeña parte requiere ser corregida? Además, muy probablemente los clientes de esta clase también estén asilados en el actor principal, así que no tiene sentido hacer que <code>SomeClass</code> ahora sea <code>nonisolated</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">nonisolated</span> <span class="kd">class</span> <span class="kt">SomeClass</span><span class="p">:</span> <span class="kt">SomeProtocol</span> <span class="p">{</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Solución 2: Conformar el protocolo en el Actor Principal
</h2>

<p>En este caso, podemos conformar el protocolo dentro del actor principal:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">class</span> <span class="kt">SomeClass</span><span class="p">:</span> <span class="kd">@MainActor</span> <span class="kt">SomeProtocol</span> <span class="p">{</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Aquí, <code>SomeClass</code> sigue estando bajo el <code>MainActor</code>, junto con la conformidad al protocolo. Luego, cualquier acceso a <code>name</code> se entiende que ocurre siempre en el Main Actor, por lo que <strong>no hay que marcarla <code>nonisolated</code></strong>.</p>

<p>No obstante, si el protocolo tenía un requisito que parecía necesitar <code>nonisolated</code>, al aislar la conformidad con <code>@MainActor</code> ya no hace falta así que: ¿De qué sirvió marcar <code>nonisolated</code> en primer lugar?</p>




<h2>
  
  
  Resolviendo: Conformación de <code>Encodable</code> puede provocar condiciones de carrera cuando se relacione con código aislado en el hilo principal
</h2>

<p>El siguiente código trae un error de compilación:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">Person</span><span class="p">:</span> <span class="kt">Codable</span> <span class="p">{</span>
  <span class="k">var</span> <span class="nv">name</span><span class="p">:</span> <span class="kt">String</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>Conformance of 'Person' to protocol 'Encodable' crosses into main actor-isolated code and can cause data races</p>
</blockquote>

<p>El error surge porque conformar este protocolo desde una estructura o clase aislada al actor principal genera que el método <code>encode(to:)</code> esté aislado en el actor principal. Sin embargo, el protocolo requiere que ese método sea <code>nonisolated</code>.</p>

<p>Como la tarea de decodificar el DTO puede ser computacionalmente costosa, es mejor hacerla en un hilo de segundo plano. Si la estructura es muy pequeña, entonces se puede considerar hacer la decodificación en primer plano. Con base en esto, podríamos tener estas dos soluciones:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="c1">// nonisolated (no tiene @MainActor)</span>
<span class="kd">nonisolated</span> <span class="kd">struct</span> <span class="kt">Sample</span><span class="p">:</span> <span class="kt">Decodable</span> <span class="p">{</span>
  <span class="k">var</span> <span class="nv">name</span><span class="p">:</span> <span class="kt">String</span>
  <span class="kd">nonisolated</span> <span class="kd">enum</span> <span class="kt">CodingKeys</span><span class="p">:</span> <span class="kt">CodingKey</span> <span class="p">{</span>
    <span class="k">case</span> <span class="n">name</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// @MainActor (no tiene nonisolated)
struct Person: @MainActor Decodable {
  var name: String
  nonisolated enum CodingKeys: CodingKey {
    case name
  }
}
</code></pre>

</div>



<p>Notar que, en caso de necesitar tener <code>CodingKeys</code>, es necesario marcar la enumeración con <code>nonisolated</code> porque <code>CodingKeys</code> necesita conformar <code>CustomDebugStringConvertable</code>, que requiere que <code>Self</code> sea <code>Sendable</code> (cosa que no se puede si es <code>@MainActor</code>)</p>




<h2>
  
  
  Referencia
</h2>

<ul>
<li><a href="https://www.donnywals.com/solving-actor-isolated-protocol-conformance-related-errors-in-swift-6-2/" rel="noopener noreferrer">Solving actor-isolated protocol conformance related errors in Swift 6.2</a></li>
</ul>

