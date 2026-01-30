---
Title: [SUI] DatePicker
Description: 
Author: GoyesDev
Date: 2026-01-30T21:22:51.000Z
Robots: noindex,nofollow
Template: index
---
<p>Usar <a href="https://developer.apple.com/documentation/swiftui/datepicker" rel="noopener noreferrer"><code>DatePicker</code></a> para seleccionar una sola fecha.</p>

<ul>
<li>
<a href="https://developer.apple.com/documentation/swiftui/datepicker/init(_:selection:in:displayedcomponents:)" rel="noopener noreferrer"><code>DatePicker.init(_:selection:in:displayedComponents:)</code></a>: <code>titleKey</code> es la etiqueta. <code>selection</code> es un <code>Binding</code> que almacena la <code>Date</code> seleccionada. <code>in</code> es un rango de selección de fechas. <code>displayComponents</code>, de tipo <a href="https://developer.apple.com/documentation/swiftui/datepickercomponents" rel="noopener noreferrer"><code>DatePickerComponents</code></a> es el tipo de valores almacenados que puede ser <a href="https://developer.apple.com/documentation/swiftui/datepickercomponents/date" rel="noopener noreferrer"><code>date</code></a> o <a href="https://developer.apple.com/documentation/swiftui/datepickercomponents/hourandminute" rel="noopener noreferrer"><code>hourAndMinute</code></a>.</li>
</ul>

<p>El estilo por defecto del <code>Picker</code> permite elegir la fecha con un calendario y la hora con una rueda.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">date</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
      <span class="kt">DatePicker</span><span class="p">(</span>
          <span class="s">"Start Date"</span><span class="p">,</span>
          <span class="nv">selection</span><span class="p">:</span> <span class="n">$date</span><span class="p">,</span>
          <span class="nv">displayedComponents</span><span class="p">:</span> <span class="p">[</span><span class="o">.</span><span class="n">date</span><span class="p">]</span>
      <span class="p">)</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh3lvexp714szu8a2ui1p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh3lvexp714szu8a2ui1p.png" alt=" " width="331" height="281"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">date</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
      <span class="kt">DatePicker</span><span class="p">(</span>
          <span class="s">"Start Date"</span><span class="p">,</span>
          <span class="nv">selection</span><span class="p">:</span> <span class="n">$date</span><span class="p">,</span>
          <span class="nv">displayedComponents</span><span class="p">:</span> <span class="p">[</span><span class="o">.</span><span class="n">date</span><span class="p">,</span> <span class="o">.</span><span class="n">hourAndMinute</span><span class="p">]</span>
      <span class="p">)</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0lx83ii55wzw98jozxm5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0lx83ii55wzw98jozxm5.png" alt=" " width="336" height="199"></a></p>

<h2>
  
  
  Rango de fechas
</h2>

<p>El rango de fechas puede ser cerrado o abierto. El siguiente ejemplo presenta un <code>DatePicker</code> que permite elegir fechas desde hoy hasta un futuro distante por medio del rango <code>Date()...</code>. En caso de necesitar lo contrario (i.e. desde un pasado distante hasta hoy) se usa <code>...Date()</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">date</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
      <span class="kt">DatePicker</span><span class="p">(</span>
          <span class="s">"Start Date"</span><span class="p">,</span>
          <span class="nv">selection</span><span class="p">:</span> <span class="n">$date</span><span class="p">,</span>
          <span class="nv">in</span><span class="p">:</span> <span class="kt">Date</span><span class="p">()</span><span class="o">...</span><span class="p">,</span>
          <span class="nv">displayedComponents</span><span class="p">:</span> <span class="p">[</span><span class="o">.</span><span class="n">date</span><span class="p">]</span>
      <span class="p">)</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpw0ol87fkz7ol9rj2sbh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpw0ol87fkz7ol9rj2sbh.png" alt=" " width="397" height="313"></a></p>

