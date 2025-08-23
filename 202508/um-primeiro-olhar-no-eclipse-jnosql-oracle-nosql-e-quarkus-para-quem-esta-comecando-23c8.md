---
Title: Um Primeiro Olhar no Eclipse JNoSQL: Oracle NoSQL e Quarkus para Quem Está Começando
Description: 
Author: Luis Fabrício De Llamas
Date: 2025-08-23T19:10:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>Fala, dev! Se você chegou até aqui, provavelmente está se perguntando: "Como eu faço pra trabalhar com NoSQL de forma simples no Java?" Então, hoje vou te mostrar uma combinação que vai te ajudar: <strong>Eclipse JNoSQL + Quarkus + Oracle NoSQL</strong>. E o melhor de tudo? Usando os heróis da Caverna do Dragão( Sim! Eu não falo só de Naruto não hehe)!</p>

<h2>
  
  
  A Magia por Trás do Eclipse JNoSQL
</h2>

<p>Antes de mais nada, vamos falar sobre o <strong>elefante na sala</strong>: por que usar JNoSQL quando eu posso chamar diretamente a API do banco?</p>

<p>A resposta é simples: <strong>padronização e flexibilidade</strong>. O Eclipse JNoSQL implementa as especificações Jakarta NoSQL e Jakarta Data, o que significa que você escreve código uma vez e pode rodar em diferentes bancos NoSQL. É tipo ter um tradutor universal para bancos de dados!</p>

<p>A primeira vez que ouvi falar do JNoSQL foi quando o <strong>Maximillian Arruda</strong> apresentou o tema para a comunidade Quarkus Club (nossa primeira palestra ao vivo!). Infelizmente não ficou gravada, mas o Max é um grande amigo e sei que teremos novas oportunidades.</p>

<h2>
  
  
  Por Que Oracle NoSQL É Especial?
</h2>

<p>Muita gente não sabe, mas a Oracle tem um banco NoSQL <strong>muito</strong> robusto. E se tem alguém que entende de bancos, é a Oracle!</p>

<ul>
<li>
<strong>Multi-model</strong>: Document + Key-Value na mesma base</li>
<li>
<strong>Transações ACID</strong>: Sim, você leu certo! NoSQL com consistência forte</li>
<li>
<strong>Integração Cloud</strong>: Nativo na Oracle Cloud Infrastructure</li>
<li>
<strong>Performance</strong>: Otimizado para workloads modernos</li>
</ul>

<h2>
  
  
  Bora pro código: Sistema de Aventureiros
</h2>

<p>Vamos criar um sistema para gerenciar nosso grupo de heróis da Caverna do Dragão! Por que esse exemplo? Porque tem personagens únicos, cada um com habilidades diferentes, porque sou velho... </p>

<p>Perfeito para mostrar a flexibilidade do NoSQL.</p>

<h3>
  
  
  Setup Inicial
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Crie o projeto</span>
quarkus create app com.caverna:heroes-management <span class="se">\</span>
    <span class="nt">--extension</span><span class="o">=</span><span class="s2">"quarkus-jnosql-oracle-nosql,quarkus-rest-jackson"</span>

<span class="nb">cd </span>heroes-management
</code></pre>

</div>



<h3>
  
  
  Configuração Inteligente
</h3>

<p>Aqui está um ponto importante que muitos artigos não mostram: configuração por ambiente.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="c"># application.properties
</span>
<span class="c"># Configuração base
</span><span class="py">jnosql.oracle.nosql.host</span><span class="p">=</span><span class="s">localhost:8080</span>
<span class="py">jnosql.document.database</span><span class="p">=</span><span class="s">realm_caverna</span>
<span class="py">jnosql.keyvalue.database</span><span class="p">=</span><span class="s">heroes_cache</span>

<span class="c"># Desenvolvimento local
</span><span class="err">%</span><span class="py">dev.jnosql.oracle.nosql.host</span><span class="p">=</span><span class="s">localhost:8080</span>
<span class="err">%</span><span class="py">dev.jnosql.document.database</span><span class="p">=</span><span class="s">caverna_dev</span>

<span class="c"># Produção (Oracle Cloud)
</span><span class="err">%</span><span class="py">prod.jnosql.oracle.nosql.host</span><span class="p">=</span><span class="s">${ORACLE_NOSQL_ENDPOINT}</span>
<span class="err">%</span><span class="py">prod.jnosql.oracle.nosql.region</span><span class="p">=</span><span class="s">${OCI_REGION}</span>
<span class="err">%</span><span class="py">prod.jnosql.oracle.nosql.compartment</span><span class="p">=</span><span class="s">${OCI_COMPARTMENT}</span>

