---
Title: Indexando dados intangíveis - otimizando consultas ao banco de dados usando colunas geradas
Description: 
Author: João Pedro
Date: 2026-02-13T22:04:48.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Resumo
</h2>

<p>Filtrar registros com base em cálculos de tempo real é um requisito comum, mas que degrada severamente a performance de grandes bancos de dados ao inutilizar índices. Este estudo de caso aborda a transformação desses "dados intangíveis" em colunas físicas indexáveis utilizando Stored Generated Columns. Em um cenário com 1 milhão de registros, a abordagem proposta eliminou a necessidade de processamento em memória e tabelas temporárias, resultando em um ganho de performance de 4.5x e uma economia de 45% em I/O. O artigo conclui que o custo marginal de armazenamento adicional é um trade-off altamente vantajoso para a escalabilidade do sistema.</p>

<h2>
  
  
  Introdução
</h2>

<p>Em sistemas de gerenciamento de banco de dados (SGBDs), é comum a necessidade de filtrar registros com base em cálculos realizados em tempo real, como por exemplo, o tempo decorrido desde a criação de um registro até o momento do seu processamento em uma requisição. Esses cálculos, embora essenciais para a lógica de negócios, são considerados "dados intangíveis" por não existirem como "colunas físicas" nas tabelas do banco de dados. Neste trabalho, o termo “dados intangíveis” é utilizado como uma abstração conceitual para valores derivados que não são persistidos fisicamente, o que dificulta sua indexação e compromete o desempenho das consultas. O problema surge quando tentamos indexar esses dados intangíveis, pois os otimizadores de consulta (Query Optimizers) geralmente não conseguem utilizar índices B-Tree quando funções matemáticas são aplicadas em cláusulas WHERE, resultando em consultas lentas que executam um Full Table Scan. Essas consultas podem se tornar um gargalo significativo, principalmente em tabelas de aplicações de grande escala, as quais podem conter milhões de registros gerados. Este artigo tem o objetivo de demonstrar como a utilização de colunas geradas (Stored Generated Columns) pode diminuir o tempo de resposta das consultas e reduzir a carga no servidor, transformando dados intangíveis em colunas físicas indexáveis, utilizando índices B-Tree, em troca de um leve custo de armazenamento adicional.</p>

<h2>
  
  
  Fundamentação Teórica
</h2>

<p>A recuperação de dados nos Sistemas Gerenciadores de Banco de Dados (SGBDs) utiliza de maneira geral as estruturas de dados conhecidas como Índices B-Tree. Estas estruturas armazenam chaves ordenadas fisicamente, permitindo a localização rápida de registros sem a necessidade de varrer toda a tabela. No entanto, para que o otimizador de consultas possa fazer uso dessas estruturas, é necessário satisfazer o critério de SARGability (Search ARGument ABility). Este conceito determina se uma consulta é capaz de utilizar os índices existentes para resolver cláusulas de filtragem (WHERE), baseando-se na correspondência direta entre os argumentos de busca e as chaves do índice.</p>

<p>Nesse contexto, as Stored Generated Columns (Colunas Geradas Armazenadas) representam um recurso arquitetural que permite persistir o resultado de uma expressão ou função como uma coluna física. No MySQL, definidas pela sintaxe GENERATED ALWAYS AS (expression) STORED, essas colunas podem ser indexadas como qualquer atributo convencional. Vale ressaltar que este padrão de materialização de cálculos é agnóstico à tecnologia e está presente na maioria dos SGBDs relacionais modernos, sendo implementado no PostgreSQL como Generated Columns, no SQL Server como Persisted Computed Columns e no Oracle como Virtual Columns.</p>

<h2>
  
  
  Metodologia
</h2>

<h3>
  
  
  Cenário
</h3>

<p>O experimento foi conduzido utilizando uma tabela populada com dados sintéticos, contendo 1 milhão de registros com datas de criação e resolução aleatórias, incluindo valores nulos para simular chamados em aberto. Isso representa um cenário típico para aplicações que precisam filtrar dados com base em cálculos de tempo decorrido. O ambiente de teste foi configurado utilizando o MySQL 8.0 rodando em um container Docker, garantindo isolamento de recursos e consistência nos resultados para ambos os cenários a serem testados, além de facilitar a replicação do experimento.</p>

<h3>
  
  
  Definição dos Testes
</h3>

