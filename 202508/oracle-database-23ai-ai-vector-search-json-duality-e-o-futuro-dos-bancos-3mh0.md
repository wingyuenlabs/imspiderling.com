---
Title: Oracle Database 23ai: AI Vector Search, JSON Duality e o Futuro dos Bancos
Description: 
Author: Luis Fabrício De Llamas
Date: 2025-08-23T20:30:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>Fala, pessoal! Hoje vou compartilhar minha experiência explorando o Oracle Database 23ai! Depois de alguns anos trabalhando principalmente com MySQL, PostgreSQL e outros bancos, resolvi dar uma olhada no que a Oracle anda aprontando. Minha última experiência séria com Oracle foi lá em 2020 com o 12c. <br>
Spoiler: mudou MUITA coisa, e de forma impressionante!</p>
<h2>
  
  
  Por Que Oracle 23ai é Diferente?
</h2>

<p>O que me chamou atenção logo de cara: Oracle Database 23ai é a próxima versão de suporte de longo prazo do Oracle Database, substituindo o 19c. Uma versão de longo prazo oferece cinco anos de suporte premium e três anos de suporte estendido.</p>

<p><strong>O que me fez olhar para Oracle novamente</strong>: Curiosidade sobre como eles estão lidando com IA nativa, já que nos outros bancos sempre precisamos de extensões ou ferramentas externas para funcionalidades similares.</p>
<h2>
  
  
  AI Vector Search: O Game Changer
</h2>

<p>A funcionalidade que mais me impressionou foi o <strong>AI Vector Search</strong>. Sério, é como se o banco de dados ganhasse superpoderes semânticos.</p>
<h3>
  
  
  O Que Isso Significa na Prática?
</h3>

<p>Oracle Database 23ai agora é um banco de dados vetorial. AI Vector Search permite criar e armazenar vetores de IA diretamente no banco em seu formato nativo. Esses vetores capturam embeddings para vários tipos de dados, incluindo texto, imagens, áudio ou até mesmo vídeo.</p>

<p>Imagine isso: ao invés de buscar por palavras-chave exatas, você pode buscar por <strong>significado</strong>. É tipo ter um Google semântico direto no seu banco Oracle.</p>
<h3>
  
  
  Experimentando com RAG
</h3>

<p>Uma das coisas mais legais que testei foi o <strong>RAG (Retrieval Augmented Generation)</strong>:</p>

<p>RAG ajuda organizações a fornecer respostas personalizadas para perguntas de negócio sem o alto custo de retreinar ou fazer fine-tuning dos LLMs.</p>

<p>O processo é fascinante:</p>

<ol>
<li>Você faz uma pergunta em linguagem natural</li>
<li>O banco busca dados similares semanticamente</li>
<li>Os resultados são formatados como prompt e contexto para o LLM. O LLM recebe dados de negócio atualizados, reduzindo assim as alucinações</li>
</ol>
<h3>
  
  
  Novo Tipo de Dados VECTOR
</h3>

<p>O tipo de dados VECTOR é introduzido com o lançamento do Oracle Database 23ai, fornecendo a base para armazenar embeddings vetoriais junto com dados de negócio no banco.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">acdc_songs</span> <span class="p">(</span>
    <span class="n">song_id</span> <span class="nb">INT</span><span class="p">,</span> 
    <span class="n">song_title</span> <span class="n">VARCHAR2</span><span class="p">(</span><span class="mi">100</span><span class="p">),</span>
    <span class="n">lyrics</span> <span class="k">CLOB</span><span class="p">,</span> 
    <span class="n">lyrics_vector</span> <span class="n">VECTOR</span>
<span class="p">);</span>
</code></pre>

</div>



<p>É simples assim! Agora posso armazenar vetores diretamente nas tabelas, lado a lado com dados relacionais tradicionais. Imaginem buscar músicas do AC/DC por <strong>sentimento</strong> ou <strong>tema</strong>, não apenas por nome!</p>