<span class="c"># Logging para debug
</span><span class="err">quarkus.log.category."org.eclipse.jnosql"</span><span class="py">.level</span><span class="p">=</span><span class="s">INFO</span>
</code></pre>

</div>



<h3>
  
  
  Modelando Nossos Heróis
</h3>

<p>Aqui é onde a coisa fica interessante. Vamos modelar os personagens de forma que aproveite as vantagens do NoSQL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.caverna.model</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.Column</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.Entity</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.eclipse.jnosql.mapping.Id</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.time.LocalDateTime</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.List</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.Map</span><span class="o">;</span>

<span class="nd">@Entity</span><span class="o">(</span><span class="s">"herois"</span><span class="o">)</span>  <span class="c1">// Nome customizado da collection</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">Heroi</span> <span class="o">{</span>

    <span class="nd">@Id</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">id</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">nome</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">ClasseHeroi</span> <span class="n">classe</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">Integer</span> <span class="n">experiencia</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="n">habilidadesEspeciais</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">ArmaLendaria</span> <span class="n">armaLendaria</span><span class="o">;</span>

    <span class="c1">// Aqui está o legal: dados flexíveis!</span>
    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">Map</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">Object</span><span class="o">&gt;</span> <span class="n">atributosPersonalizados</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">LocalDateTime</span> <span class="n">ultimaAventura</span><span class="o">;</span>

    <span class="nd">@Column</span>
    <span class="kd">private</span> <span class="nc">Boolean</span> <span class="n">ativo</span> <span class="o">=</span> <span class="kc">true</span><span class="o">;</span>

    <span class="c1">// Construtor padrão obrigatório</span>
    <span class="kd">public</span> <span class="nf">Heroi</span><span class="o">()</span> <span class="o">{}</span>

    <span class="c1">// Construtor útil</span>
    <span class="kd">public</span> <span class="nf">Heroi</span><span class="o">(</span><span class="nc">String</span> <span class="n">nome</span><span class="o">,</span> <span class="nc">ClasseHeroi</span> <span class="n">classe</span><span class="o">,</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="n">habilidades</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">this</span><span class="o">.</span><span class="na">nome</span> <span class="o">=</span> <span class="n">nome</span><span class="o">;</span>
        <span class="k">this</span><span class="o">.</span><span class="na">classe</span> <span class="o">=</span> <span class="n">classe</span><span class="o">;</span>
        <span class="k">this</span><span class="o">.</span><span class="na">habilidadesEspeciais</span> <span class="o">=</span> <span class="n">habilidades</span><span class="o">;</span>
        <span class="k">this</span><span class="o">.</span><span class="na">experiencia</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
        <span class="k">this</span><span class="o">.</span><span class="na">ultimaAventura</span> <span class="o">=</span> <span class="nc">LocalDateTime</span><span class="o">.</span><span class="na">now</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="c1">// Getters e setters...</span>
    <span class="kd">public</span> <span class="nc">String</span> <span class="nf">getId</span><span class="o">()</span> <span class="o">{</span> <span class="k">return</span> <span class="n">id</span><span class="o">;</span> <span class="o">}</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">setId</span><span class="o">(</span><span class="nc">String</span> <span class="n">id</span><span class="o">)</span> <span class="o">{</span> <span class="k">this</span><span class="o">.</span><span class="na">id</span> <span class="o">=</span> <span class="n">id</span><span class="o">;</span> <span class="o">}</span>

    <span class="kd">public</span> <span class="nc">String</span> <span class="nf">getNome</span><span class="o">()</span> <span class="o">{</span> <span class="k">return</span> <span class="n">nome</span><span class="o">;</span> <span class="o">}</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">setNome</span><span class="o">(</span><span class="nc">String</span> <span class="n">nome</span><span class="o">)</span> <span class="o">{</span> <span class="k">this</span><span class="o">.</span><span class="na">nome</span> <span class="o">=</span> <span class="n">nome</span><span class="o">;</span> <span class="o">}</span>

    <span class="kd">public</span> <span class="nc">ClasseHeroi</span> <span class="nf">getClasse</span><span class="o">()</span> <span class="o">{</span> <span class="k">return</span> <span class="n">classe</span><span class="o">;</span> <span class="o">}</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">setClasse</span><span class="o">(</span><span class="nc">ClasseHeroi</span> <span class="n">classe</span><span class="o">)</span> <span class="o">{</span> <span class="k">this</span><span class="o">.</span><span class="na">classe</span> <span class="o">=</span> <span class="n">classe</span><span class="o">;</span> <span class="o">}</span>

    <span class="c1">// ... outros getters/setters</span>
<span class="o">}</span>

