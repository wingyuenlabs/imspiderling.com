---
Title: Bora falar do padrão Strategy?
Description: 
Author: Bruno Souza
Date: 2026-02-10T22:04:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>Se você já se pegou pensando <em>“putz, esse <code>if/else</code> aqui tá crescendo demais”</em> ou <em>“cada nova regra quebra algo que já funcionava”</em>, é bem provável que o <strong>padrão Strategy</strong> seja exatamente o que você está procurando.</p>

<p>Neste post, vamos falar:</p>

<ul>
<li>O que é o Strategy</li>
<li>Qual problema ele resolve</li>
<li>Quando usar (e quando NÃO usar)</li>
<li>Exemplos práticos</li>
</ul>




<h2>
  
  
  O problema clássico
</h2>

<p>Imagine um sistema que precisa executar uma ação de formas diferentes dependendo do contexto:</p>

<ul>
<li>Tipos diferentes de cálculo</li>
<li>Regras que mudam por ambiente (QA, prod)</li>
<li>Variações de comportamento por cliente, país ou feature</li>
</ul>

<p>O caminho mais comum (e perigoso) costuma ser algo assim:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if tipo == A {
  faz isso
} else if tipo == B {
  faz aquilo
} else if tipo == C {
  faz outra coisa
}
</code></pre>

</div>



<p>Funciona? Funciona.<br>
Escala? Nem um pouco 😅</p>

<p>Cada nova regra:</p>

<ul>
<li>aumenta a complexidade</li>
<li>quebra o princípio aberto/fechado (<a href="https://pt.wikipedia.org/wiki/Princ%C3%ADpio_do_aberto/fechado" rel="noopener noreferrer">Open/Closed</a>)</li>
<li>transforma a função num monstro difícil de testar</li>
</ul>


<h2>
  
  
  O que é o padrão Strategy
</h2>

<p>O <strong>Strategy</strong> é um padrão de projeto comportamental que resolve exatamente esse problema.</p>

<p>A ideia central é simples:</p>

<blockquote>
<p><strong>Encapsular comportamentos intercambiáveis atrás de uma interface comum</strong></p>
</blockquote>

<p>Ou seja:</p>

<ul>
<li>você define <em>o que</em> precisa ser feito (interface)</li>
<li>cria várias formas de fazer isso (strategies)</li>
<li>escolhe qual usar em tempo de execução</li>
</ul>

<p>Sem <code>if/else</code> espalhado.<br>
Sem acoplamento desnecessário.</p>


<h2>
  
  
  Estrutura do Strategy
</h2>

<p>Conceitualmente, temos três peças:</p>

<ol>
<li>
<strong>Strategy (interface)</strong> – define o contrato</li>
<li>
<strong>Concrete Strategies</strong> – implementações do comportamento</li>
<li>
<strong>Context</strong> – quem usa a strategy</li>
</ol>

<p>Visualmente:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Context -&gt; Strategy
              ↑
   -------------------------
   |           |           |
StrategyA  StrategyB  StrategyC
</code></pre>

</div>






<h2>
  
  
  Exemplo prático
</h2>

<p>Vamos imaginar um cenário simples: <br>
cálculo de taxa.</p>
<h3>
  
  
  Interface
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">TaxStrategy</span> <span class="k">interface</span> <span class="p">{</span>
    <span class="n">Calculate</span><span class="p">(</span><span class="n">value</span> <span class="kt">float64</span><span class="p">)</span> <span class="kt">float64</span>
<span class="p">}</span>
</code></pre>

</div>

<h3>
  
  
  Implementações
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">BrazilTax</span> <span class="k">struct</span><span class="p">{}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">b</span> <span class="o">*</span><span class="n">BrazilTax</span><span class="p">)</span> <span class="n">Calculate</span><span class="p">(</span><span class="n">value</span> <span class="kt">float64</span><span class="p">)</span> <span class="kt">float64</span> <span class="p">{</span>
    <span class="k">return</span> <span class="n">value</span> <span class="o">*</span> <span class="m">0.15</span>
<span class="p">}</span>
</code></pre>

