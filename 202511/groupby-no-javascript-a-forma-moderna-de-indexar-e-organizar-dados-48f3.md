---
Title: GroupBy no JavaScript: a forma fácil de indexar e organizar dados
Description: 
Author: Angela Caldas
Date: 2025-11-14T21:29:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>Quantas vezes você preciso usar <code>reduce</code> para arrays de objetos? No meu artigo anterior sobre <a href="https://dev.to/sucodelarangela/indexacao-de-dados-para-aplicacoes-frontend-44dp">Indexação de dados para aplicações front-end</a>, explorei exatamente isso, mostrando como organizar estruturas complexas usando <code>reduce</code> e até <code>Map</code>.</p>

<p>Mas, convenhamos, <code>reduce</code> tende a deixar o código complexo e difícil de entender... </p>

<p>Quer uma boa notícia? Desde o lançamento do <a href="https://tc39.es/ecma262/2024/" rel="noopener noreferrer"><strong>ECMAScript 2024</strong></a> temos uma forma muito mais simples, expressiva e legível para fazer isso, utilizando os métodos <code>Object.groupBy</code> e <code>Map.groupBy</code>.</p>

<h4>
  
  
  Sumário
</h4>

<blockquote>
<p><em>Por que agrupar dados?<br>
Object.groupBy<br>
Map.groupBy<br>
Conclusão</em></p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1em2a3gjsx2vr47yif8h.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1em2a3gjsx2vr47yif8h.gif" alt="Group hug!" width="488" height="275"></a></p>




<h2>
  
  
  Por que agrupar dados?
</h2>

<p>Agrupar dados é uma forma de indexação: você pega uma lista linear e reorganiza tudo usando uma chave. Esse tipo de organização melhora a performance, além de facilitar buscas e o consumo de dados em interfaces dinâmicas.</p>

<p>Antes dessas novas APIs, recorrer ao <code>reduce</code> era quase obrigatório. Funcionava, claro, mas exigia escrever mais código do que o necessário e a legibilidade acabava sofrendo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Como fazíamos antes</span>
<span class="kd">const</span> <span class="nx">produtos</span> <span class="o">=</span> <span class="p">[</span>
  <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">"</span><span class="s2">iPhone</span><span class="dl">"</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">"</span><span class="s2">eletrônicos</span><span class="dl">"</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">5000</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Notebook</span><span class="dl">"</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">"</span><span class="s2">eletrônicos</span><span class="dl">"</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">3000</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Camiseta</span><span class="dl">"</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">"</span><span class="s2">roupas</span><span class="dl">"</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">50</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Calça</span><span class="dl">"</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">"</span><span class="s2">roupas</span><span class="dl">"</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">120</span> <span class="p">},</span>
<span class="p">];</span>

<span class="kd">const</span> <span class="nx">agrupados</span> <span class="o">=</span> <span class="nx">produtos</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">acc</span><span class="p">,</span> <span class="nx">produto</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">categoria</span> <span class="o">=</span> <span class="nx">produto</span><span class="p">.</span><span class="nx">categoria</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">acc</span><span class="p">[</span><span class="nx">categoria</span><span class="p">])</span> <span class="nx">acc</span><span class="p">[</span><span class="nx">categoria</span><span class="p">]</span> <span class="o">=</span> <span class="p">[];</span>
  <span class="nx">acc</span><span class="p">[</span><span class="nx">categoria</span><span class="p">].</span><span class="nf">push</span><span class="p">(</span><span class="nx">produto</span><span class="p">);</span>
  <span class="k">return</span> <span class="nx">acc</span><span class="p">;</span>
<span class="p">},</span> <span class="p">{});</span>
</code></pre>

</div>






<h2>
  
  
  Object.groupBy
</h2>

<p>O <strong>Object.groupBy</strong> chega para resolver o agrupamento mais simples — aquele em que a chave é uma <em>string</em>. Você passa a lista e uma função que retorna a chave e pronto. O JavaScript devolve um objeto organizado, sem precisar criar inicializações manuais, verificações ou <em>boilerplate</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">produtosPorCategoria</span> <span class="o">=</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">groupBy</span><span class="p">(</span>
  <span class="nx">produtos</span><span class="p">,</span> <span class="c1">// aqui a lista</span>
  <span class="p">(</span><span class="nx">produto</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">produto</span><span class="p">.</span><span class="nx">categoria</span> <span class="c1">// e aqui a função</span>
<span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">produtosPorCategoria</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">produtosPorCategoria</span> <span class="k">instanceof</span> <span class="nb">Object</span><span class="p">);</span> <span class="c1">// true</span>
</code></pre>

</div>



<p>O resultado é direto, limpo e fácil de interpretar:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="nl">eletrônicos</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">'</span><span class="s1">iPhone</span><span class="dl">'</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">'</span><span class="s1">eletrônicos</span><span class="dl">'</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">5000</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Notebook</span><span class="dl">'</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">'</span><span class="s1">eletrônicos</span><span class="dl">'</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">3000</span> <span class="p">}</span>
  <span class="p">],</span>
  <span class="nx">roupas</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Camiseta</span><span class="dl">'</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">'</span><span class="s1">roupas</span><span class="dl">'</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">50</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">nome</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Calça</span><span class="dl">'</span><span class="p">,</span> <span class="na">categoria</span><span class="p">:</span> <span class="dl">'</span><span class="s1">roupas</span><span class="dl">'</span><span class="p">,</span> <span class="na">preco</span><span class="p">:</span> <span class="mi">120</span> <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p><small> <em>Voltar ao sumário</em> </small></p>




