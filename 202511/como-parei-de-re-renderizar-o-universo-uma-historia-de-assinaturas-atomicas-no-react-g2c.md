---
Title: Como parei de re-renderizar o universo: Uma história de assinaturas atômicas no React
Description: 
Author: quojs
Date: 2025-11-05T21:30:46.000Z
Robots: noindex,nofollow
Template: index
---
<p>Como construí a animação do logo do Quo.js com um Engine fora do React, assinaturas atômicas dentro do React, e quase zero boilerplate.</p>

<p><strong>TL;DR:</strong> Transformei um PNG estático do logo do Quo.js em centenas de círculos SVG animados que se montam, se dispersam ao redor do mouse, e deslizam de volta para casa — tudo em React 19 + TypeScript usando <a href="https://www.npmjs.com/package/@quojs/core" rel="noopener noreferrer">@quojs/core</a> e <a href="https://www.npmjs.com/package/@quojs/react" rel="noopener noreferrer">@quojs/react</a>. O truque é rodar um pequeno Engine completamente fora do React, transmitir atualizações de estado em lotes para uma store do Quo.js, e então renderizar com assinaturas atômicas no React para que a UI só re-renderize quando as coordenadas de um círculo específico realmente mudem.</p>

<p><strong>Tech:</strong> React 19, TypeScript, Vite, SVG</p>

<p><strong>State:</strong> <a href="https://github.com/quojs/quojs/blob/main/packages/core/README.pt.md" rel="noopener noreferrer">@quojs/core</a> e <a href="https://github.com/quojs/quojs/blob/main/packages/react/README.pt.md" rel="noopener noreferrer">@quojs/react</a></p>

<p><strong>O que NÃO é usado:</strong> three.js, WebGL, WebGPU. Isso é puro DOM/SVG + <code>requestAnimationFrame</code>.</p>

<h2>
  
  
  O que você verá (GIFs)
</h2>

<p>Se você visitar o site do Quo.js, verá isto:</p>

<ul>
<li><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnptu4qnbpg8vox9zt3as.gif" alt="Animação de introdução do logo do Quo.js — círculos se montam no logo do Quo.js" width="640" height="320"></li>
</ul>

<p><em>Intro: partículas convergem para a marca do logo "Quo.js".</em></p>

<p>E se você brincar com o cursor:</p>

<ul>
<li><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F21hybe2g6v51j8o7yva4.gif" alt="Interatividade do logo do Quo.js — círculos evitam o cursor e relaxam de volta" width="640" height="320"></li>
</ul>

<p><em>Interatividade: círculos orbitam para longe do cursor, então relaxam de volta para "casa".</em></p>

<p>Dev.to pode comprimir os arquivos GIF, se você vir animações lentas, considere baixar os arquivos GIF (<a href="https://quojs.dev/assets/devto/quojs-intro.gif" rel="noopener noreferrer">intro</a> e <a href="https://quojs.dev/assets/devto/quojs-intro.gif" rel="noopener noreferrer">interação</a>) ou visitar <a href="https://quojs.dev/?lang=pt" rel="noopener noreferrer">Quo.js site</a> para 60fps suaves.</p>

<h2>
  
  
  A ideia em um diagrama
</h2>

<p>Mantemos o React leve empurrando todo o trabalho de animação para um nano-Engine simples e plano de TypeScript. O Engine possui o loop de frames, quadtree e matemática de movimento. Ele envia atualizações em lotes para uma Store do Quo.js. O React consome apenas o pequeno pedaço que precisa por círculo via seletores atômicos.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmmszup9sfi48ufrsf609.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmmszup9sfi48ufrsf609.png" alt="Mermaid" width="800" height="494"></a>  </p>

<ul>
<li>
<strong>Engine (fora do React):</strong> loop requestAnimationFrame, clamp de dt, suavização de FPS, quadtree, física de círculos.</li>
<li>
<strong>Store Quo.js:</strong> reducers puramente funcionais, efeitos para pequenos temporizadores e eventos de ciclo de vida.</li>
<li>
<strong>React:</strong> apenas renderiza nós de círculo via <code>useSliceProp('logo', 'd.circle_0')</code> etc.</li>
</ul>

<h2>
  
  
  Por que esse design? (Performance e simplicidade)
</h2>

