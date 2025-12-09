---
Title: Swift #10: Bucles
Description: 
Author: David Goyes
Date: 2025-12-09T21:54:53.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  <code>While</code>
</h2>

<p><code>while</code> valida una condición y ejecuta un bloque de código hasta que la condición se vuelva <code>false</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="k">var</span> <span class="nv">counter</span> <span class="o">=</span> <span class="mi">0</span>
<span class="k">while</span> <span class="n">counter</span> <span class="o">&lt;</span> <span class="mi">10</span> <span class="p">{</span>
  <span class="nf">print</span><span class="p">(</span><span class="n">counter</span><span class="p">)</span>
  <span class="n">counter</span> <span class="o">+=</span> <span class="mi">1</span>
<span class="p">}</span>
<span class="c1">// counter == 10</span>
</code></pre>

</div>



<h2>
  
  
  <code>Repeat while</code>
</h2>

<p>Si la condición es falsa en la primera vuelta, es posible que no se ejecute el bloque. Para ejecutarlo al menos una vez se debe usar <code>repeat while</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="k">var</span> <span class="nv">counter</span> <span class="o">=</span> <span class="mi">10</span>
<span class="k">repeat</span> <span class="p">{</span>
  <span class="nf">print</span><span class="p">(</span><span class="n">counter</span><span class="p">)</span>
  <span class="n">counter</span> <span class="o">+=</span> <span class="mi">1</span>
<span class="p">}</span> <span class="k">while</span> <span class="n">counter</span> <span class="o">&lt;</span> <span class="mi">10</span>
<span class="c1">// counter == 11</span>
</code></pre>

</div>



<h2>
  
  
  <code>For In</code>
</h2>

<p><code>for in</code> permite recorrer una colección de valores en orden. El sistema copia uno a uno los valores en una constante que puede usar en un bloque de código. El bucle finaliza cuando se termina la colección. La sintaxis es <code>for &lt;constant&gt; in &lt;collection&gt; { ... }</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="k">let</span> <span class="nv">input</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">5</span><span class="p">]</span>
<span class="k">for</span> <span class="n">value</span> <span class="k">in</span> <span class="n">input</span> <span class="p">{</span>
  <span class="nf">print</span><span class="p">(</span><span class="n">value</span> <span class="o">*</span> <span class="mi">2</span><span class="p">)</span>
<span class="p">}</span>
<span class="c1">// Imprime 2, 6, 10</span>
</code></pre>

</div>



<p>Cuando la constante del <code>for in</code> no se necesita, se puede reemplazar con un guion bajo.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="k">let</span> <span class="nv">input</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">5</span><span class="p">]</span>
<span class="k">for</span> <span class="n">_</span> <span class="k">in</span> <span class="n">input</span> <span class="p">{</span>
  <span class="nf">print</span><span class="p">(</span><span class="s">"Hola"</span><span class="p">)</span>
<span class="p">}</span>
<span class="c1">// Imprime: "Hola", "Hola", "Hola" </span>
</code></pre>

</div>



<p>La cláusula <code>where</code> también sirve en <code>for in</code> para solo ejecutar las iteraciones que cumplan con cierta condición. Por ejemplo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="k">let</span> <span class="nv">input</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">5</span><span class="p">,</span> <span class="mi">6</span><span class="p">]</span>
<span class="k">for</span> <span class="n">value</span> <span class="k">in</span> <span class="n">input</span> <span class="k">where</span> <span class="n">value</span> <span class="o">%</span> <span class="mi">3</span> <span class="o">!=</span> <span class="mi">0</span> <span class="p">{</span>
  <span class="nf">print</span><span class="p">(</span><span class="n">value</span> <span class="o">*</span> <span class="mi">2</span><span class="p">)</span>
<span class="p">}</span>
<span class="c1">// Imprime 2, 10</span>
</code></pre>

</div>



