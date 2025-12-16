---
Title: Como provisionar um cluster PostgreSQL no DBaaS da Magalu Cloud com failover automático
Description: 
Author: Monica Hillman
Date: 2025-12-16T21:32:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>Bancos de dados relacionais continuam sendo componentes centrais em aplicações modernas. Quando o banco é crítico para o negócio, <strong>alta disponibilidade</strong> deixa de ser diferencial e passa a ser requisito.</p>

<p>A <strong><a href="https://magalu.cloud/?utm_source=site&amp;utm_medium=organico&amp;utm_campaign=artigos+comunidade&amp;utm_id=artigos+comunidade" rel="noopener noreferrer">Magalu Cloud</a></strong> oferece o <strong>Database as a Service (DBaaS)</strong>, que abstrai a complexidade operacional de bancos de dados gerenciados, incluindo replicação e failover automático.</p>

<p>Neste artigo, você vai aprender:</p>

<ul>
<li>O que é um cluster PostgreSQL no DBaaS da Magalu Cloud</li>
<li>Quando faz sentido usar alta disponibilidade</li>
<li>Como provisionar um cluster PostgreSQL com failover automático usando a CLI <code>mgc</code>
</li>
</ul>




<h2>
  
  
  O que é um cluster PostgreSQL
</h2>

<p>Um <strong>cluster PostgreSQL</strong> no DBaaS da Magalu Cloud é uma configuração de <strong>alta disponibilidade</strong>, formada por múltiplos nós do banco de dados:</p>

<ul>
<li>
<strong>1 nó primário</strong>, responsável por leitura e escrita</li>
<li>
<strong>Réplicas em espera (standby)</strong>, distribuídas em Zonas de Disponibilidade diferentes</li>
<li>
<strong>Failover automático</strong>, que promove uma réplica a primária em caso de falha</li>
</ul>

<p>As alterações realizadas no nó primário são replicadas continuamente para as réplicas, reduzindo o tempo de indisponibilidade e o risco de perda de dados.</p>

<p>A distribuição entre <strong>Zonas de Disponibilidade (AZs)</strong> garante maior resiliência dentro de uma mesma região.</p>




<h2>
  
  
  Quando usar um cluster PostgreSQL
</h2>

<p>Clusters PostgreSQL com failover automático são indicados quando a aplicação:</p>

<ul>
<li>Não pode tolerar indisponibilidade prolongada</li>
<li>Precisa de replicação contínua de dados</li>
<li>Depende de alta disponibilidade para o negócio</li>
<li>Quer reduzir esforço operacional com gerenciamento de banco</li>
</ul>




<h2>
  
  
  Provisionamento via CLI (<code>mgc</code>)
</h2>

<h3>
  
  
  Pré-requisitos
</h3>

<p>Antes de começar, você precisa:</p>

<ul>
<li>Ter a CLI <strong><code>mgc</code></strong> instalada e configurada</li>
<li>Ter acesso à região desejada</li>
<li>Identificar a engine PostgreSQL e o tipo de instância adequado</li>
</ul>




<h2>
  
  
  1. Listar engines disponíveis
</h2>

<p>Primeiro, identifique a engine PostgreSQL disponível:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc dbaas engines list <span class="nt">--status</span><span class="o">=</span>ACTIVE
</code></pre>

</div>



<p>Anote o <code>id</code> correspondente ao PostgreSQL (por exemplo, PostgreSQL 16).</p>




<h2>
  
  
  2. Listar tipos de instância compatíveis
</h2>

<p>Em seguida, liste os tipos de instância disponíveis para essa engine:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc dbaas instance-types list <span class="se">\</span>
  <span class="nt">--engine-id</span><span class="o">=</span><span class="s2">"SEU_ENGINE_ID"</span> <span class="se">\</span>
  <span class="nt">--status</span><span class="o">=</span>ACTIVE
</code></pre>

</div>



<p>Cada tipo de instância define CPU e memória de <strong>cada nó do banco</strong>.</p>

<h3>
  
  
  Entendendo o <code>compatible_product</code>
</h3>

<p>No resultado, observe o campo:</p>

<ul>
<li>
<code>SINGLE_INSTANCE</code> - pode ser usado apenas para instâncias simples</li>
<li>
<code>CLUSTER</code> - pode ser usado apenas para clusters com alta disponibilidade</li>
</ul>

<p>Esse campo é <strong>informativo</strong> e serve para orientar a escolha correta do <code>instance-type-id</code>.</p>




<h2>
  
  
  3. Criar um cluster PostgreSQL com failover automático
</h2>

<p>A criação do cluster é feita com o comando <strong><code>mgc dbaas clusters create</code></strong>.</p>

<p>Diferente de uma instância simples, o cluster já nasce com múltiplos nós e com alta disponibilidade configurada desde o início.</p>