<ul>
<li>React fica ocioso a menos que seja necessário. Cada componente <code>&lt;Circle&gt;</code> se inscreve em seu próprio caminho x,y. Sem re-renders globais. Sem prop drilling.</li>
<li>Engine é agnóstico de framework. Ele não sabe sobre React. Apenas despacha ações (batchUpdate) para a store.</li>
<li>Menos boilerplate que Redux/RTK. Quo.js usa canais + eventos, efeitos sem thunks/sagas, e seletores atômicos para se inscrever em caminhos exatos com notação de pontos.</li>
<li>Atualizações em lotes por frame + um reducer compacto mantêm as escritas de estado mínimas.</li>
<li>rAF + clamp de dt + buffer circular de FPS dão movimento suave sem engasgos.</li>
</ul>

<h2>
  
  
  Passo 1 – Modelar o estado e eventos de canal
</h2>

<p>Mantemos o estado <code>logo</code> para a animação.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">type</span> <span class="nx">Circle</span> <span class="o">=</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">r</span><span class="p">?:</span> <span class="kr">number</span> <span class="p">};</span>
<span class="k">export</span> <span class="kd">type</span> <span class="nx">GroupedCircle</span> <span class="o">=</span> <span class="p">{</span> <span class="na">group</span><span class="p">:</span> <span class="dl">"</span><span class="s2">d</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">u</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">x</span><span class="dl">"</span> <span class="p">}</span> <span class="o">&amp;</span> <span class="nx">Circle</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">LogoState</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">enabled</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">;</span>
  <span class="nl">fps</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">itemCount</span><span class="p">:</span> <span class="p">{</span> <span class="na">d</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">u</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">x</span><span class="p">:</span> <span class="kr">number</span> <span class="p">};</span>
  <span class="nl">size</span><span class="p">:</span> <span class="p">{</span> <span class="na">height</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">width</span><span class="p">:</span> <span class="kr">number</span> <span class="p">};</span>
  <span class="nl">d</span><span class="p">:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">Circle</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nl">u</span><span class="p">:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">Circle</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nl">x</span><span class="p">:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">Circle</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nl">intro</span><span class="p">:</span> <span class="p">{</span> <span class="na">remaining</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">total</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">done</span><span class="p">:</span> <span class="nx">boolean</span> <span class="p">};</span>
<span class="p">};</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">LogoAM</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">start</span><span class="p">:</span> <span class="p">{};</span>
  <span class="nl">stop</span><span class="p">:</span> <span class="p">{};</span>
  <span class="nl">fps</span><span class="p">:</span> <span class="p">{</span> <span class="na">fps</span><span class="p">:</span> <span class="kr">number</span> <span class="p">};</span>
  <span class="nl">size</span><span class="p">:</span> <span class="p">{</span> <span class="na">height</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">width</span><span class="p">:</span> <span class="kr">number</span> <span class="p">};</span>
  <span class="nl">count</span><span class="p">:</span> <span class="p">{</span> <span class="na">d</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">u</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">x</span><span class="p">:</span> <span class="kr">number</span> <span class="p">};</span>
  <span class="nl">update</span><span class="p">:</span> <span class="nx">GroupedCircle</span><span class="p">;</span>
  <span class="nl">batchUpdate</span><span class="p">:</span> <span class="p">{</span> <span class="na">changes</span><span class="p">:</span> <span class="nx">GroupedCircle</span><span class="p">[]</span> <span class="p">};</span>
  <span class="nl">introProgress</span><span class="p">:</span> <span class="p">{</span> <span class="na">remaining</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">total</span><span class="p">:</span> <span class="kr">number</span> <span class="p">};</span>
  <span class="nl">introComplete</span><span class="p">:</span> <span class="p">{};</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Padrão de dispatch:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">store</span><span class="p">.</span><span class="nf">dispatch</span><span class="p">(</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">batchUpdate</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">changes</span><span class="p">:</span> <span class="p">[...]</span> <span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Passo 2 – O reducer: imutável, rápido
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">ActionPair</span><span class="p">,</span> <span class="nx">ReducerSpec</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@quojs/core</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">AppAM</span><span class="p">,</span> <span class="nx">LogoAM</span><span class="p">,</span> <span class="nx">LogoState</span><span class="p">,</span> <span class="nx">Circle</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">../types</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">LOGO_INITIAL_STATE</span><span class="p">:</span> <span class="nx">LogoState</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">d</span><span class="p">:</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span><span class="kc">null</span><span class="p">),</span>
  <span class="na">u</span><span class="p">:</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span><span class="kc">null</span><span class="p">),</span>
  <span class="na">x</span><span class="p">:</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span><span class="kc">null</span><span class="p">),</span>
  <span class="na">fps</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
  <span class="na">itemCount</span><span class="p">:</span> <span class="p">{</span> <span class="na">d</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="na">u</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="na">x</span><span class="p">:</span> <span class="mi">0</span> <span class="p">},</span>
  <span class="na">size</span><span class="p">:</span> <span class="p">{</span> <span class="na">height</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="na">width</span><span class="p">:</span> <span class="mi">0</span> <span class="p">},</span>
  <span class="na">intro</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">remaining</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="na">total</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="na">done</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">LOGO_ACTIONS</span> <span class="o">=</span> <span class="p">[</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">update</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">stop</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">fps</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">size</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">count</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">batchUpdate</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">introProgress</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">introComplete</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">[</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">start</span><span class="dl">"</span><span class="p">],</span>
<span class="p">]</span> <span class="k">as</span> <span class="kd">const</span> <span class="nx">satisfies</span> <span class="k">readonly</span> <span class="nx">ActionPair</span><span class="o">&lt;</span><span class="nx">AppAM</span><span class="o">&gt;</span><span class="p">[];</span>

