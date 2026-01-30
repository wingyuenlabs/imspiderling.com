---
Title: [SUI] LabeledContent
Description: 
Author: GoyesDev
Date: 2026-01-30T21:52:29.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://developer.apple.com/documentation/swiftui/labeledcontent" rel="noopener noreferrer"><code>LabeledContent</code></a> es un contenedor que adjunta una etiqueta a un control.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">selection</span><span class="p">:</span> <span class="kt">Int</span> <span class="o">=</span> <span class="mi">1</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">stepperValue</span> <span class="o">=</span> <span class="mi">1</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">NavigationView</span> <span class="p">{</span>
      <span class="kt">Form</span> <span class="p">{</span>
        <span class="kt">LabeledContent</span><span class="p">(</span><span class="s">"Etiqueta visible"</span><span class="p">)</span> <span class="p">{</span>
          <span class="kt">HStack</span> <span class="p">{</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Tengo: </span><span class="se">\(</span><span class="n">stepperValue</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
            <span class="kt">Stepper</span><span class="p">(</span><span class="s">"Etiqueta oculta"</span><span class="p">,</span> <span class="nv">value</span><span class="p">:</span> <span class="n">$stepperValue</span><span class="p">,</span> <span class="nv">in</span><span class="p">:</span> <span class="mi">0</span><span class="o">...</span><span class="mi">10</span><span class="p">)</span>
              <span class="o">.</span><span class="nf">labelsHidden</span><span class="p">()</span>
          <span class="p">}</span>
        <span class="p">}</span>
        <span class="kt">Picker</span><span class="p">(</span><span class="s">"Selected Value"</span><span class="p">,</span> <span class="nv">selection</span><span class="p">:</span> <span class="n">$selection</span><span class="p">)</span> <span class="p">{</span>
          <span class="kt">Text</span><span class="p">(</span><span class="s">"Option 1"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
          <span class="kt">Text</span><span class="p">(</span><span class="s">"Option 2"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvn7owft238fwmi0ozsnc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvn7owft238fwmi0ozsnc.png" alt=" " width="344" height="95"></a></p>

<h2>
  
  
  Etiqueta con vista personalizada
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">selection</span><span class="p">:</span> <span class="kt">Int</span> <span class="o">=</span> <span class="mi">1</span>
  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">stepperValue</span> <span class="o">=</span> <span class="mi">1</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">NavigationView</span> <span class="p">{</span>
      <span class="kt">Form</span> <span class="p">{</span>
        <span class="kt">LabeledContent</span> <span class="p">{</span>
          <span class="kt">HStack</span> <span class="p">{</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Tengo: </span><span class="se">\(</span><span class="n">stepperValue</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
            <span class="kt">Stepper</span><span class="p">(</span><span class="s">"Etiqueta oculta"</span><span class="p">,</span> <span class="nv">value</span><span class="p">:</span> <span class="n">$stepperValue</span><span class="p">,</span> <span class="nv">in</span><span class="p">:</span> <span class="mi">0</span><span class="o">...</span><span class="mi">10</span><span class="p">)</span>
              <span class="o">.</span><span class="nf">labelsHidden</span><span class="p">()</span>
          <span class="p">}</span>
        <span class="p">}</span> <span class="nv">label</span><span class="p">:</span> <span class="p">{</span>
          <span class="kt">HStack</span> <span class="p">{</span>
            <span class="kt">Image</span><span class="p">(</span><span class="nv">systemName</span><span class="p">:</span> <span class="s">"pencil"</span><span class="p">)</span>
            <span class="kt">Text</span><span class="p">(</span><span class="s">"Etiqueta visible"</span><span class="p">)</span>
          <span class="p">}</span>
          <span class="kt">Text</span><span class="p">(</span><span class="s">"Subtitle"</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="kt">Picker</span><span class="p">(</span><span class="s">"Selected Value"</span><span class="p">,</span> <span class="nv">selection</span><span class="p">:</span> <span class="n">$selection</span><span class="p">)</span> <span class="p">{</span>
          <span class="kt">Text</span><span class="p">(</span><span class="s">"Option 1"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
          <span class="kt">Text</span><span class="p">(</span><span class="s">"Option 2"</span><span class="p">)</span><span class="o">.</span><span class="nf">tag</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyy12oodjdkp8yukhvg6t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyy12oodjdkp8yukhvg6t.png" alt=" " width="343" height="108"></a></p>