<h2>
  
  
  JSON Relational Duality: Unindo Dois Mundos
</h2>

<p>Se AI Vector Search é impressionante, <strong>JSON Relational Duality Views</strong> é revolucionário para quem trabalha com APIs modernas.</p>

<h3>
  
  
  O Problema que Resolve
</h3>

<p>Sabe aquela eterna briga entre "usar SQL ou NoSQL"? JSON Relational Duality é uma nova capacidade de modelagem de dados que apresenta visualizações de documento JSON atualizáveis e consistentes sobre dados relacionais.</p>

<p>Permite que os dados permaneçam em tabelas relacionais normalizadas, mas sejam acessados como documentos JSON. Essa dualidade de acesso e armazenamento elimina a necessidade de escolher entre modelos relacionais e de documento para desenvolvimento de aplicações.</p>

<h3>
  
  
  Testando na Prática
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">OR</span> <span class="k">REPLACE</span> <span class="n">JSON</span> <span class="n">RELATIONAL</span> <span class="n">DUALITY</span> <span class="k">VIEW</span> <span class="n">acdc_member_dv</span> <span class="k">AS</span>
<span class="k">SELECT</span> <span class="n">JSON</span> <span class="p">{</span>
    <span class="s1">'_id'</span> <span class="p">:</span> <span class="n">m</span><span class="p">.</span><span class="n">member_id</span><span class="p">,</span>
    <span class="s1">'name'</span> <span class="p">:</span> <span class="n">m</span><span class="p">.</span><span class="n">first_name</span> <span class="o">||</span> <span class="s1">' '</span> <span class="o">||</span> <span class="n">m</span><span class="p">.</span><span class="n">last_name</span><span class="p">,</span>
    <span class="s1">'instrument'</span> <span class="p">:</span> <span class="n">m</span><span class="p">.</span><span class="n">instrument</span><span class="p">,</span>
    <span class="s1">'join_date'</span> <span class="p">:</span> <span class="n">m</span><span class="p">.</span><span class="n">join_date</span><span class="p">,</span>
    <span class="s1">'band_info'</span> <span class="p">:</span> <span class="p">{</span>
        <span class="s1">'band_id'</span> <span class="p">:</span> <span class="n">b</span><span class="p">.</span><span class="n">band_id</span><span class="p">,</span>
        <span class="s1">'band_name'</span> <span class="p">:</span> <span class="n">b</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
        <span class="s1">'formed_year'</span> <span class="p">:</span> <span class="n">b</span><span class="p">.</span><span class="n">formed_year</span><span class="p">,</span>
        <span class="s1">'origin'</span> <span class="p">:</span> <span class="n">b</span><span class="p">.</span><span class="n">origin</span>
    <span class="p">}</span>
<span class="p">}</span>
<span class="k">FROM</span> <span class="n">members</span> <span class="n">m</span><span class="p">,</span> <span class="n">bands</span> <span class="n">b</span> 
<span class="k">WHERE</span> <span class="n">m</span><span class="p">.</span><span class="n">band_id</span> <span class="o">=</span> <span class="n">b</span><span class="p">.</span><span class="n">band_id</span> 
<span class="k">AND</span> <span class="n">b</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="s1">'AC/DC'</span>
<span class="k">WITH</span> <span class="k">INSERT</span> <span class="k">UPDATE</span> <span class="k">DELETE</span><span class="p">;</span>
</code></pre>

</div>



<p>O resultado? Posso fazer um simples:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">acdc_member_dv</span><span class="p">;</span>
</code></pre>

</div>



<p>E recebo JSON estruturado com os membros da banda! Mas o mais legal: <strong>posso fazer INSERT, UPDATE e DELETE diretamente no JSON</strong>, e ele atualiza as tabelas relacionais automaticamente. </p>

<p>Imaginem uma API que retorna informações dos membros do AC/DC como JSON, mas por trás usa tabelas relacionais normalizadas!</p>