<span class="c1">// açúcar para inferência de tipos</span>
<span class="kd">type</span> <span class="nx">GroupKey</span> <span class="o">=</span> <span class="kr">keyof</span> <span class="nb">Pick</span><span class="o">&lt;</span><span class="nx">LogoState</span><span class="p">,</span> <span class="dl">"</span><span class="s2">d</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">u</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">x</span><span class="dl">"</span><span class="o">&gt;</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">upsertItem</span><span class="p">(</span><span class="nx">state</span><span class="p">:</span> <span class="nx">LogoState</span><span class="p">,</span> <span class="nx">group</span><span class="p">:</span> <span class="nx">GroupKey</span><span class="p">,</span> <span class="nx">next</span><span class="p">:</span> <span class="nx">Circle</span><span class="p">):</span> <span class="nx">LogoState</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">groupMap</span> <span class="o">=</span> <span class="nx">state</span><span class="p">[</span><span class="nx">group</span><span class="p">];</span>
  <span class="kd">const</span> <span class="nx">prev</span> <span class="o">=</span> <span class="nx">groupMap</span><span class="p">[</span><span class="nx">next</span><span class="p">.</span><span class="nx">id</span><span class="p">];</span>

  <span class="c1">// inserir</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">prev</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">nextGroup</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">groupMap</span><span class="p">,</span> <span class="p">[</span><span class="nx">next</span><span class="p">.</span><span class="nx">id</span><span class="p">]:</span> <span class="nx">next</span> <span class="p">};</span>
    <span class="k">return</span> <span class="p">{</span> <span class="p">...</span><span class="nx">state</span><span class="p">,</span> <span class="p">[</span><span class="nx">group</span><span class="p">]:</span> <span class="nx">nextGroup</span> <span class="p">};</span>
  <span class="p">}</span>

  <span class="c1">// atualizar apenas se algo realmente mudou</span>
  <span class="k">if </span><span class="p">(</span>
    <span class="nx">prev</span><span class="p">.</span><span class="nx">x</span> <span class="o">===</span> <span class="nx">next</span><span class="p">.</span><span class="nx">x</span> <span class="o">&amp;&amp;</span>
    <span class="nx">prev</span><span class="p">.</span><span class="nx">y</span> <span class="o">===</span> <span class="nx">next</span><span class="p">.</span><span class="nx">y</span>
  <span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">state</span><span class="p">;</span> <span class="c1">// no-op</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">nextGroup</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">groupMap</span><span class="p">,</span> <span class="p">[</span><span class="nx">next</span><span class="p">.</span><span class="nx">id</span><span class="p">]:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">prev</span><span class="p">,</span> <span class="p">...</span><span class="nx">next</span> <span class="p">}</span> <span class="p">};</span>
  <span class="k">return</span> <span class="p">{</span> <span class="p">...</span><span class="nx">state</span><span class="p">,</span> <span class="p">[</span><span class="nx">group</span><span class="p">]:</span> <span class="nx">nextGroup</span> <span class="p">};</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">logoReducer</span><span class="p">:</span> <span class="nx">ReducerSpec</span><span class="o">&lt;</span><span class="nx">LogoState</span><span class="p">,</span> <span class="nx">AppAM</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">actions</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">...</span><span class="nx">LOGO_ACTIONS</span>
  <span class="p">],</span>
  <span class="na">state</span><span class="p">:</span> <span class="nx">LOGO_INITIAL_STATE</span><span class="p">,</span>
  <span class="na">reducer</span><span class="p">:</span> <span class="p">(</span><span class="nx">state</span><span class="p">,</span> <span class="nx">action</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">action</span><span class="p">.</span><span class="nx">channel</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">state</span><span class="p">.</span><span class="nx">enabled</span> <span class="o">&amp;&amp;</span> <span class="nx">action</span><span class="p">.</span><span class="nx">event</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">start</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>

    <span class="k">switch </span><span class="p">(</span><span class="nx">action</span><span class="p">.</span><span class="nx">event</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">case</span> <span class="dl">"</span><span class="s2">update</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="p">{</span> <span class="nx">group</span><span class="p">,</span> <span class="nx">id</span><span class="p">,</span> <span class="nx">x</span><span class="p">,</span> <span class="nx">y</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">action</span><span class="p">.</span><span class="nx">payload</span><span class="p">;</span>
        <span class="kd">const</span> <span class="na">next</span><span class="p">:</span> <span class="nx">Circle</span> <span class="o">=</span> <span class="p">{</span> <span class="nx">id</span><span class="p">,</span> <span class="nx">x</span><span class="p">,</span> <span class="nx">y</span> <span class="p">};</span>

        <span class="k">return</span> <span class="nf">upsertItem</span><span class="p">(</span><span class="nx">state</span><span class="p">,</span> <span class="nx">group</span> <span class="k">as</span> <span class="nx">GroupKey</span><span class="p">,</span> <span class="nx">next</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">start</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">state</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>

        <span class="k">return</span> <span class="p">{</span>
          <span class="p">...</span><span class="nx">state</span><span class="p">,</span>
          <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
        <span class="p">};</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">stop</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">state</span><span class="p">.</span><span class="nx">enabled</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>

        <span class="k">return</span> <span class="p">{</span>
          <span class="p">...</span><span class="nx">state</span><span class="p">,</span>
          <span class="na">enabled</span><span class="p">:</span> <span class="kc">false</span>
        <span class="p">};</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">fps</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="p">{</span> <span class="nx">fps</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">action</span><span class="p">.</span><span class="nx">payload</span> <span class="k">as</span> <span class="nx">LogoAM</span><span class="p">[</span><span class="dl">"</span><span class="s2">fps</span><span class="dl">"</span><span class="p">];</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">state</span><span class="p">.</span><span class="nx">fps</span> <span class="o">===</span> <span class="nx">fps</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>

        <span class="k">return</span> <span class="p">{</span>
          <span class="p">...</span><span class="nx">state</span><span class="p">,</span>
          <span class="nx">fps</span><span class="p">,</span>
        <span class="p">};</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">count</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">next</span> <span class="o">=</span> <span class="nx">action</span><span class="p">.</span><span class="nx">payload</span> <span class="k">as</span> <span class="nx">LogoAM</span><span class="p">[</span><span class="dl">"</span><span class="s2">count</span><span class="dl">"</span><span class="p">];</span>
        <span class="kd">const</span> <span class="nx">prev</span> <span class="o">=</span> <span class="nx">state</span><span class="p">.</span><span class="nx">itemCount</span><span class="p">;</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">prev</span><span class="p">.</span><span class="nx">d</span> <span class="o">===</span> <span class="nx">next</span><span class="p">.</span><span class="nx">d</span> <span class="o">&amp;&amp;</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">u</span> <span class="o">===</span> <span class="nx">next</span><span class="p">.</span><span class="nx">u</span> <span class="o">&amp;&amp;</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">x</span> <span class="o">===</span> <span class="nx">next</span><span class="p">.</span><span class="nx">x</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>

        <span class="k">return</span> <span class="p">{</span> <span class="p">...</span><span class="nx">state</span><span class="p">,</span> <span class="na">itemCount</span><span class="p">:</span> <span class="nx">next</span> <span class="p">};</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">size</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="p">{</span> <span class="nx">height</span><span class="p">,</span> <span class="nx">width</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">action</span><span class="p">.</span><span class="nx">payload</span> <span class="k">as</span> <span class="nx">LogoAM</span><span class="p">[</span><span class="dl">"</span><span class="s2">size</span><span class="dl">"</span><span class="p">];</span>
        <span class="kd">const</span> <span class="nx">prev</span> <span class="o">=</span> <span class="nx">state</span><span class="p">.</span><span class="nx">size</span><span class="p">;</span>

        <span class="k">if </span><span class="p">(</span><span class="nx">prev</span><span class="p">.</span><span class="nx">height</span> <span class="o">===</span> <span class="nx">height</span> <span class="o">&amp;&amp;</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">width</span> <span class="o">===</span> <span class="nx">width</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>

        <span class="k">return</span> <span class="p">{</span>
          <span class="p">...</span><span class="nx">state</span><span class="p">,</span> <span class="na">size</span><span class="p">:</span> <span class="p">{</span>
            <span class="nx">height</span><span class="p">,</span>
            <span class="nx">width</span><span class="p">,</span>
          <span class="p">}</span>
        <span class="p">};</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">batchUpdate</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">action</span><span class="p">.</span><span class="nx">payload</span><span class="p">.</span><span class="nx">changes</span><span class="p">.</span><span class="nx">length</span><span class="p">)</span> <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>
        <span class="kd">const</span> <span class="p">{</span> <span class="nx">changes</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">action</span><span class="p">.</span><span class="nx">payload</span><span class="p">;</span>

        <span class="kd">let</span> <span class="nx">wroteAny</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
        <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">c</span> <span class="k">of</span> <span class="nx">changes</span><span class="p">)</span> <span class="p">{</span>
          <span class="kd">let</span> <span class="nx">prev</span> <span class="o">=</span> <span class="nx">state</span><span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">group</span><span class="p">][</span><span class="nx">c</span><span class="p">.</span><span class="nx">id</span><span class="p">];</span>

          <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">prev</span><span class="p">)</span> <span class="p">{</span>
            <span class="nx">prev</span> <span class="o">=</span> <span class="p">{</span>
              <span class="p">...</span><span class="nx">state</span><span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">group</span><span class="p">][</span><span class="nx">c</span><span class="p">.</span><span class="nx">id</span><span class="p">],</span>
              <span class="p">...</span><span class="nx">c</span><span class="p">,</span>
            <span class="p">};</span>

            <span class="nx">state</span> <span class="o">=</span> <span class="p">{</span>
              <span class="p">...</span><span class="nx">state</span><span class="p">,</span>
              <span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">group</span><span class="p">]:</span> <span class="p">{</span>
                <span class="p">...</span><span class="nx">state</span><span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">group</span><span class="p">],</span>
                <span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">id</span><span class="p">]:</span> <span class="nx">prev</span><span class="p">,</span>
              <span class="p">}</span>
            <span class="p">};</span>

            <span class="nx">wroteAny</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
            <span class="k">continue</span><span class="p">;</span>
          <span class="p">}</span>

          <span class="kd">const</span> <span class="nx">nx</span> <span class="o">=</span> <span class="nx">c</span><span class="p">.</span><span class="nx">x</span> <span class="o">??</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">x</span><span class="p">;</span>
          <span class="kd">const</span> <span class="nx">ny</span> <span class="o">=</span> <span class="nx">c</span><span class="p">.</span><span class="nx">y</span> <span class="o">??</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">y</span><span class="p">;</span>

          <span class="k">if </span><span class="p">(</span><span class="nx">nx</span> <span class="o">!==</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">x</span> <span class="o">||</span> <span class="nx">ny</span> <span class="o">!==</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">y</span><span class="p">)</span> <span class="p">{</span>
            <span class="nx">state</span> <span class="o">=</span> <span class="p">{</span>
              <span class="p">...</span><span class="nx">state</span><span class="p">,</span>
              <span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">group</span><span class="p">]:</span> <span class="p">{</span>
                <span class="p">...</span><span class="nx">state</span><span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">group</span><span class="p">],</span>
                <span class="p">[</span><span class="nx">c</span><span class="p">.</span><span class="nx">id</span><span class="p">]:</span> <span class="p">{</span> <span class="p">...</span><span class="nx">prev</span><span class="p">,</span> <span class="na">x</span><span class="p">:</span> <span class="nx">nx</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="nx">ny</span> <span class="p">},</span>
              <span class="p">}</span>
            <span class="p">};</span>

            <span class="nx">wroteAny</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
          <span class="p">}</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="nx">wroteAny</span> <span class="p">?</span> <span class="p">{</span> <span class="p">...</span><span class="nx">state</span> <span class="p">}</span> <span class="p">:</span> <span class="nx">state</span><span class="p">;</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">introProgress</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="p">{</span> <span class="nx">remaining</span><span class="p">,</span> <span class="nx">total</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">action</span><span class="p">.</span><span class="nx">payload</span><span class="p">;</span>

        <span class="k">return</span> <span class="p">{</span>
          <span class="p">...</span><span class="nx">state</span><span class="p">,</span>
          <span class="na">intro</span><span class="p">:</span> <span class="p">{</span>
            <span class="p">...</span><span class="nx">state</span><span class="p">.</span><span class="nx">intro</span><span class="p">,</span>
            <span class="nx">remaining</span><span class="p">,</span>
            <span class="nx">total</span><span class="p">,</span>
          <span class="p">}</span>
        <span class="p">};</span>
      <span class="p">}</span>

      <span class="k">case</span> <span class="dl">"</span><span class="s2">introComplete</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="k">return</span> <span class="p">{</span>
          <span class="p">...</span><span class="nx">state</span><span class="p">,</span>
          <span class="na">intro</span><span class="p">:</span> <span class="p">{</span>
            <span class="p">...(</span><span class="nx">state</span><span class="p">.</span><span class="nx">intro</span> <span class="o">??</span> <span class="p">{}),</span>
            <span class="na">remaining</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
            <span class="na">done</span><span class="p">:</span> <span class="kc">true</span>
          <span class="p">}</span>
        <span class="p">};</span>
      <span class="p">}</span>

      <span class="nl">default</span><span class="p">:</span>
        <span class="k">return</span> <span class="nx">state</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<ul>
