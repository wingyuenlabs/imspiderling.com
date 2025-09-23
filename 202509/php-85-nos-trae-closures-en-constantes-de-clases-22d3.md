---
Title: PHP 8.5 nos trae closures en constantes de clases.
Description: 
Author: oskar calvo
Date: 2025-09-23T21:52:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>La verdad es que esta nueva funcionalidad me deja frío, y es añadir una constante que puede ejecutar una función anónima no termina de convencerme.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="kd">class</span> <span class="nc">Example</span> <span class="p">{</span>
    <span class="k">public</span> <span class="k">const</span> <span class="no">VALIDATOR</span> <span class="o">=</span> <span class="k">static</span> <span class="k">function</span><span class="p">(</span><span class="nv">$value</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="o">!</span><span class="k">empty</span><span class="p">(</span><span class="nv">$value</span><span class="p">);</span>
    <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<p>(fuente del código: <a href="https://pixicstudio.medium.com/php-8-5-new-developer-features-f9617311c590" rel="noopener noreferrer">https://pixicstudio.medium.com/php-8-5-new-developer-features-f9617311c590</a>) <br>
Para resolver eso mismo podemos crear un <a href="https://www.kai-sassnowski.com/post/php-functors-1/" rel="noopener noreferrer">functor</a> o podemos usar también un <a href="https://www.php.net/manual/en/language.oop5.traits.php" rel="noopener noreferrer">trait</a>, o incluso una clase con un método estático (pero mejor los traits).</p>

<p>Un functor al ser una clase se puede inyectar sin problemas en otras clases y un trait está precisamente pensado para ser usado dentro de otras clases.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="kd">interface</span> <span class="nc">Functor</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">fmap</span><span class="p">(</span><span class="kt">callable</span> <span class="nv">$fn</span><span class="p">):</span> <span class="kt">Functor</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="kd">class</span> <span class="nc">Arr</span> <span class="kd">implements</span> <span class="nc">Functor</span>
<span class="p">{</span>
    <span class="k">protected</span> <span class="nv">$items</span><span class="p">;</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">__construct</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$items</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">items</span> <span class="o">=</span> <span class="nv">$items</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">fmap</span><span class="p">(</span><span class="kt">callable</span> <span class="nv">$fn</span><span class="p">):</span> <span class="kt">Functor</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nc">static</span><span class="p">(</span><span class="nb">array_map</span><span class="p">(</span><span class="nv">$fn</span><span class="p">,</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">items</span><span class="p">));</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>(fuente del código: <a href="https://www.kai-sassnowski.com/post/php-functors-1/" rel="noopener noreferrer">https://www.kai-sassnowski.com/post/php-functors-1/</a>) </p>

<p>Un functor o un servicio normal los plantearía en caso de que sea necesario conectar con la capa de Infraestructura (acceso a bbdd, a servicios externos de terceros, etc.), un trait por otro lado lo implementaría en caso de que correspondiere a la capa de Dominio para manipular datos pero es que no veo en que momento sería buena idea o interesante añadir una función anónima a una constante.</p>

<p>Así que lo dicho, no veo en qué momento sería buena idea añadir una función anónima a una constante. </p>

