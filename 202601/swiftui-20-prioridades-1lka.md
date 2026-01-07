---
Title: SwiftUI #20: Prioridades
Description: 
Author: David Goyes
Date: 2026-01-07T21:39:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>Un <code>Stack</code> divide el espacio de forma equidistante entre las vistas. Si las vistas no caben, asigna un tamaño fijo a los <code>Image</code> y reduce el tamaño de <code>Text</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">HStack</span> <span class="p">{</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Hola Mundo"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="n">title</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">lineLimit</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
      <span class="kt">Image</span><span class="p">(</span><span class="nv">systemName</span><span class="p">:</span> <span class="s">"circle"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="nf">system</span><span class="p">(</span><span class="nv">size</span><span class="p">:</span> <span class="mi">100</span><span class="p">))</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Hola Mundo"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="n">title</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">lineLimit</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F21feug03o87vv66540d7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F21feug03o87vv66540d7.png" alt=" " width="557" height="145"></a></p>

<p>En el ejemplo anterior, el contenido de los campos <code>Text</code> está truncado por el uso de <code>.lineLimit(1)</code>. De lo contrario, ocuparía más de una línea.</p>

<p>Para que un <code>Text</code> se muestre por completo, se debe cambiar su prioridad.</p>

<ul>
<li>
<a href="https://developer.apple.com/documentation/swiftui/view/layoutpriority(_:)" rel="noopener noreferrer"><code>layoutPriority(_:)</code></a>. La prioridad por defecto de una vista para pintarse es 0. Una vista tomarea tanto espacio como le sea posible, entre más alta sea su prioridad.</li>
<li>
<a href="https://developer.apple.com/documentation/swiftui/view/fixedsize(horizontal:vertical:)" rel="noopener noreferrer"><code>fixedSize(horizontal:vertical:)</code></a>: Fija la vista a su tamaño ideal. Si no se especifica ningún argumento, el tamaño es fijado en ambas dimensiones.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">HStack</span> <span class="p">{</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Hola Mundo"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="n">title</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">lineLimit</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
      <span class="kt">Image</span><span class="p">(</span><span class="nv">systemName</span><span class="p">:</span> <span class="s">"circle"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="nf">system</span><span class="p">(</span><span class="nv">size</span><span class="p">:</span> <span class="mi">100</span><span class="p">))</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Hola Mundo"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="n">title</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">lineLimit</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0n7djano4zrxqietp97g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0n7djano4zrxqietp97g.png" alt=" " width="566" height="147"></a></p>

<p><a href="https://developer.apple.com/documentation/swiftui/view/fixedsize(horizontal:vertical:)" rel="noopener noreferrer"><code>fixedSize(horizontal:vertical:)</code></a> tiene incluso más prioridad que <a href="https://developer.apple.com/documentation/swiftui/view/layoutpriority(_:)" rel="noopener noreferrer"><code>layoutPriority(_:)</code></a> puesto que si <code>fixedSize(horizontal:vertical:)</code> está activo en una vista, esta tomará el tamaño que necesita, sin importar la prioridad de los otros elementos.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">HStack</span> <span class="p">{</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Hola Mundo"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="n">title</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">lineLimit</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">fixedSize</span><span class="p">()</span>
      <span class="kt">Image</span><span class="p">(</span><span class="nv">systemName</span><span class="p">:</span> <span class="s">"circle"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="nf">system</span><span class="p">(</span><span class="nv">size</span><span class="p">:</span> <span class="mi">100</span><span class="p">))</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Hola Mundo"</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">font</span><span class="p">(</span><span class="o">.</span><span class="n">title</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">lineLimit</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">layoutPriority</span><span class="p">(</span><span class="mi">100</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffu90onf7y596suqd9try.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffu90onf7y596suqd9try.png" alt=" " width="554" height="139"></a></p>