<li>Evita escritas se x,y não mudaram.</li>
<li>Atualiza muitas mudanças em lote por frame.</li>
<li>Apenas toca as chaves modificadas.</li>
</ul>

<h2>
  
  
  Passo 3 – Criar a store
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">store</span> <span class="o">=</span> <span class="nf">createStore</span><span class="p">({</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Quo.js</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">reducer</span><span class="p">:</span> <span class="p">{</span> <span class="na">logo</span><span class="p">:</span> <span class="nx">logoReducer</span> <span class="p">},</span>
  <span class="na">effects</span><span class="p">:</span> <span class="p">[],</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Passo 4 – O Engine (fora do React)
</h2>

<ul>
<li>Roda loop rAF.</li>
<li>Calcula dt e FPS.</li>
<li>Despacha logo/batchUpdate.</li>
<li>Se inscreve em efeitos da store para start/stop.</li>
<li>Nunca importa React.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">if </span><span class="p">(</span><span class="nx">dt</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="k">this</span><span class="p">.</span><span class="nx">simulation</span><span class="p">.</span><span class="nf">loop</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_dt</span><span class="p">,</span> <span class="nx">now</span><span class="p">);</span>
<span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">_running</span> <span class="o">&amp;&amp;</span> <span class="nx">gen</span> <span class="o">===</span> <span class="k">this</span><span class="p">.</span><span class="nx">_rafGen</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">_handle</span> <span class="o">=</span> <span class="nf">requestAnimationFrame</span><span class="p">((</span><span class="nx">t</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nf">_tick</span><span class="p">(</span><span class="nx">t</span><span class="p">,</span> <span class="nx">gen</span><span class="p">));</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Cada Circle emite atualizações <code>{ group, id, x, y }</code> quando se move.</p>

<h2>
  
  
  Passo 5 – Conectar Engine e Store no React
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">engine</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Engine</span><span class="p">({</span> <span class="na">targetFPS</span><span class="p">:</span> <span class="mi">60</span><span class="p">,</span> <span class="na">autoStart</span><span class="p">:</span> <span class="kc">false</span> <span class="p">},</span> <span class="nx">store</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">setup</span> <span class="o">=</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">image</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">loadImagePixels</span><span class="p">(</span><span class="nx">quoLogo</span><span class="p">);</span>
    <span class="kd">const</span> <span class="p">{</span> <span class="nx">specs</span><span class="p">,</span> <span class="nx">width</span><span class="p">,</span> <span class="nx">height</span><span class="p">,</span> <span class="nx">groupCounts</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">extractCircleSpecsFromImage</span><span class="p">(</span>
      <span class="nx">image</span><span class="p">,</span>
      <span class="p">{</span> <span class="na">spacing</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span> <span class="na">initialR</span><span class="p">:</span> <span class="mf">0.5</span><span class="p">,</span> <span class="na">maxCircles</span><span class="p">:</span> <span class="mi">1500</span> <span class="p">}</span>
    <span class="p">);</span>

    <span class="nx">store</span><span class="p">.</span><span class="nf">dispatch</span><span class="p">(</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">size</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="nx">height</span><span class="p">,</span> <span class="nx">width</span> <span class="p">});</span>
    <span class="nx">store</span><span class="p">.</span><span class="nf">dispatch</span><span class="p">(</span><span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">count</span><span class="dl">"</span><span class="p">,</span> <span class="nx">groupCounts</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">sim</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Simulation</span><span class="p">(</span><span class="nx">engine</span><span class="p">,</span> <span class="p">{</span> <span class="na">items</span><span class="p">:</span> <span class="nx">specs</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Quo Packing</span><span class="dl">"</span> <span class="p">});</span>

    <span class="nx">engine</span><span class="p">.</span><span class="nf">attach</span><span class="p">(</span><span class="nx">sim</span><span class="p">);</span>
    <span class="nx">engine</span><span class="p">.</span><span class="nf">init</span><span class="p">();</span>
    <span class="nx">engine</span><span class="p">.</span><span class="nf">start</span><span class="p">();</span>
  <span class="p">};</span>

  <span class="nf">setup</span><span class="p">();</span>
  <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">engine</span><span class="p">.</span><span class="nf">teardown</span><span class="p">();</span>
<span class="p">},</span> <span class="p">[</span><span class="nx">store</span><span class="p">]);</span>
</code></pre>