<h2>
  
  
  Comparando com o Que Conhecia (Oracle 12c vs MySQL/PostgreSQL)
</h2>

<p>Vindo de uma experiência antiga com Oracle 12c e anos recentes com MySQL e PostgreSQL, as diferenças são gritantes:</p>

<h3>
  
  
  Oracle 12c
</h3>

<ul>
<li>JSON básico como CLOB</li>
<li>Recursos de IA/ML praticamente inexistentes</li>
<li>Principalmente focado em dados relacionais tradicionais</li>
<li>Complexo para configurar e usar</li>
</ul>

<h3>
  
  
  MySQL/PostgreSQL (minha experiência recente)
</h3>

<ul>
<li>JSON nativo no PostgreSQL, mais limitado no MySQL</li>
<li>Extensões para machine learning (como pgvector no Postgres)</li>
<li>Ecosistema open source </li>
<li>Simplicidade para desenvolvimento moderno</li>
</ul>

<h3>
  
  
  Oracle 23ai
</h3>

<ul>
<li>23ai oferece o novo tipo de dados JSON, JSON Relational Duality Views avançadas para integração relacional-JSON, funções SQL/JSON aprimoradas, mecanismos de indexação melhorados e opções de armazenamento binário eficientes</li>
<li>IA integrada ao kernel do banco (isso é NOVO!)</li>
<li>Busca semântica nativa</li>
<li>Combine livremente APIs JSON, REST e SQL padrão, dependendo do seu caso de uso</li>
</ul>

<p><strong>Honestamente</strong>: Se alguém me perguntasse há alguns anos se consideraria Oracle para projetos novos, eu diria "talvez não". Agora? Oracle está competindo de igual para igual com soluções modernas, e em alguns aspectos, ultrapassou.</p>

<h2>
  
  
  Outras Features que Me Surpreenderam
</h2>

<h3>
  
  
  SQL Firewall
</h3>

<p>SQL Firewall é uma funcionalidade de segurança do banco construída no kernel do Oracle Database 23ai que inspeciona todas as conexões de entrada e declarações SQL e permite/registra/bloqueia atividades não autorizadas seguindo políticas específicas do usuário do banco.</p>

<h3>
  
  
  Boolean Data Type Nativo
</h3>

<p>Finalmente! Oracle Database agora suporta o tipo de dados Boolean compatível com o padrão ISO SQL. Isso permite armazenar valores True e False em tabelas e usar expressões booleanas em declarações SQL.</p>

<p>Não mais aquele <code>VARCHAR2(1)</code> com 'Y'/'N'. Aleluia!</p>

<h3>
  
  
  Property Graphs
</h3>

<p>Oracle 23ai vem com suporte nativo aprimorado para modelos de dados property graph, permitindo construir e consultar grafos junto com dados transacionais, JSON e espaciais.</p>

<h2>
  
  
  Experimentando com Oracle 23ai Free
</h2>

<p>Oracle Database 23ai Free oferece aos desenvolvedores acesso às funcionalidades mais recentes que simplificam o desenvolvimento.</p>

<p>Baixei a versão free e foi surpreendentemente fácil de configurar. Em menos de 30 minutos tinha um ambiente completo rodando com todas as funcionalidades de AI.</p>

<h2>
  
  
  Possibilidades que Me Empolgam
</h2>

<h3>
  
  
  1. Chatbot AC/DC Inteligente
</h3>

<p>Com AI Vector Search + RAG, posso criar um chatbot que responde perguntas sobre a história da banda, letras das músicas, curiosidades dos álbuns - tudo baseado em dados reais sem alucinar informações.</p>

<p>"Qual música do AC/DC fala sobre estrada?" → O sistema busca semanticamente e encontra "Highway to Hell", "T.N.T.", etc.</p>

<h3>
  
  
  2. APIs de Discografia Híbridas
</h3>

<p>JSON Duality Views permite APIs que servem tanto JSON (para apps mobile) quanto consultas SQL (para relatórios), usando a mesma base de dados.</p>

