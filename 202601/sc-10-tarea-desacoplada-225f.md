---
Title: SC #10: Tarea desacoplada
Description: 
Author: GoyesDev
Date: 2026-01-19T21:18:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>Una tarea desacoplada ("Detached Task") ejecuta una operación de forma asíncrona, fuera del contexto de concurrencia estructurado que la envuelve. No heredar este contexto implica que no se hereda:</p>

<ul>
<li>La prioridad del contexto</li>
<li>Estado de cancelación del contexto.</li>
</ul>

<p>Para crear la tarea se usa el método estático <a href="https://developer.apple.com/documentation/swift/task/detached(name:priority:operation:)-795w1" rel="noopener noreferrer"><code>detached(name:priority:operation:)</code></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kt">Task</span><span class="o">.</span><span class="n">detached</span> <span class="p">{</span>
  <span class="c1">// Ejecuta una operación de forma asíncrona fuera del contexto estructurado de concurrencia</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Considerar el siguiente ejemplo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">func</span> <span class="nf">detachedTaskExample</span><span class="p">()</span> <span class="k">async</span> <span class="p">{</span>
  <span class="kd">func</span> <span class="nf">asyncFunc</span><span class="p">(</span><span class="n">_</span> <span class="nv">string</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span> <span class="k">async</span> <span class="p">{</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">string</span><span class="p">)</span>
  <span class="p">}</span>
  <span class="k">await</span> <span class="nf">asyncFunc</span><span class="p">(</span><span class="s">"</span><span class="se">\(</span><span class="mi">1</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
  <span class="kt">Task</span><span class="o">.</span><span class="n">detached</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nf">asyncFunc</span><span class="p">(</span><span class="s">"</span><span class="se">\(</span><span class="mi">2</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
  <span class="p">}</span>
  <span class="k">await</span> <span class="nf">asyncFunc</span><span class="p">(</span><span class="s">"</span><span class="se">\(</span><span class="mi">3</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>El código anterior puede producir el siguiente resultado, aunque no se garantiza el orden:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>1
3
2
</code></pre>

</div>



<h2>
  
  
  Riesgo de cancelación de tarea desacoplada
</h2>

<p>Considerar el siguiente ejemplo, donde <code>longRunningAsyncOperation</code> es una tarea asíncrona que se demora (no termina antes de que se cancele) y <code>detachedTaskExample</code> es una tarea que invoca a <code>longRunningAsyncOperation</code> como subtarea y luego crea un <code>Task.detached</code> para invocar a <code>longRunningAsyncOperation</code> de forma desacoplada:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">func</span> <span class="nf">detachedTaskExample</span><span class="p">()</span> <span class="k">async</span> <span class="p">{</span>
  <span class="k">let</span> <span class="nv">outerTask</span> <span class="o">=</span> <span class="kt">Task</span> <span class="p">{</span>
    <span class="c1">/// Esta subtarea sí se va a cancelar</span>
    <span class="k">await</span> <span class="nf">longRunningAsyncOperation</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>

    <span class="c1">/// Esta tarea desacoplada NO hereda el estado de cancelación</span>
    <span class="kt">Task</span><span class="o">.</span><span class="nf">detached</span><span class="p">(</span><span class="nv">priority</span><span class="p">:</span> <span class="o">.</span><span class="n">background</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// Por eso este checkCancellation no hace nada</span>
      <span class="k">try</span> <span class="kt">Task</span><span class="o">.</span><span class="nf">checkCancellation</span><span class="p">()</span>

      <span class="c1">/// Por eso esta subtarea no se va a cancelar</span>
      <span class="k">await</span> <span class="nf">longRunningAsyncOperation</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="n">outerTask</span><span class="o">.</span><span class="nf">cancel</span><span class="p">()</span>
<span class="p">}</span>

<span class="kd">private</span> <span class="kd">func</span> <span class="nf">longRunningAsyncOperation</span><span class="p">(</span><span class="n">_</span> <span class="nv">id</span><span class="p">:</span> <span class="kt">Int</span><span class="p">)</span> <span class="k">async</span> <span class="p">{</span>
  <span class="k">do</span> <span class="p">{</span>
    <span class="nf">print</span><span class="p">(</span><span class="s">"Empezando tarea </span><span class="se">\(</span><span class="n">id</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
    <span class="k">try</span> <span class="k">await</span> <span class="kt">Task</span><span class="o">.</span><span class="nf">sleep</span><span class="p">(</span><span class="nv">for</span><span class="p">:</span> <span class="o">.</span><span class="nf">seconds</span><span class="p">(</span><span class="mi">5</span><span class="p">))</span>
    <span class="nf">print</span><span class="p">(</span><span class="s">"Terminé la tarea"</span><span class="p">)</span>
  <span class="p">}</span> <span class="k">catch</span> <span class="p">{</span>
    <span class="nf">print</span><span class="p">(</span><span class="s">"</span><span class="se">\(</span><span class="k">#function</span><span class="se">)</span><span class="s"> - Tarea </span><span class="se">\(</span><span class="n">id</span><span class="se">)</span><span class="s"> falló con error: </span><span class="se">\(</span><span class="n">error</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>La salida de este ejemplo es:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>Empezando tarea 1
longRunningAsyncOperation<span class="o">(</span>_:<span class="o">)</span> - Tarea 1 falló con error: CancellationError<span class="o">()</span>
Empezando tarea 2
Terminé la tarea
</code></pre>

</div>



<p>En este ejemplo:</p>

<ol>
<li>El primer llamado a <code>longRunningAsyncOperation</code> es cancelado correctamente porque se invoca dentro del contexto estructurado del <code>Task</code> cancelado.</li>
<li>En el segundo llamado a <code>longRunningAsyncOperation</code>, incluso aunque se haga <code>try Task.checkCancellation()</code>, no hay forma de detectar la cancelación porque precisamente <strong>NO estamos heredando este estado</strong>.</li>
</ol>

<p>Para cancelar una <code>Task</code> desacoplada, se debe tener una referencia a ella y cancelarla manualmente. Además, estas no son canceladas automáticamente cuando la referencia se libera, así que sí o sí de la debe cancelar manualmente.</p>

<h2>
  
  
  ¿Cuándo usar una tarea desacoplada?
</h2>

<p>Es válido crear una tarea desacoplada en escenarios donde <strong>una operación pueda correr de forma independiente</strong> y <strong>no requiera conexión</strong> al contexto contenedor, y sea aceptable que termine <strong>a pesar de que el contenedor haya sido cancelado</strong>.</p>

<p>En el siguiente ejemplo es válido crear la tarea desacoplada, teniendo en cuenta que no hay ninguna referencia a <code>self</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kt">Task</span><span class="o">.</span><span class="nf">detached</span><span class="p">(</span><span class="nv">priority</span><span class="p">:</span> <span class="o">.</span><span class="n">background</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">await</span> <span class="kt">DirectoryCleaner</span><span class="o">.</span><span class="nf">cleanup</span><span class="p">()</span>
<span class="p">}</span>
</code></pre>

</div>