</div>



<p><strong>Espera aí, manu—como transformamos um PNG em um enxame de círculos?</strong></p>

<p>Por baixo dos panos, <code>extractCircleSpecsFromImage</code> carrega o PNG do logo em um canvas fora da tela, amostra o alpha de pixels em uma grade configurável (padrão: espaçamento de 8px), e emite uma especificação de círculo onde alpha &gt; 50.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">threshold</span> <span class="o">=</span> <span class="mf">0.2</span><span class="p">;</span> <span class="c1">// 0–1, ajuste para densidade</span>
<span class="kd">const</span> <span class="nx">spacing</span> <span class="o">=</span> <span class="mi">8</span><span class="p">;</span>

<span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">y</span> <span class="o">&lt;</span> <span class="nx">h</span><span class="p">;</span> <span class="nx">y</span> <span class="o">+=</span> <span class="nx">spacing</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">x</span> <span class="o">&lt;</span> <span class="nx">w</span><span class="p">;</span> <span class="nx">x</span> <span class="o">+=</span> <span class="nx">spacing</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">alpha</span> <span class="o">=</span> <span class="nx">ctx</span><span class="p">.</span><span class="nf">getImageData</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">).</span><span class="nx">data</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span> <span class="o">/</span> <span class="mi">255</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">alpha</span> <span class="o">&gt;</span> <span class="nx">threshold</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">circles</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span> <span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">,</span> <span class="na">r</span><span class="p">:</span> <span class="nx">spacing</span> <span class="o">*</span> <span class="mf">0.45</span> <span class="p">});</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Quer um enxame mais denso? Diminua o espaçamento. Quer iluminação ambiente? Mapeie alpha → raio. É apenas dados—hackeie à vontade.</p>

