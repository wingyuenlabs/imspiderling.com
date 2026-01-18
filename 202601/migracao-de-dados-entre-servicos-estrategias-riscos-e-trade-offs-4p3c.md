---
Title: Migração de dados entre serviços: estratégias, riscos e trade-offs
Description: 
Author: Luma Montes
Date: 2026-01-18T21:37:17.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Contexto</strong></p>

<p>Recentemente precisei migrar dezenas de milhares de registros de <br>
auditoria entre dois serviços durante uma task! Era minha primeira vez fazendo uma migração de dados, ainda mais dessa escala e aprendi bastante coisa, então decidi registrar neste artigo.</p>

<p>Basicamente, uma migração de dados envolve pegar um conjunto de informações que estão armazenadas em uma determinada base (seja um banco de dados, uma planilha ou outro formato) e transformá-las para que possam ser consumidas corretamente por outro serviço, respeitando o modelo e as regras esperadas no destino.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftidf3075vbpf88htky3x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftidf3075vbpf88htky3x.png" alt=" " width="800" height="155"></a></p>

<p>As motivações para uma migração são diversas: Em alguns casos, estamos lidando com dados legados que precisam ser movidos para um novo serviço. Em outros, diferentes sistemas passam a compartilhar responsabilidades e precisam de uma interseção consistente de dados. Muitas vezes esse processo acontece de forma planejada, o que permite mais tempo para análise e execução. Mas também pode surgir de maneira urgente, pode ser demanda de um cliente, de um setor ou por uma necessidade operacional inesperada.</p>

<p>A decisão de migrar pode partir tanto do time de produto quanto da engenharia. Ainda assim, é <strong>responsabilidade</strong> do time de <strong>engenharia</strong> avaliar e bater o martelo sobre a viabilidade técnica dessa migração! E essa viabilidade depende diretamente do estudo que a gente faz sobre os dados disponíveis.</p>

<p>Por exemplo, quando a demanda vem de produto, normalmente existe um espaço maior para a engenharia investigar complexidade, riscos e custos. Mas nada disso se resolve apenas com um “dá pra migrar, galera! deixa que eu resolvo”. Antes de qualquer compromisso, é muuuuito importante analisar volumetria, estrutura e complexidade dos dados envolvidos.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftijffltga9lazq3ip299.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftijffltga9lazq3ip299.png" alt=" " width="800" height="275"></a></p>

<p>Um exemplo claro: vale a pena a gente paralisar um ou dois devs por DUAS sprints para migrar 15 registros? Na maioria dos cenários, <strong>não</strong>. O custo de engenharia pode ser alto demais para um impacto pequeno no produto.</p>

<p>Claro que viabilidade não se resume apenas a números. O contexto do produto também importa. Que tipo de dado está sendo migrado? São dados simples ou recursivos? Existem dependências entre registros? Há estados intermediários, históricos ou regras de negócio acopladas a esses dados? Tudo isso influencia diretamente na decisão.</p>

<p>Uma vez que a migração é considerada viável, fica mais claro o conjunto de problemas que precisam ser resolvidos. Em geral, eles se dividem em três grandes etapas:</p>

<ul>
<li>
<strong>Extração</strong>: como os dados serão puxados?</li>
<li>
<strong>Transformação</strong>: como esses dados serão adaptados para o formato esperado?</li>
<li>
<strong>Carregamento</strong>: como os dados transformados serão enviados para o serviço de destino?</li>
</ul>

<p>Foi nesse contexto que eu aprendi sobre <strong>ETL</strong>, que é uma abordagem criada justamente para organizar e estruturar esse tipo de processo (Parabéns, você também aprendeu o que é ETL :))) </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Filolo2cpy7t1xqxes4fa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Filolo2cpy7t1xqxes4fa.png" alt=" " width="800" height="367"></a></p>

<p>Ainda assim, é importante reforçar: <strong>ETL</strong> não é bala de prata. Em muitos casos, criar uma aplicação dedicada para migração pode ser uma complexidade desnecessária. Antes de qualquer implementação, é essencial estudar a viabilidade real da migração, entender o formato dos dados de origem, o esforço de transformação e o custo de manter essa solução também.</p>

<p>De forma geral, ETLs fazem mais sentido quando a migração precisa ser executada mais de uma vez ou reaproveitada no futuro. Para migrações pontuais, uma solução mais simples pode ser suficiente e mais segura.</p>