<h3>
  
  
  3. Busca Semântica em Letras
</h3>

<p>Imaginem buscar músicas por <strong>conceito</strong> ao invés de palavras exatas:</p>

<ul>
<li>"Músicas sobre rebeldia" → "For Those About to Rock", "Dirty Deeds Done Dirt Cheap"</li>
<li>"Música para festa" → "Let There Be Rock", "You Shook Me All Night Long"</li>
</ul>

<h3>
  
  
  4. Análise de Sentimento das Letras
</h3>

<p>Processar o sentimento das letras do AC/DC e classificar as músicas por energia, tema ou emoção - diretamente no banco.</p>

<h2>
  
  
  O Veredicto (de Quem Voltou ao Oracle)
</h2>

<p>Oracle 23ai não é apenas uma atualização incremental - é uma mudança de paradigma. Depois de anos no ecossistema MySQL/PostgreSQL, devo admitir: fiquei impressionado.</p>

<p>Comparado às soluções anteriores de banco Oracle, esta é a primeira versão que foca pesadamente em funcionalidades para IA. É também por isso que a Oracle decidiu renomear de Database 23c para o mais tecnicamente atrativo Database 23ai.</p>

<p><strong>Por que isso me fez repensar Oracle</strong>:</p>

<ul>
<li>
<strong>Facilidade de uso</strong>: Oracle 23ai Free baixou MUITO a barreira de entrada</li>
<li>
<strong>Funcionalidades nativas de IA</strong>: Não preciso de extensões ou gambiarras como em outros bancos</li>
<li>
<strong>JSON Duality</strong>: Resolve de vez o dilema SQL vs NoSQL</li>
<li>
<strong>Performance</strong>: Os benchmarks que vi são impressionantes</li>
</ul>

<p>Se você, como eu, teve experiências antigas com Oracle e migrou para outras soluções, minha recomendação: baixe o 23ai Free e experimente. Me surpreendi positivamente, e talvez você também se surpreenda.</p>

<p>A era da IA chegou ao Oracle, e francamente... eles fizeram o dever de casa!</p>

<p>Ah, e se você curtiu esse conteúdo e quer aprender mais sobre tecnologias emergentes, te faço um convite! Temos uma comunidade super ativa onde compartilhamos conhecimento, tiramos dúvidas e fazemos networking.</p>

<p><strong>Junte-se ao Quarkus Club:</strong></p>

<ul>
<li>
<strong>Discord</strong>: <a href="https://discord.gg/nMeSs2u4ZJ" rel="noopener noreferrer">https://discord.gg/nMeSs2u4ZJ</a>
</li>
<li>
<strong>YouTube</strong>: <a href="https://www.youtube.com/@QuarkusClub" rel="noopener noreferrer">https://www.youtube.com/@QuarkusClub</a>
</li>
</ul>

<p>E se quiser trocar uma ideia sobre Oracle, IA, ou qualquer tech em geral, me adiciona no LinkedIn:</p>

<ul>
<li>
<strong>LinkedIn</strong>: <a href="https://www.linkedin.com/in/luisfabriciodellamas/" rel="noopener noreferrer">https://www.linkedin.com/in/luisfabriciodellamas/</a>
</li>
</ul>

<h2>
  
  
  Referências
</h2>

<ul>
<li><a href="https://www.oracle.com/database/23ai/" rel="noopener noreferrer">Oracle Database 23ai Features</a></li>
<li><a href="https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/overview-ai-vector-search.html" rel="noopener noreferrer">AI Vector Search User's Guide</a></li>
<li><a href="https://www.oracle.com/database/json-relational-duality/" rel="noopener noreferrer">JSON Relational Duality</a></li>
<li><a href="https://www.oracle.com/database/free/" rel="noopener noreferrer">Oracle Database 23ai Free Download</a></li>
</ul>




<p><em>Por Luis De Llamas</em><br><br>
<em>Developer Advocate</em></p>