<blockquote>
<p>Apenas tipos de instância com <code>compatible_product: CLUSTER</code> podem ser utilizados neste comando.</p>
</blockquote>

<h3>
  
  
  Exemplo de criação de cluster
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc dbaas clusters create <span class="se">\</span>
  <span class="nt">--name</span><span class="o">=</span><span class="s2">"meu-postgres-cluster"</span> <span class="se">\</span>
  <span class="nt">--user</span><span class="o">=</span><span class="s2">"admin"</span> <span class="se">\</span>
  <span class="nt">--password</span><span class="o">=</span><span class="s2">"SenhaSegura!"</span> <span class="se">\</span>
  <span class="nt">--engine-id</span><span class="o">=</span><span class="s2">"ID_DA_ENGINE_POSTGRESQL"</span> <span class="se">\</span>
  <span class="nt">--instance-type-id</span><span class="o">=</span><span class="s2">"ID_DO_INSTANCE_TYPE_CLUSTER"</span> <span class="se">\</span>
  <span class="nt">--volume</span>.size<span class="o">=</span>100 <span class="se">\</span>
  <span class="nt">--volume</span>.type<span class="o">=</span><span class="s2">"CLOUD_NVME20K"</span>
</code></pre>

</div>



<h3>
  
  
  Principais parâmetros
</h3>

<ul>
<li><p><code>--name</code><br>
Nome do cluster PostgreSQL</p></li>
<li><p><code>--user</code><br>
Usuário administrador do banco</p></li>
<li><p><code>--password</code><br>
Senha do usuário administrador</p></li>
<li><p><code>--engine-id</code><br>
Identificador da engine PostgreSQL</p></li>
<li><p><code>--instance-type-id</code><br>
Tipo de instância compatível com <strong>CLUSTER</strong>, que define CPU e memória de cada nó</p></li>
<li><p><code>--volume.size</code><br>
Tamanho do volume de dados em GiB</p></li>
<li><p><code>--volume.type</code><br>
Tipo de armazenamento gerenciado do DBaaS - Você pode verificar os valores na <a href="https://docs.magalu.cloud/docs/dbaas/postgres/create-manage/create-instance/?utm_source=devto" rel="noopener noreferrer">documentação</a>.</p></li>
</ul>

<p>Após a execução, o cluster entra em processo de provisionamento, o que pode levar alguns minutos.</p>




<h2>
  
  
  4. Acompanhar o status do cluster e das instâncias
</h2>

<p>Para acompanhar o status do cluster:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc dbaas clusters get <span class="s2">"ID_DO_CLUSTER"</span>
</code></pre>

</div>



<p>Após a criação, você pode listar os nós do cluster:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc dbaas instances list
</code></pre>

</div>



<p>E obter detalhes de um nó específico:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc dbaas instances get <span class="s2">"ID_DA_INSTANCIA"</span>
</code></pre>

</div>






<h2>
  
  
  Failover automático na prática
</h2>

<p>Quando o cluster está ativo, o DBaaS da Magalu Cloud monitora continuamente o nó primário. Em caso de falha:</p>

<ul>
<li>Uma réplica é promovida automaticamente a primária</li>
<li>O endpoint do cluster permanece o mesmo</li>
<li>A aplicação precisa apenas lidar com reconexões</li>
</ul>

<p>Todo o processo ocorre sem intervenção manual.</p>




<h2>
  
  
  Boas práticas
</h2>

<ul>
<li>Utilize senhas fortes para usuários do banco</li>
<li>Escolha tipos de instância compatíveis com a carga da aplicação</li>
<li>Monitore o status das instâncias via CLI</li>
<li>Garanta que a aplicação lide bem com reconexões</li>
</ul>




<h2>
  
  
  Conclusão
</h2>

<p>Provisionar um cluster PostgreSQL com failover automático no DBaaS da Magalu Cloud permite unir <strong>simplicidade operacional</strong> e <strong>alta disponibilidade</strong>.</p>

<p>Com a CLI <code>mgc</code>, você consegue:</p>

<ul>
<li>Identificar engines e tipos de instância</li>
<li>Criar clusters PostgreSQL com alta disponibilidade desde a criação</li>
<li>Contar com failover automático gerenciado pela plataforma</li>
<li>Monitorar o estado do banco de dados de forma centralizada</li>
</ul>

<p>Isso reduz a complexidade operacional e permite que o time foque no desenvolvimento da aplicação, aproveitando a infraestrutura gerenciada da <a href="https://magalu.cloud/?utm_source=site&amp;utm_medium=organico&amp;utm_campaign=artigos+comunidade&amp;utm_id=artigos+comunidade" rel="noopener noreferrer">Magalu Cloud</a>.</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/x3sW2stCHgI">
</iframe>
</p>

