---
Title: [SUI] MultiDatePicker
Description: 
Author: GoyesDev
Date: 2026-01-30T21:23:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>Usar <a href="https://developer.apple.com/documentation/swiftui/multidatepicker" rel="noopener noreferrer"><code>MultiDatePicker</code></a> para elegir varias fechas.</p>

<ul>
<li>
<a href="https://developer.apple.com/documentation/swiftui/multidatepicker/init(_:selection:in:)" rel="noopener noreferrer"><code>MultiDatePicker.init(_:selection:in:)</code></a>. <code>titleKey</code> es la etiqueta. <code>selection</code> es un <code>Binding</code> que guarda un <code>Set</code> de <a href="https://developer.apple.com/documentation/Foundation/DateComponents" rel="noopener noreferrer"><code>DateComponents</code></a>. <code>in</code> define el rango de fechas.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">dates</span><span class="p">:</span> <span class="kt">Set</span><span class="o">&lt;</span><span class="kt">DateComponents</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">[]</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">title</span> <span class="o">=</span> <span class="s">""</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">VStack</span> <span class="p">{</span>
      <span class="kt">Text</span><span class="p">(</span><span class="n">title</span><span class="p">)</span>
      <span class="kt">MultiDatePicker</span><span class="p">(</span><span class="s">"Dates Available"</span><span class="p">,</span> <span class="nv">selection</span><span class="p">:</span> <span class="n">$dates</span><span class="p">)</span>
        <span class="o">.</span><span class="nf">onChange</span><span class="p">(</span><span class="nv">of</span><span class="p">:</span> <span class="n">dates</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">let</span> <span class="nv">days</span> <span class="o">=</span> <span class="n">dates</span><span class="o">.</span><span class="n">map</span> <span class="p">{</span> <span class="kt">String</span><span class="p">(</span><span class="nv">$0</span><span class="o">.</span><span class="n">day</span><span class="o">!</span><span class="p">)</span> <span class="p">}</span>
          <span class="n">title</span> <span class="o">=</span> <span class="n">days</span><span class="o">.</span><span class="nf">joined</span><span class="p">(</span><span class="nv">separator</span><span class="p">:</span> <span class="s">", "</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6wdtnw3opqz53w88o0zs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6wdtnw3opqz53w88o0zs.png" alt=" " width="358" height="302"></a></p>