<h2>
  
  
  Cambiar el estilo del <code>DatePicker</code>
</h2>

<p>Usar <a href="https://developer.apple.com/documentation/swiftui/view/datepickerstyle(_:)" rel="noopener noreferrer"><code>datePickerStyle(_:)</code></a> para definir el estilo de tipo <a href="https://developer.apple.com/documentation/swiftui/datepickerstyle" rel="noopener noreferrer"><code>DatePickerStyle</code></a>, que puede tomar valores predefinidos:</p>

<ul>
<li><a href="https://developer.apple.com/documentation/swiftui/datepickerstyle/automatic" rel="noopener noreferrer"><code>automatic</code></a></li>
<li>
<a href="https://developer.apple.com/documentation/swiftui/datepickerstyle/compact" rel="noopener noreferrer"><code>compact</code></a>: Muestra un botón para revelar un calendario o rueda.</li>
<li>
<a href="https://developer.apple.com/documentation/swiftui/datepickerstyle/graphical" rel="noopener noreferrer"><code>graphical</code></a>: Directamente se muestra el calendario.</li>
<li>
<a href="https://developer.apple.com/documentation/swiftui/datepickerstyle/wheel" rel="noopener noreferrer"><code>wheel</code></a>: Directamente se muestra una rueda.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">date</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">VStack</span> <span class="p">{</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Fecha: </span><span class="se">\(</span><span class="n">date</span><span class="o">.</span><span class="nf">formatted</span><span class="p">(</span><span class="o">.</span><span class="n">dateTime</span><span class="o">.</span><span class="nf">day</span><span class="p">()</span><span class="o">.</span><span class="nf">month</span><span class="p">())</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
      <span class="kt">DatePicker</span><span class="p">(</span>
        <span class="s">"Start Date"</span><span class="p">,</span>
        <span class="nv">selection</span><span class="p">:</span> <span class="n">$date</span><span class="p">,</span>
        <span class="nv">in</span><span class="p">:</span> <span class="kt">Date</span><span class="p">()</span><span class="o">...</span><span class="p">,</span>
        <span class="nv">displayedComponents</span><span class="p">:</span> <span class="p">[</span><span class="o">.</span><span class="n">date</span><span class="p">]</span>
      <span class="p">)</span>
      <span class="o">.</span><span class="nf">datePickerStyle</span><span class="p">(</span><span class="o">.</span><span class="n">wheel</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1uznayniq7sf5wzeuupv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1uznayniq7sf5wzeuupv.png" alt=" " width="335" height="199"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">date</span> <span class="o">=</span> <span class="kt">Date</span><span class="p">()</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">VStack</span> <span class="p">{</span>
      <span class="kt">Text</span><span class="p">(</span><span class="s">"Fecha: </span><span class="se">\(</span><span class="n">date</span><span class="o">.</span><span class="nf">formatted</span><span class="p">(</span><span class="o">.</span><span class="n">dateTime</span><span class="o">.</span><span class="nf">day</span><span class="p">()</span><span class="o">.</span><span class="nf">month</span><span class="p">())</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
      <span class="kt">DatePicker</span><span class="p">(</span>
        <span class="s">"Start Date"</span><span class="p">,</span>
        <span class="nv">selection</span><span class="p">:</span> <span class="n">$date</span><span class="p">,</span>
        <span class="nv">in</span><span class="p">:</span> <span class="kt">Date</span><span class="p">()</span><span class="o">...</span><span class="p">,</span>
        <span class="nv">displayedComponents</span><span class="p">:</span> <span class="p">[</span><span class="o">.</span><span class="n">date</span><span class="p">]</span>
      <span class="p">)</span>
      <span class="o">.</span><span class="nf">datePickerStyle</span><span class="p">(</span><span class="o">.</span><span class="n">graphical</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbhcwwi2lfbsjul2g6jry.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbhcwwi2lfbsjul2g6jry.png" alt=" " width="349" height="319"></a></p>