<h2>
  
  
  Estratégias
</h2>

<p>Quando estou lidando com migrações menores, costumo optar por soluções simples. Em especial, gosto bastante de usar Jupyter Notebook, principalmente quando os dados de origem estão em planilhas ou em bases pequenas. Nesses cenários, não vejo muito sentido em construir uma aplicação inteira só para executar um processo pontual.</p>

<p>Criar um notebook é relativamente simples: basta um arquivo com extensão .ipynb e um kernel configurado localmente. Dependendo do contexto, dá para usar Kotlin, Python, Deno, entre outras opções. A grande vantagem aqui é a rapidez para explorar os dados, testar transformações e iterar sem muitas dificuldades.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fppnmk045glscgqcx19so.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fppnmk045glscgqcx19so.png" alt=" " width="800" height="581"></a></p>

<p>Já para migrações mais complexas, minha preferência passa a ser criar uma aplicação local que roda via terminal — uma CLI simples, mas controlada. Esse tipo de abordagem permite mais previsibilidade, controle de execução e facilidade para lidar com grandes volumes de dados.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo8w59uslh300rzxxop5e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo8w59uslh300rzxxop5e.png" alt=" " width="649" height="448"></a></p>

<p>Aqui, uma preocupação fundamental é ter uma estratégia de rollback. Nunca dá para assumir que a migração vai rodar perfeitamente de primeira. Sempre que possível, vale testar em ambiente de staging antes de executar em produção, além de alinhar muito bem o rollout com o time.</p>

<p>A forma de carregar os dados também faz diferença. Em alguns casos, inserir diretamente no banco pode ser suficiente. Em outros, usar uma API ou até publicar eventos em fila é mais seguro, principalmente quando estamos lidando com grandes volumes e queremos processar tudo de forma assíncrona.</p>

<p>No fim das contas, o formato do ETL importa menos do que o objetivo final. Ele pode ser um notebook, uma CLI ou até um script simples — desde que atenda às necessidades de extração, transformação e carregamento de forma segura e previsível.</p>

<h3>
  
  
  Desafios práticos que você vai enfrentar
</h3>

<h5>
  
  
  Dados faltando ou inconsistentes
</h5>

<p>Em migrações reais, é muito comum encontrar dados incompletos, inconsistentes ou marcados como removidos (como casos de soft delete). Esses cenários precisam ser tratados explicitamente, seja ignorando registros, preenchendo valores padrão ou sinalizando dados desconhecidos.</p>

<h5>
  
  
  Enriquecimento de dados
</h5>

<p>Nem sempre todas as informações necessárias estão em um único lugar. Em alguns casos, é preciso bater em outros bancos de dados, outros serviços/apis pra buscar as informações que você precisa. É bom estar de olho nesse ponto desde o início, pois aumenta a complexidade</p>




<h3>
  
  
  Trade-offs técnicos
</h3>

<p>Abaixo estão alguns dos principais trade-offs que enfrentei: </p>

<h4>
  
  
  Velocidade vs correção
</h4>

<p>Quando a migração é urgente, você precisa decidir até onde aceitar inconsistências nos dados sem comprometer a entrega.</p>

<h4>
  
  
  Jupyter Notebook vs CLI
</h4>

<p>Para migrações pequenas, notebook resolve e acelera muito! Mas por experiência própria, ele pode crescer muito e dificultar a legibilidade e manutenção do script. Quando isso acontece, vale  mais a pena adotar uma CLI. </p>

<h3>
  
  
  Migrar tudo vs migrar só o essencial
</h3>

<p>Nem todo dado merece ser migrado, é muito importante alinhar com o time de produto quais dados serão preservados e quais dados podemos aceitar como "perdidos", de forma consciente.</p>

<h4>
  
  
  Processar tudo de uma vez vs em lotes
</h4>

<p>Por experiência própria, recomendo ir processando os registros em batches e ir verificando após cada uma se os dados foram da forma esperada. Isso é muito melhor do que ter que processar tudo de uma vez e precisar reprocessar por qualquer errinho nos tratamentos de dados. </p>

<h3>
  
  
  Conclusão
</h3>

<p>Se ninguém percebeu que a migração de dados aconteceu em produção, provavelmente foi um bom sinal. No fim, tudo se resume a contexto, equilíbrio e trade-offs — e a manter o time alinhado sobre o que é possível entregar.</p>