</div>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">USATax</span> <span class="k">struct</span><span class="p">{}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">u</span> <span class="o">*</span><span class="n">USATax</span><span class="p">)</span> <span class="n">Calculate</span><span class="p">(</span><span class="n">value</span> <span class="kt">float64</span><span class="p">)</span> <span class="kt">float64</span> <span class="p">{</span>
    <span class="k">return</span> <span class="n">value</span> <span class="o">*</span> <span class="m">0.08</span>
<span class="p">}</span>
</code></pre>

</div>

<h3>
  
  
  Context
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">Order</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Tax</span> <span class="n">TaxStrategy</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">o</span> <span class="o">*</span><span class="n">Order</span><span class="p">)</span> <span class="n">FinalValue</span><span class="p">(</span><span class="n">value</span> <span class="kt">float64</span><span class="p">)</span> <span class="kt">float64</span> <span class="p">{</span>
    <span class="k">return</span> <span class="n">value</span> <span class="o">+</span> <span class="n">o</span><span class="o">.</span><span class="n">Tax</span><span class="o">.</span><span class="n">Calculate</span><span class="p">(</span><span class="n">value</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>

<h3>
  
  
  Uso
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">order</span> <span class="o">:=</span> <span class="n">Order</span><span class="p">{</span><span class="n">Tax</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">BrazilTax</span><span class="p">{}}</span>
<span class="n">final</span> <span class="o">:=</span> <span class="n">order</span><span class="o">.</span><span class="n">FinalValue</span><span class="p">(</span><span class="m">100</span><span class="p">)</span>
</code></pre>

</div>


<p>Trocar a regra não exige mudar o <code>Order</code>. <br>
Só trocar a strategy.</p>


<h2>
  
  
  Registry de strategies
</h2>

<p>Em sistemas maiores, é comum usar um <strong>registry</strong> para centralizar as strategies disponíveis:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">var</span> <span class="n">registry</span> <span class="o">=</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="n">TaxStrategy</span><span class="p">{</span>
    <span class="s">"br"</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">BrazilTax</span><span class="p">{},</span>
    <span class="s">"us"</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">USATax</span><span class="p">{},</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Isso facilita:</p>

<ul>
<li>extensão do sistema</li>
<li>leitura do código</li>
<li>configuração por ambiente ou feature</li>
</ul>




<h2>
  
  
  Quando usar Strategy
</h2>

<p>Use Strategy quando:</p>

<p>✅ Você tem variações de comportamento<br>
✅ Essas variações crescem com o tempo<br>
✅ Você quer eliminar <code>if/else</code> baseados em tipo, regra ou contexto<br>
✅ Testabilidade é importante</p>


<h2>
  
  
  Quando NÃO usar
</h2>

<p>Nem tudo precisa de Strategy 👀</p>

<p>Evite quando:</p>

<p>❌ Existe só uma implementação e nenhuma chance real de variação<br>
❌ A abstração adiciona mais complexidade do que resolve<br>
❌ O comportamento nunca muda</p>

<p>Padrão de projeto não é troféu — é ferramenta.</p>


<h2>
  
  
  Erro comum ao usar Strategy
</h2>

<p>Um erro clássico é deixar a strategy <strong>conhecer o contexto errado</strong>, como ambiente:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">if</span> <span class="n">env</span> <span class="o">==</span> <span class="s">"qa"</span> <span class="p">{</span>
   <span class="c">// lógica especial</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Se isso começa a aparecer, é sinal de que:</p>

<ul>
<li>você precisa de strategies diferentes</li>
<li>ou de comportamentos injetáveis</li>
</ul>

<p>Strategy boa é <strong>pura</strong> e <strong>focada em uma única responsabilidade</strong>.</p>




<h2>
  
  
  Conclusão
</h2>

<p>O padrão Strategy é um dos padrões mais úteis no dia a dia, especialmente em sistemas que:</p>

<ul>
<li>evoluem rápido</li>
<li>têm regras de negócio mutáveis</li>
<li>precisam ser fáceis de testar e manter</li>
</ul>

<p>Se você sente que seu código está virando uma árvore de <code>if/else</code>, talvez não seja falta de esforço</p>

<p>Talvez só esteja faltando uma boa Estratégia 😉</p>