<h2>
  
  
  Passo 6 – Cola do React com @quojs/react
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="p">{</span> <span class="nx">useStore</span><span class="p">,</span> <span class="nx">useDispatch</span><span class="p">,</span> <span class="nx">useSliceProp</span> <span class="p">}</span> <span class="o">=</span>
  <span class="nf">createQuoHooks</span><span class="p">(</span><span class="nx">AppStoreContext</span><span class="p">);</span>
</code></pre>

</div>



<p>Agora cada <code>&lt;Circle&gt;</code> se inscreve atomicamente:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">Circle</span> <span class="o">=</span> <span class="p">({</span> <span class="nx">id</span><span class="p">,</span> <span class="nx">group</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">path</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">group</span><span class="p">}</span><span class="s2">.</span><span class="p">${</span><span class="nx">id</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>

  <span class="kd">const</span> <span class="p">{</span> <span class="nx">x</span><span class="p">,</span> <span class="nx">y</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useSliceProp</span><span class="p">({</span>
    <span class="na">reducer</span><span class="p">:</span> <span class="dl">"</span><span class="s2">logo</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">property</span><span class="p">:</span> <span class="nx">path</span><span class="p">,</span>
  <span class="p">})</span> <span class="o">??</span> <span class="p">{</span> <span class="na">x</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="mi">0</span> <span class="p">};</span>

  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">circle</span> <span class="nx">className</span><span class="o">=</span><span class="p">{</span><span class="s2">`group-</span><span class="p">${</span><span class="nx">group</span><span class="p">}</span><span class="s2">`</span><span class="p">}</span> <span class="nx">cx</span><span class="o">=</span><span class="p">{</span><span class="s2">`</span><span class="p">${</span><span class="nx">x</span><span class="p">}</span><span class="s2">px`</span><span class="p">}</span> <span class="nx">cy</span><span class="o">=</span><span class="p">{</span><span class="s2">`</span><span class="p">${</span><span class="nx">y</span><span class="p">}</span><span class="s2">px`</span><span class="p">}</span> <span class="sr">/&gt;</span><span class="err">;
</span><span class="p">};</span>
</code></pre>