<ul>
<li>
<strong>Cenário A (Controle)</strong>: Neste cenário, a consulta utiliza o cálculo de tempo decorrido em tempo de execução, utilizando a função <code>TIMESTAMPDIFF</code> diretamente na expressão da cláusula WHERE. </li>
</ul>

<p>O impacto técnico observado é que, como o resultado de uma função aplicada sobre uma coluna precisa ser avaliado linha a linha e não está persistido de forma ordenada, o SGBD não consegue percorrer a estrutura B-Tree. Isso torna a consulta não-SARGable, forçando a leitura sequencial da tabela e levando a uma degradação linear da performance à medida que o volume de dados aumenta.</p>

<ul>
<li>
<strong>Cenário B (Experimental)</strong>: Neste cenário, a consulta utiliza uma coluna gerada (SGC) que armazena o resultado do cálculo de tempo decorrido. Sobre esta coluna, foi criado um índice B-Tree, permitindo que o SGBD realize a filtragem através de um Index Range Scan, resultando em um cenário de dados tangíveis. </li>
</ul>

<p>A premissa desta abordagem é que, quando os dados derivados são armazenados como colunas físicas, o SGBD recupera a capacidade de utilizar o índice para acessar diretamente os registros relevantes (tornando a query SARGable). O objetivo é demonstrar que essa abordagem permite ao otimizador evitar a varredura completa, resultando na melhoria de performance esperada para tabelas volumosas.</p>

<p>Para validar a hipótese, foram comparadas duas abordagens distintas, mensurando-se o tempo médio de execução (após aquecimento de cache) e o custo computacional via EXPLAIN ANALYZE, que detalha o plano de execução da consulta, incluindo o tipo de acesso (Full Table Scan vs Index Range Scan) e a quantidade de dados consultados (I/O).</p>

<h2>
  
  
  Resultados e Discussão
</h2>

<h3>
  
  
  Análise do Cenário A
</h3>

<p>No Cenário A, a aplicação da função <code>TIMESTAMPDIFF</code> na coluna de data impediu o otimizador do MySQL de utilizar índices, tornando a consulta non-SARGable. Como resultado, o plano de execução (Figura 1) migrou para um Full Table Scan, obrigando o banco a ler sequencialmente cerca de 994.000 registros para aplicar o filtro em memória. Além da alta latência (~0.59s), essa abordagem gerou um custo alto de I/O, fazendo a movimentação de 368MB de dados internamente.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxelvudpazhyd1dkors5g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxelvudpazhyd1dkors5g.png" alt="Plano de execução do MySQL mostrando Full Table Scan e leitura de 368MB" width="665" height="466"></a><br>
<em>Figura 1: Plano de execução detalhado indicando "Full Table Scan" e custo de leitura de 368MB.</em></p>

<p>O EXPLAIN também revelou o uso de <code>Using temporary</code>, o que indica que o banco precisou alocar recursos adicionais (de buffer ou disco) para agrupar os dados calculados, já que o resultado da função não possui uma ordenação natural, o que é degradante para a performance, especialmente em tabelas de grande volume. Uma vez que o banco não pode utilizar índices, ele é forçado a ler toda a tabela, alocando na memória todos os registros para filtrar, o que acaba por custar ciclos de CPU e banda de memória para a execução do volume dos 368MB (que traduzindo para uma forma mais visível seria algo como 368.000.000 bytes / 1.000.000 linhas = approx 368 bytes por linha).</p>

<h3>
  
  
  Análise do Cenário B
</h3>

<p>No Cenário B, a materialização do cálculo com base em uma Coluna Gerada Armazenada permitiu a criação de um índice B-Tree (idx_faixa_resolucao). Isso alterou a natureza da consulta, permitindo que o otimizador utilizasse a estratégia de Index Range Scan (Figura 2). </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flcdh4dvz3rta67tzmhtv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flcdh4dvz3rta67tzmhtv.png" alt="Plano de execução otimizado no MySQL mostrando Index Range Scan e redução de I/O" width="730" height="483"></a><br>
<em>Figura 2: Plano de execução otimizado utilizando o índice na coluna gerada (Index Range Scan).</em></p>

<p>A mudança amostrada foi drástica se comparada ao cenário anterior: o banco leu apenas as linhas pertinentes ao filtro (~497.000 registros), ignorando completamente os valores nulos e reduzindo a movimentação para 204MB (uma economia de 45% em I/O). O tempo médio de resposta caiu para aproximadamente 0.13s, representando um ganho na performance da consulta de aproximadamente 4.5 vezes. Ademais, como a árvore B-Tree mantém os dados fisicamente ordenados (alocando os dados em páginas ordenadas), a operação de agrupamento (GROUP BY) foi realizada instantaneamente, o que por sua vez eliminou a necessidade da utilização de uma tabela temporária (<code>Using temporary</code>), o que também contribuiu para a redução do custo computacional.</p>