<span class="c1">// Enums para type safety</span>
<span class="kd">enum</span> <span class="nc">ClasseHeroi</span> <span class="o">{</span>
    <span class="no">CAVALEIRO</span><span class="o">,</span> <span class="no">RANGER</span><span class="o">,</span> <span class="no">MAGO</span><span class="o">,</span> <span class="no">LADINA</span><span class="o">,</span> <span class="no">ACROBATA</span><span class="o">,</span> <span class="no">BARBARO</span>
<span class="o">}</span>

<span class="c1">// Classe aninhada para demonstrar objetos complexos</span>
<span class="kd">class</span> <span class="nc">ArmaLendaria</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">nome</span><span class="o">;</span>
    <span class="kd">private</span> <span class="nc">Integer</span> <span class="n">poder</span><span class="o">;</span>
    <span class="kd">private</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="n">encantamentos</span><span class="o">;</span>

    <span class="c1">// Construtores, getters e setters...</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  Repository Inteligente
</h3>

<p>Aqui está onde o Jakarta Data brilha - queries derivadas e type safety:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.caverna.repository</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">com.caverna.model.Heroi</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">com.caverna.model.ClasseHeroi</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.data.repository.Repository</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.data.repository.Query</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.data.repository.OrderBy</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.enterprise.context.ApplicationScoped</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.time.LocalDateTime</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.List</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.Optional</span><span class="o">;</span>