</div>



<p>Apenas esse círculo re-renderiza quando suas próprias coordenadas mudam.</p>

<h2>
  
  
  Resumo do fluxo de dados
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzjcg9y9bdp3jr1luiweo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzjcg9y9bdp3jr1luiweo.png" alt="Mermaid" width="800" height="494"></a>  </p>

<h2>
  
  
  FAQ / Notas
</h2>

<ul>
<li>
<strong>Por que não RTK?</strong> Precisava do modelo de canal/evento, efeitos assíncronos, assinaturas atômicas.</li>
<li>
<strong>Por que não three.js/WebGL?</strong> Exagero para partículas 2D; SVG + Quo.js é leve.</li>
<li>
<strong>Conclusão da intro?</strong> A simulação rastreia círculos restantes → despacha introProgress → introComplete.</li>
<li>
<strong>FPS?</strong> Média via buffer circular, despachado esparsamente.</li>
</ul>

<h2>
  
  
  Por que despachos esparsos, não spam por frame?
</h2>

<p>Inundar o React com 60 atualizações de estado/seg é um big-bang de renderização. Em vez disso, o Engine usa um buffer circular para rastrear tempos de frame, calculando FPS apenas quando o buffer completa (~a cada 30 frames).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">frameCount</span> <span class="o">%</span> <span class="mi">30</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">fps</span> <span class="o">=</span> <span class="mi">30</span> <span class="o">/</span> <span class="p">(</span><span class="nx">deltaSum</span> <span class="o">/</span> <span class="mi">1000</span><span class="p">);</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">store</span><span class="p">.</span><span class="nf">dispatch</span><span class="p">(</span><span class="nf">setFps</span><span class="p">(</span><span class="nx">fps</span><span class="p">));</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Resultado? Uma atualização atômica a cada ~500ms, zero jank, e Redux Devtools permanecem sãos. —sim, você leu certo: Quo.js suporta Redux Devtools—. Telemetria de performance deve servir a animação—não matá-la de fome.</p>