<h2>
  
  
  Map.groupBy
</h2>

<p>Já <strong>Map.groupBy</strong> atende aos casos em que a chave não pode ser tratada como texto comum. Quando você precisa agrupar usando:</p>

<ul>
<li>datas,</li>
<li>objetos,</li>
<li>símbolos,</li>
<li>ou qualquer valor que não deve virar <em>string</em>,</li>
</ul>

<p>é ele que faz a diferença:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">vendas</span> <span class="o">=</span> <span class="p">[</span>
  <span class="p">{</span> <span class="na">valor</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span> <span class="na">data</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-01-15</span><span class="dl">"</span><span class="p">)</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">valor</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span> <span class="na">data</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-01-15</span><span class="dl">"</span><span class="p">)</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">valor</span><span class="p">:</span> <span class="mi">150</span><span class="p">,</span> <span class="na">data</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="dl">"</span><span class="s2">2024-01-16</span><span class="dl">"</span><span class="p">)</span> <span class="p">},</span>
<span class="p">];</span>

<span class="kd">const</span> <span class="nx">vendasPorData</span> <span class="o">=</span> <span class="nb">Map</span><span class="p">.</span><span class="nf">groupBy</span><span class="p">(</span>
  <span class="nx">vendas</span><span class="p">,</span> <span class="c1">// novamente, a lista</span>
  <span class="p">(</span><span class="nx">venda</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">venda</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nf">toDateString</span><span class="p">()</span> <span class="c1">// e a função</span>
<span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">vendasPorData</span> <span class="k">instanceof</span> <span class="nb">Map</span><span class="p">);</span> <span class="c1">// true</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">vendasPorData</span><span class="p">);</span>

<span class="cm">/*
{ 
  'Sun Jan 14 2024': [ 
    {valor: 100, data: 'Sun Jan 14 2024...'}, 
    {valor: 200, data: 'Sun Jan 14 2024...'}, 
  ], 
  'Mon Jan 15 2024': [ 
    {valor: 150, data: 'Mon Jan 15 2024...'} 
  ] 
}
*/</span>
</code></pre>

</div>



<p>Acessar ou manipular os grupos fica mais explícito, o que ajuda bastante em projetos maiores:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">vendasDia15</span> <span class="o">=</span> <span class="nx">vendasPorData</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">Tue Jan 15 2024</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">vendasDia15</span><span class="p">);</span>
</code></pre>

</div>



<p>A principal vantagem é justamente essa: o Map mantém o tipo original da chave, o que evita conversões automáticas e ambiguidades — algo que pode gerar problemas silenciosos quando usamos objetos normais.</p>

<p>No fim das contas, Map.groupBy acaba sendo a alternativa mais flexível para quem lida com coleções diversas no JavaScript moderno.</p>

<p><small> <em>Voltar ao sumário</em> </small></p>




<h2>
  
  
  Conclusão
</h2>

<p>Os novos métodos de agrupamento chegam para simplificar algo que sempre fez parte do nosso cotidiano de desenvolvimento. Em vez de escrever blocos repetitivos com <code>reduce</code>, agora podemos contar com soluções nativas, pensadas exatamente para esse propósito.</p>

<p>Quando combinados com as técnicas que apresentei no artigo anterior, eles deixam o fluxo de indexação mais claro, mais direto e muito mais expressivo.</p>

<p>Se você trabalha com listas, filtros, tabelas ou dashboards, vale experimentar essas APIs, provavelmente elas vão enxugar bastante o seu código.</p>




<h4>
  
  
  ⚠️ Nota sobre compatibilidade
</h4>

<blockquote>
<p><em>Se você está se perguntando sobre suporte nos navegadores, boa notícia:</em> segundo os dados do <a href="https://caniuse.com/?search=groupBy" rel="noopener noreferrer">Can I Use</a>, tanto <code>Object.groupBy</code> quanto <code>Map.groupBy</code> são amplamente suportados nos navegadores modernos.<br>
A única exceção relevante é o <strong>Internet Explorer</strong>, que não oferece suporte a essas APIs — e não receberá.</p>
</blockquote>

<p>Ou seja: se o seu público não usa IE, você pode adotar <code>groupBy</code> sem preocupações.</p>

