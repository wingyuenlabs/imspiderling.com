---
Title: [SUI] Barra de búsqueda
Description: 
Author: GoyesDev
Date: 2026-02-02T22:02:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>Un <code>NavigationStack</code> puede incluir una barra de búsqueda, para lo cual se debe aplicar el siguiente modificador:</p>

<ul>
<li>
<a href="https://developer.apple.com/documentation/swiftui/view/searchable(text:tokens:suggestedtokens:placement:prompt:token:)-9q3oc" rel="noopener noreferrer"><code>searchable(text:tokens:suggestedTokens:placement:prompt:token:)</code></a>: <code>text</code> es el texto introducido por el usuario. <code>tokens</code> sigue el rastro de los tokens mostrados al usuario. <code>suggestedTokens</code> tiene una lista de los tokens para sugerir valores al usuario. <code>placement</code> es la ubicación de la barra (que puede ser <code>automatic</code>, <code>navigationBarDrawer</code>, <a href="https://developer.apple.com/documentation/swiftui/searchfieldplacement/navigationbardrawer(displaymode:)" rel="noopener noreferrer"><code>navigationBarDrawer(displayMode:)</code></a>, <code>sidebar</code> y <code>toolbar</code>). <code>prompt</code> es el placeholder. <code>token</code> son las vistas para mostrar los tokens.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">Person</span><span class="p">:</span> <span class="kt">Identifiable</span> <span class="p">{</span>
  <span class="k">let</span> <span class="nv">id</span> <span class="o">=</span> <span class="kt">UUID</span><span class="p">()</span>
  <span class="k">let</span> <span class="nv">name</span><span class="p">:</span> <span class="kt">String</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">private</span> <span class="k">let</span> <span class="nv">people</span><span class="p">:</span> <span class="p">[</span><span class="kt">Person</span><span class="p">]</span> <span class="o">=</span> <span class="p">[</span>
  <span class="o">.</span><span class="nf">init</span><span class="p">(</span><span class="nv">name</span><span class="p">:</span> <span class="s">"David Goyes"</span><span class="p">),</span>
  <span class="o">.</span><span class="nf">init</span><span class="p">(</span><span class="nv">name</span><span class="p">:</span> <span class="s">"Midoriya Izuku"</span><span class="p">),</span>
  <span class="o">.</span><span class="nf">init</span><span class="p">(</span><span class="nv">name</span><span class="p">:</span> <span class="s">"Tanjiro Kamado"</span><span class="p">),</span>
  <span class="o">.</span><span class="nf">init</span><span class="p">(</span><span class="nv">name</span><span class="p">:</span> <span class="s">"David Beckham"</span><span class="p">),</span>