<span class="nd">@Repository</span>
<span class="nd">@ApplicationScoped</span>
<span class="kd">public</span> <span class="kd">interface</span> <span class="nc">HeroiRepository</span> <span class="kd">extends</span> <span class="nc">NoSQLRepository</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="o">{</span>

    <span class="c1">// Queries derivadas - o JNoSQL gera automaticamente!</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">findByClasse</span><span class="o">(</span><span class="nc">ClasseHeroi</span> <span class="n">classe</span><span class="o">);</span>

    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">findByAtivoTrue</span><span class="o">();</span>

    <span class="nd">@OrderBy</span><span class="o">(</span><span class="s">"experiencia DESC"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">findByExperienciaGreaterThan</span><span class="o">(</span><span class="nc">Integer</span> <span class="n">minExperiencia</span><span class="o">);</span>

    <span class="c1">// Query customizada - aqui usamos a linguagem própria do JNoSQL</span>
    <span class="nd">@Query</span><span class="o">(</span><span class="s">"SELECT * FROM herois WHERE habilidadesEspeciais CONTAINS ?1 AND ativo = true"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">buscarPorHabilidade</span><span class="o">(</span><span class="nc">String</span> <span class="n">habilidade</span><span class="o">);</span>

    <span class="c1">// Query mais complexa</span>
    <span class="nd">@Query</span><span class="o">(</span><span class="s">"SELECT * FROM herois WHERE ultimaAventura &gt;= ?1 ORDER BY experiencia DESC LIMIT 5"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">heroisMaisAtivos</span><span class="o">(</span><span class="nc">LocalDateTime</span> <span class="n">dataLimite</span><span class="o">);</span>

    <span class="c1">// Contadores</span>
    <span class="kt">long</span> <span class="nf">countByClasse</span><span class="o">(</span><span class="nc">ClasseHeroi</span> <span class="n">classe</span><span class="o">);</span>

    <span class="c1">// Operações em lote</span>
    <span class="nd">@Query</span><span class="o">(</span><span class="s">"SELECT * FROM herois WHERE classe IN (?1) AND experiencia &gt;= ?2"</span><span class="o">)</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">buscarEliteDeClasses</span><span class="o">(</span><span class="nc">List</span><span class="o">&lt;</span><span class="nc">ClasseHeroi</span><span class="o">&gt;</span> <span class="n">classes</span><span class="o">,</span> <span class="nc">Integer</span> <span class="n">minExperiencia</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  Service Layer com Regras de Negócio
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.caverna.service</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">com.caverna.model.Heroi</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">com.caverna.model.ClasseHeroi</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">com.caverna.repository.HeroiRepository</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.enterprise.context.ApplicationScoped</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.inject.Inject</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.time.LocalDateTime</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.List</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.Optional</span><span class="o">;</span>

<span class="nd">@ApplicationScoped</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">GrupoAventureirosService</span> <span class="o">{</span>

    <span class="nd">@Inject</span>
    <span class="nc">HeroiRepository</span> <span class="n">repository</span><span class="o">;</span>

    <span class="kd">public</span> <span class="nc">Heroi</span> <span class="nf">recrutar</span><span class="o">(</span><span class="nc">Heroi</span> <span class="n">novoHeroi</span><span class="o">)</span> <span class="o">{</span>
        <span class="c1">// Regra de negócio: novos heróis começam com experiência 0</span>
        <span class="n">novoHeroi</span><span class="o">.</span><span class="na">setExperiencia</span><span class="o">(</span><span class="mi">0</span><span class="o">);</span>
        <span class="n">novoHeroi</span><span class="o">.</span><span class="na">setUltimaAventura</span><span class="o">(</span><span class="nc">LocalDateTime</span><span class="o">.</span><span class="na">now</span><span class="o">());</span>
        <span class="k">return</span> <span class="n">repository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">novoHeroi</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">montarGrupoBalanceado</span><span class="o">()</span> <span class="o">{</span>
        <span class="c1">// Lógica para montar um grupo equilibrado</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="n">tanques</span> <span class="o">=</span> <span class="n">repository</span><span class="o">.</span><span class="na">findByClasse</span><span class="o">(</span><span class="nc">ClasseHeroi</span><span class="o">.</span><span class="na">CAVALEIRO</span><span class="o">);</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="n">dps</span> <span class="o">=</span> <span class="n">repository</span><span class="o">.</span><span class="na">findByClasse</span><span class="o">(</span><span class="nc">ClasseHeroi</span><span class="o">.</span><span class="na">RANGER</span><span class="o">);</span>
        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="n">suporte</span> <span class="o">=</span> <span class="n">repository</span><span class="o">.</span><span class="na">findByClasse</span><span class="o">(</span><span class="nc">ClasseHeroi</span><span class="o">.</span><span class="na">MAGO</span><span class="o">);</span>

        <span class="c1">// Aqui você implementaria a lógica de balanceamento...</span>
        <span class="k">return</span> <span class="n">repository</span><span class="o">.</span><span class="na">findByAtivoTrue</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">promoverHeroi</span><span class="o">(</span><span class="nc">String</span> <span class="n">heroiId</span><span class="o">,</span> <span class="nc">Integer</span> <span class="n">bonusExperiencia</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">Optional</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="n">heroi</span> <span class="o">=</span> <span class="n">repository</span><span class="o">.</span><span class="na">findById</span><span class="o">(</span><span class="n">heroiId</span><span class="o">);</span>
        <span class="k">if</span> <span class="o">(</span><span class="n">heroi</span><span class="o">.</span><span class="na">isPresent</span><span class="o">())</span> <span class="o">{</span>
            <span class="nc">Heroi</span> <span class="n">h</span> <span class="o">=</span> <span class="n">heroi</span><span class="o">.</span><span class="na">get</span><span class="o">();</span>
            <span class="n">h</span><span class="o">.</span><span class="na">setExperiencia</span><span class="o">(</span><span class="n">h</span><span class="o">.</span><span class="na">getExperiencia</span><span class="o">()</span> <span class="o">+</span> <span class="n">bonusExperiencia</span><span class="o">);</span>
            <span class="n">h</span><span class="o">.</span><span class="na">setUltimaAventura</span><span class="o">(</span><span class="nc">LocalDateTime</span><span class="o">.</span><span class="na">now</span><span class="o">());</span>
            <span class="n">repository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">h</span><span class="o">);</span>
        <span class="o">}</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">buscarEspecialistas</span><span class="o">(</span><span class="nc">String</span> <span class="n">habilidade</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="n">repository</span><span class="o">.</span><span class="na">buscarPorHabilidade</span><span class="o">(</span><span class="n">habilidade</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  API REST Moderna
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.caverna.resource</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">com.caverna.model.Heroi</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">com.caverna.model.ClasseHeroi</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">com.caverna.service.GrupoAventureirosService</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.inject.Inject</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.ws.rs.*</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.ws.rs.core.MediaType</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">jakarta.ws.rs.core.Response</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.List</span><span class="o">;</span>

<span class="nd">@Path</span><span class="o">(</span><span class="s">"/herois"</span><span class="o">)</span>
<span class="nd">@Produces</span><span class="o">(</span><span class="nc">MediaType</span><span class="o">.</span><span class="na">APPLICATION_JSON</span><span class="o">)</span>
<span class="nd">@Consumes</span><span class="o">(</span><span class="nc">MediaType</span><span class="o">.</span><span class="na">APPLICATION_JSON</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">HeroiResource</span> <span class="o">{</span>

    <span class="nd">@Inject</span>
    <span class="nc">GrupoAventureirosService</span> <span class="n">service</span><span class="o">;</span>

    <span class="nd">@POST</span>
    <span class="nd">@Path</span><span class="o">(</span><span class="s">"/recrutar"</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">Response</span> <span class="nf">recrutar</span><span class="o">(</span><span class="nc">Heroi</span> <span class="n">heroi</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">Heroi</span> <span class="n">recrutado</span> <span class="o">=</span> <span class="n">service</span><span class="o">.</span><span class="na">recrutar</span><span class="o">(</span><span class="n">heroi</span><span class="o">);</span>
        <span class="k">return</span> <span class="nc">Response</span><span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="mi">201</span><span class="o">).</span><span class="na">entity</span><span class="o">(</span><span class="n">recrutado</span><span class="o">).</span><span class="na">build</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="nd">@GET</span>
    <span class="nd">@Path</span><span class="o">(</span><span class="s">"/grupo-balanceado"</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">montarGrupo</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="n">service</span><span class="o">.</span><span class="na">montarGrupoBalanceado</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="nd">@PUT</span>
    <span class="nd">@Path</span><span class="o">(</span><span class="s">"/{id}/promover/{bonus}"</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">Response</span> <span class="nf">promover</span><span class="o">(</span><span class="nd">@PathParam</span><span class="o">(</span><span class="s">"id"</span><span class="o">)</span> <span class="nc">String</span> <span class="n">id</span><span class="o">,</span> <span class="nd">@PathParam</span><span class="o">(</span><span class="s">"bonus"</span><span class="o">)</span> <span class="nc">Integer</span> <span class="n">bonus</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">service</span><span class="o">.</span><span class="na">promoverHeroi</span><span class="o">(</span><span class="n">id</span><span class="o">,</span> <span class="n">bonus</span><span class="o">);</span>
        <span class="k">return</span> <span class="nc">Response</span><span class="o">.</span><span class="na">ok</span><span class="o">().</span><span class="na">build</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="nd">@GET</span>
    <span class="nd">@Path</span><span class="o">(</span><span class="s">"/especialistas/{habilidade}"</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Heroi</span><span class="o">&gt;</span> <span class="nf">buscarEspecialistas</span><span class="o">(</span><span class="nd">@PathParam</span><span class="o">(</span><span class="s">"habilidade"</span><span class="o">)</span> <span class="nc">String</span> <span class="n">habilidade</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="n">service</span><span class="o">.</span><span class="na">buscarEspecialistas</span><span class="o">(</span><span class="n">habilidade</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h2>
  
  
  Testando Nossa Aventura
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Inicie o Quarkus</span>
./mvnw quarkus:dev

<span class="c"># Recrute o Hank</span>
curl <span class="nt">-X</span> POST localhost:8080/herois/recrutar <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "nome": "Hank",
    "classe": "CAVALEIRO",
    "habilidadesEspeciais": ["Escudo Mágico", "Liderança"],
    "armaLendaria": {
      "nome": "Escudo do Valor",
      "poder": 95,
      "encantamentos": ["Proteção", "Reflexão"]
    }
  }'</span>

<span class="c"># Recrute a Diana</span>
curl <span class="nt">-X</span> POST localhost:8080/herois/recrutar <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "nome": "Diana",
    "classe": "ACROBATA", 
    "habilidadesEspeciais": ["Bastão Mágico", "Acrobacia Extrema"]
  }'</span>
</code></pre>

</div>



<h2>
  
  
  Dicas Práticas úteis
</h2>

<h3>
  
  
  1. <strong>Performance com Índices</strong>
</h3>

<p>Configure índices no Oracle NoSQL para suas queries mais frequentes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// Adicione isso como configuração</span>
<span class="nd">@Query</span><span class="o">(</span><span class="s">"CREATE INDEX idx_classe_experiencia ON herois (classe, experiencia)"</span><span class="o">)</span>
</code></pre>

</div>



<h3>
  
  
  2. <strong>Aproveitando o Key-Value</strong>
</h3>

<p>Use o modo key-value para cache rápido:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Inject</span>
<span class="nd">@ConfigProperty</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"jnosql.keyvalue.database"</span><span class="o">)</span>
<span class="nc">KeyValueTemplate</span> <span class="n">keyValueTemplate</span><span class="o">;</span>

<span class="c1">// Cache rápido</span>
<span class="n">keyValueTemplate</span><span class="o">.</span><span class="na">put</span><span class="o">(</span><span class="s">"ultimo_grupo"</span><span class="o">,</span> <span class="n">grupoAtual</span><span class="o">);</span>
</code></pre>

</div>



<h3>
  
  
  3. <strong>Monitoramento</strong>
</h3>

<p>Configure health checks específicos:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Readiness</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OracleNoSQLHealthCheck</span> <span class="kd">implements</span> <span class="nc">HealthCheck</span> <span class="o">{</span>
    <span class="nd">@Override</span>
    <span class="kd">public</span> <span class="nc">HealthCheckResponse</span> <span class="nf">call</span><span class="o">()</span> <span class="o">{</span>
        <span class="c1">// Teste sua conexão Oracle NoSQL aqui</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h2>
  
  
  Vantagens Reais Dessa Abordagem
</h2>

<ol>
<li>
<strong>Type Safety</strong>: Queries checadas em tempo de compilação</li>
<li>
<strong>Flexibilidade</strong>: Fácil migração entre bancos NoSQL</li>
<li>
<strong>Performance</strong>: JNoSQL Lite sem reflection</li>
<li>
<strong>Produtividade</strong>: Menos código boilerplate</li>
<li>
<strong>Robustez</strong>: Oracle NoSQL com ACID</li>
</ol>

<h2>
  
  
  Referências e Inspirações
</h2>

<p>Este artigo foi inspirado no trabalho do <strong>Otavio Santana</strong> sobre <a href="https://dzone.com/articles/unlocking-the-power-of-oracle-nosql-with-quarkus-s" rel="noopener noreferrer">Oracle NoSQL com Quarkus</a> e na documentação oficial do Eclipse JNoSQL.</p>

<p>Links úteis:</p>

<ul>
<li><a href="https://docs.quarkiverse.io/quarkus-jnosql/dev/index.html" rel="noopener noreferrer">Documentação Quarkus JNoSQL</a></li>
<li><a href="https://www.oracle.com/database/nosql/" rel="noopener noreferrer">Oracle NoSQL Database</a></li>
<li><a href="https://github.com/eclipse/jnosql" rel="noopener noreferrer">Eclipse JNoSQL</a></li>
</ul>

<p>Ah, e se você curtiu esse conteúdo e quer aprender mais sobre Quarkus, te faço um convite! Temos uma comunidade super ativa onde compartilhamos conhecimento, tiramos dúvidas e fazemos networking.</p>

<p><strong>Junte-se ao Quarkus Club:</strong></p>

<ul>
<li>
<strong>Discord</strong>: <a href="https://discord.gg/nMeSs2u4ZJ" rel="noopener noreferrer">https://discord.gg/nMeSs2u4ZJ</a>
</li>
<li>
<strong>YouTube</strong>: <a href="https://www.youtube.com/@QuarkusClub" rel="noopener noreferrer">https://www.youtube.com/@QuarkusClub</a>
</li>
</ul>

<p>E se quiser trocar uma ideia sobre Java, Quarkus, NoSQL ou carreira, me adiciona no LinkedIn:</p>

<ul>
<li>
<strong>LinkedIn</strong>: <a href="https://www.linkedin.com/in/luisfabriciodellamas/" rel="noopener noreferrer">https://www.linkedin.com/in/luisfabriciodellamas/</a>
</li>
</ul>

<h2>
  
  
  Conclusão
</h2>

<p>Eclipse JNoSQL com Oracle NoSQL no Quarkus não é apenas mais uma stack - é uma forma inteligente de trabalhar com dados modernos mantendo a produtividade que você já conhece no Java.</p>

<p>O diferencial está na <strong>padronização</strong>: você aprende uma API e pode usar com qualquer banco NoSQL. O Oracle NoSQL te dá robustez enterprise, o Quarkus te dá performance cloud-native, e o JNoSQL te dá flexibilidade.</p>

<p>Gracias!</p>




<p><em>Por Luis De Llamas</em><br><br>
<em>Developer Advocate</em></p>