<h2>
  
  
  Reproduzir localmente
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add @quojs/core@0.2.0 @quojs/react@0.2.0
</code></pre>

</div>



<ol>
<li>Criar store (ver Passo 3).</li>
<li>Envolver <code>&lt;AppStoreContext.Provider value={store}&gt;</code>.</li>
<li>Montar Engine (Passo 5).</li>
<li>Renderizar SVG Circles (Passo 6).</li>
</ol>

<p><strong>Docs para Quo.js e amigos:</strong></p>

<ul>
<li><a href="https://github.com/quojs/quojs/blob/main/packages/core/README.pt.md" rel="noopener noreferrer">Core</a></li>
<li><a href="https://github.com/quojs/quojs/blob/main/packages/react/README.pt.md" rel="noopener noreferrer">React</a></li>
</ul>

<h2>
  
  
  Padrões pequenos mas impactantes
</h2>

<ul>
<li>Escritas em lotes por frame.</li>
<li>Guarda de no-op em valores idênticos.</li>
<li>Assinaturas a caminhos exatos.</li>
<li>Guarda de tokens de geração rAF.</li>
<li>Despacho esparso de telemetria.</li>
</ul>

<h2>
  
  
  Experimente!
</h2>

<p>⭐ <a href="https://github.com/quojs/quojs" rel="noopener noreferrer">Dê uma estrela ao Quo.js no GitHub</a><br><br>
🧪 Experimente <a href="https://www.npmjs.com/package/@quojs/core" rel="noopener noreferrer">@quojs/core@0.2.0</a><br><br>
🧪 Experimente <a href="https://www.npmjs.com/package/@quojs/react" rel="noopener noreferrer">@quojs/react@0.2.0</a><br><br>
🐞 <a href="https://github.com/quojs/quojs/issues" rel="noopener noreferrer">Registre issues / ideias</a><br><br>
🧩 <a href="https://github.com/quojs/quojs/tree/main/examples/v0" rel="noopener noreferrer">Contribua exemplos</a></p>

<h2>
  
  
  Reflexões finais
</h2>

<p>Esta animação mostra como React pode ser um renderizador, não um loop de jogo. Lidando com mudanças de estado precisas e granulares, você obtém alto FPS, pouca agitação do React, e lógica mantível.</p>

<p><strong>Uma nota rápida sobre os nomes dos hooks para useSliceProp e useSliceProps:</strong> Estes provavelmente mudarão para algo mais significativo como <strong>useAtomicProp</strong> e <strong>useAtomicProps</strong> antes do próximo lançamento (em uma semana), então não os use ainda em produção. Quo.js ainda está em testes beta.</p>

<p>Tenha um lindo coding, <a href="https://x.com/manu_codes" rel="noopener noreferrer">manu</a>.</p>

<p><strong>Licença:</strong> <a href="https://github.com/quojs/quojs?tab=License-1-ov-file" rel="noopener noreferrer">MPL-2.0 — compartilhe com o mundo</a>.</p>