<span class="p">]</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>

  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">searchText</span><span class="p">:</span> <span class="kt">String</span> <span class="o">=</span> <span class="s">""</span>

  <span class="kd">private</span> <span class="k">var</span> <span class="nv">filteredPeople</span><span class="p">:</span> <span class="p">[</span><span class="kt">Person</span><span class="p">]</span> <span class="p">{</span>
    <span class="k">if</span> <span class="n">searchText</span><span class="o">.</span><span class="n">isEmpty</span> <span class="p">{</span>
      <span class="n">people</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="n">people</span><span class="o">.</span><span class="n">filter</span> <span class="p">{</span> <span class="nv">$0</span><span class="o">.</span><span class="n">name</span><span class="o">.</span><span class="nf">localizedStandardContains</span><span class="p">(</span><span class="n">searchText</span><span class="p">)</span> <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">NavigationStack</span> <span class="p">{</span>
      <span class="kt">List</span><span class="p">(</span><span class="n">filteredPeople</span><span class="p">)</span> <span class="p">{</span> <span class="n">person</span> <span class="k">in</span>
        <span class="kt">Text</span><span class="p">(</span><span class="n">person</span><span class="o">.</span><span class="n">name</span><span class="p">)</span>
      <span class="p">}</span>
      <span class="o">.</span><span class="nf">navigationTitle</span><span class="p">(</span><span class="s">"Personas"</span><span class="p">)</span>
      <span class="o">.</span><span class="nf">searchable</span><span class="p">(</span><span class="nv">text</span><span class="p">:</span> <span class="n">$searchText</span><span class="p">,</span> <span class="nv">prompt</span><span class="p">:</span> <span class="s">"¿A quién busca?"</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>En iOS 26, <strong>POR DEFECTO</strong> la barra de búsqueda queda en la parte inferior de la pantalla porque tiene <code>SearchFieldPlacement</code> en <a href="https://developer.apple.com/documentation/swiftui/searchfieldplacement/automatic" rel="noopener noreferrer"><code>automatic</code></a>, que en iOS, MacOS y iPadOS sería lo mismo que <a href="https://developer.apple.com/documentation/swiftui/searchfieldplacement/toolbar" rel="noopener noreferrer"><code>toolbar</code></a>. Cuando la barra se vuelve el "first responder", queda encima del teclado cuando este se presenta.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm32fwddlxeosassjtyw6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm32fwddlxeosassjtyw6.png" alt=" " width="425" height="865"></a></p>

<p>Notar que para que funcionase el ejemplo anterior, <code>filteredPeople</code> se hizo una variable computada y no almacenada. De lo contrario, habría sido necesario usar una aproximación basada en <a href="https://developer.apple.com/documentation/swiftui/view/onchange(of:initial:_:)-8wgw9" rel="noopener noreferrer"><code>onChange(of:initial:_:)</code></a>, teniendo en cuenta que se debe marcar <code>initial=true</code> para que se cargue la información en <code>filteredPeople</code> en el arranque.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>

  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">searchText</span><span class="p">:</span> <span class="kt">String</span> <span class="o">=</span> <span class="s">""</span>

  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">filteredPeople</span><span class="p">:</span> <span class="p">[</span><span class="kt">Person</span><span class="p">]</span> <span class="o">=</span> <span class="p">[]</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">NavigationStack</span> <span class="p">{</span>
      <span class="kt">List</span><span class="p">(</span><span class="n">filteredPeople</span><span class="p">)</span> <span class="p">{</span> <span class="n">person</span> <span class="k">in</span>
        <span class="kt">Text</span><span class="p">(</span><span class="n">person</span><span class="o">.</span><span class="n">name</span><span class="p">)</span>
      <span class="p">}</span>
      <span class="o">.</span><span class="nf">navigationTitle</span><span class="p">(</span><span class="s">"Personas"</span><span class="p">)</span>
      <span class="o">.</span><span class="nf">searchable</span><span class="p">(</span><span class="nv">text</span><span class="p">:</span> <span class="n">$searchText</span><span class="p">,</span> <span class="nv">prompt</span><span class="p">:</span> <span class="s">"¿A quién busca?"</span><span class="p">)</span>
      <span class="o">.</span><span class="nf">onChange</span><span class="p">(</span><span class="nv">of</span><span class="p">:</span> <span class="n">searchText</span><span class="p">,</span> <span class="nv">initial</span><span class="p">:</span> <span class="kc">true</span><span class="p">)</span> <span class="p">{</span>
        <span class="nf">filterPeople</span><span class="p">()</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="kd">private</span> <span class="kd">func</span> <span class="nf">filterPeople</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">filteredPeople</span> <span class="o">=</span> <span class="k">if</span> <span class="n">searchText</span><span class="o">.</span><span class="n">isEmpty</span> <span class="p">{</span>
      <span class="n">people</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="n">people</span><span class="o">.</span><span class="n">filter</span> <span class="p">{</span> <span class="nv">$0</span><span class="o">.</span><span class="n">name</span><span class="o">.</span><span class="nf">localizedStandardContains</span><span class="p">(</span><span class="n">searchText</span><span class="p">)</span> <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Visibilidad de la barra de búsqueda
</h2>

<p>Cuando la barra de búsqueda usa la ubicación por defecto (i.e. <code>automatic</code>) o <code>toolbar</code>, siempre será visible. Si hacemos que aparezca en la barra de navegación con <a href="https://developer.apple.com/documentation/swiftui/searchfieldplacement/navigationbardrawer" rel="noopener noreferrer"><code>navigationBarDrawer</code></a>, entonces la barra de búsqueda desaparecerá cuando desplacemos la lista hacia arriba, y volverá a aparecer cuando la desplacemos hacia abajo.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftfq44t6epxbrx5q9h0iq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftfq44t6epxbrx5q9h0iq.png" alt=" " width="354" height="236"></a></p>

<p>Para que la barra de búsqueda siempre sea visible en la barra de navegación, se puede usar <a href="https://developer.apple.com/documentation/swiftui/searchfieldplacement/navigationbardrawer(displaymode:)" rel="noopener noreferrer"><code>navigationBarDrawer(displayMode:)</code></a>, pasando <a href="https://developer.apple.com/documentation/swiftui/searchfieldplacement/navigationbardrawerdisplaymode/always" rel="noopener noreferrer"><code>always</code></a> como argumento.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>

  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">searchText</span><span class="p">:</span> <span class="kt">String</span> <span class="o">=</span> <span class="s">""</span>

  <span class="kd">private</span> <span class="k">var</span> <span class="nv">filteredPeople</span><span class="p">:</span> <span class="p">[</span><span class="kt">Person</span><span class="p">]</span> <span class="p">{</span>
    <span class="k">if</span> <span class="n">searchText</span><span class="o">.</span><span class="n">isEmpty</span> <span class="p">{</span>
      <span class="n">people</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="n">people</span><span class="o">.</span><span class="n">filter</span> <span class="p">{</span> <span class="nv">$0</span><span class="o">.</span><span class="n">name</span><span class="o">.</span><span class="nf">localizedStandardContains</span><span class="p">(</span><span class="n">searchText</span><span class="p">)</span> <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">NavigationStack</span> <span class="p">{</span>
      <span class="kt">List</span><span class="p">(</span><span class="n">filteredPeople</span><span class="p">)</span> <span class="p">{</span> <span class="n">person</span> <span class="k">in</span>
        <span class="kt">Text</span><span class="p">(</span><span class="n">person</span><span class="o">.</span><span class="n">name</span><span class="p">)</span>
      <span class="p">}</span>
      <span class="o">.</span><span class="nf">navigationTitle</span><span class="p">(</span><span class="s">"Personas"</span><span class="p">)</span>
      <span class="o">.</span><span class="nf">searchable</span><span class="p">(</span><span class="nv">text</span><span class="p">:</span> <span class="n">$searchText</span><span class="p">,</span> <span class="nv">placement</span><span class="p">:</span> <span class="o">.</span><span class="nf">navigationBarDrawer</span><span class="p">(</span><span class="nv">displayMode</span><span class="p">:</span> <span class="o">.</span><span class="n">always</span><span class="p">),</span> <span class="nv">prompt</span><span class="p">:</span> <span class="s">"¿A quién busca?"</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzojiio7j2l3duqy83bnl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzojiio7j2l3duqy83bnl.png" alt=" " width="339" height="144"></a></p>

<h2>
  
  
  Activando la barra de búsqueda de forma programática
</h2>

<p>Se puede fijar/quitar el foco sobre una barra de búsqueda de forma programática a través del modificador <a href="https://developer.apple.com/documentation/swiftui/view/searchfocused(_:)" rel="noopener noreferrer"><code>searchFocused(_:)</code></a> usando un <code>Binding</code> de tipo <code>Bool</code> con el "property-wrapper" <a href="https://developer.apple.com/documentation/swiftui/focusstate" rel="noopener noreferrer"><code>FocusState</code></a>.</p>

<p>En el siguiente ejemplo, el botón "Buscar" del toolbar, pone <code>isFocused</code> en <code>true</code>, lo que enfoca la barra de búsqueda y muestra el teclado.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">ContentView</span><span class="p">:</span> <span class="kt">View</span> <span class="p">{</span>

  <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">searchText</span><span class="p">:</span> <span class="kt">String</span> <span class="o">=</span> <span class="s">""</span>
  <span class="c1">// ⚠️ Notar el uso de @FocusState</span>
  <span class="kd">@FocusState</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">isFocused</span><span class="p">:</span> <span class="kt">Bool</span>

  <span class="kd">private</span> <span class="k">var</span> <span class="nv">filteredPeople</span><span class="p">:</span> <span class="p">[</span><span class="kt">Person</span><span class="p">]</span> <span class="p">{</span>
    <span class="k">if</span> <span class="n">searchText</span><span class="o">.</span><span class="n">isEmpty</span> <span class="p">{</span>
      <span class="n">people</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="n">people</span><span class="o">.</span><span class="n">filter</span> <span class="p">{</span> <span class="nv">$0</span><span class="o">.</span><span class="n">name</span><span class="o">.</span><span class="nf">localizedStandardContains</span><span class="p">(</span><span class="n">searchText</span><span class="p">)</span> <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
    <span class="kt">NavigationStack</span> <span class="p">{</span>
      <span class="kt">List</span><span class="p">(</span><span class="n">filteredPeople</span><span class="p">)</span> <span class="p">{</span> <span class="n">person</span> <span class="k">in</span>
        <span class="kt">Text</span><span class="p">(</span><span class="n">person</span><span class="o">.</span><span class="n">name</span><span class="p">)</span>
      <span class="p">}</span>
      <span class="o">.</span><span class="nf">navigationTitle</span><span class="p">(</span><span class="s">"Personas"</span><span class="p">)</span>
      <span class="o">.</span><span class="nf">toolbar</span><span class="p">(</span><span class="nv">content</span><span class="p">:</span> <span class="p">{</span>
        <span class="kt">ToolbarItem</span><span class="p">(</span><span class="nv">placement</span><span class="p">:</span> <span class="o">.</span><span class="n">topBarTrailing</span><span class="p">)</span> <span class="p">{</span>
          <span class="kt">Button</span><span class="p">(</span><span class="s">"Buscar"</span><span class="p">,</span> <span class="nv">systemImage</span><span class="p">:</span> <span class="s">"magnifyingglass"</span><span class="p">)</span> <span class="p">{</span>
            <span class="c1">// ⚠️ Aquí se cambia isFocused para centrar el foco</span>
            <span class="c1">// en la barra de búsqueda</span>
            <span class="n">isFocused</span> <span class="o">=</span> <span class="kc">true</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">})</span>
      <span class="o">.</span><span class="nf">searchable</span><span class="p">(</span><span class="nv">text</span><span class="p">:</span> <span class="n">$searchText</span><span class="p">,</span> <span class="nv">placement</span><span class="p">:</span> <span class="o">.</span><span class="nf">navigationBarDrawer</span><span class="p">(</span><span class="nv">displayMode</span><span class="p">:</span> <span class="o">.</span><span class="n">automatic</span><span class="p">),</span> <span class="nv">prompt</span><span class="p">:</span> <span class="s">"¿A quién busca?"</span><span class="p">)</span>
      <span class="c1">// ⚠️ por medio de searchFocused se puede cambiar el foco</span>
      <span class="o">.</span><span class="nf">searchFocused</span><span class="p">(</span><span class="n">$isFocused</span><span class="p">)</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc00dylznjs7qtvu8xuq7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc00dylznjs7qtvu8xuq7.png" alt=" " width="346" height="244"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1t6ufvuk19p17zk187qt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1t6ufvuk19p17zk187qt.png" alt=" " width="337" height="180"></a></p>