<h3>
  
  
  Tabela: Comparativo de Performance
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Indicador de Performance</th>
<th>Cenário A (Controle)</th>
<th>Cenário B (Experimental)</th>
<th>Resultado</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Latência (Warm Cache)</strong></td>
<td>0.59 segundos</td>
<td><strong>0.13 segundos</strong></td>
<td>4.5x mais rápido</td>
</tr>
<tr>
<td><strong>Volume de Dados (I/O)</strong></td>
<td>368 MB</td>
<td><strong>204 MB</strong></td>
<td>Redução de 45% dos dados movimentados</td>
</tr>
<tr>
<td><strong>Estratégia do Otimizador</strong></td>
<td>Full Table Scan (ALL)</td>
<td><strong>Index Range Scan</strong></td>
<td>Otimizado</td>
</tr>
<tr>
<td><strong>Custo Computacional</strong></td>
<td>Alto (Ordenação em memória)</td>
<td><strong>Baixo (Ordenado no Índice)</strong></td>
<td>Eficiente</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Conclusão
</h2>

<p>Os resultados deste experimento validam que a dependência de dados intangíveis para a filtragem de registros em consultas impõe uma penalidade significativa à performance em bancos de dados relacionais. A utilização de funções de cálculo temporal (como TIMESTAMPDIFF) na cláusula WHERE impede a utilização de índices (tornando a consulta non-SARGable) e força o SGBD a realizar um Full Table Scan, resultando em uma alta movimentação de dados internos (I/O) e processamento de CPU para verificar os dados e descartar registros irrelevantes.</p>

<p>A implementação de Colunas Geradas Armazenadas demonstrou ser uma solução arquitetural eficaz para este cenário. Ao persistir o resultado do cálculo lógico na estrutura da tabela, foi possível converter uma operação de processamento linha a linha em uma operação de busca indexada (Index Range Scan).</p>

<p>As métricas coletadas, sendo uma redução de 45% na leitura de dados (I/O) e um ganho de performance de 4.5 vezes no tempo de resposta, confirmam a eficácia da nova implementação. Portanto, podemos concluir que, em sistemas que demandam filtragem baseada em dados calculados, o custo marginal de armazenamento adicional decorrente da materialização da coluna é um trade-off justificável. A otimização do design do esquema (movendo a complexidade da leitura para a escrita) provou-se superior à execução de cálculos em tempo real, garantindo uma melhor escalabilidade da aplicação frente ao crescimento do volume de dados.</p>

<h2>
  
  
  Referências
</h2>

<ol>
<li><p>What Is Sargability in SQL? — Baeldung on SQL (artigo explicativo sobre SARGability): <a href="https://www.baeldung.com/sql/sargability" rel="noopener noreferrer">https://www.baeldung.com/sql/sargability</a></p></li>
<li><p>MySQL 8.0 Reference Manual — CREATE TABLE and Generated Columns: <a href="https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html" rel="noopener noreferrer">https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html</a></p></li>
<li><p>MySQL 8.0 Reference Manual — Secondary Indexes and Generated Columns: <a href="https://dev.mysql.com/doc/refman/8.0/en/create-table-secondary-indexes.html" rel="noopener noreferrer">https://dev.mysql.com/doc/refman/8.0/en/create-table-secondary-indexes.html</a></p></li>
<li><p>MySQL 8.4 Reference Manual — Optimizer Use of Generated Column Indexes: <a href="https://dev.mysql.com/doc/refman/8.4/en/generated-column-index-optimizations.html" rel="noopener noreferrer">https://dev.mysql.com/doc/refman/8.4/en/generated-column-index-optimizations.html</a></p></li>
<li><p>MySQL 8.4 Reference Manual — WHERE Clause Optimization: <a href="https://dev.mysql.com/doc/refman/8.4/en/where-optimization.html" rel="noopener noreferrer">https://dev.mysql.com/doc/refman/8.4/en/where-optimization.html</a></p></li>
<li><p>Github Repository with Experiment Code/Infos: <a href="https://github.com/SilvaJoaoP/IFTSI-TABD-LP" rel="noopener noreferrer">https://github.com/SilvaJoaoP/IFTSI-TABD-LP</a></p></li>
</ol>

