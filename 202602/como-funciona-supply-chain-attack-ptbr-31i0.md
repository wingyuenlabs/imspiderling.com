---
Title: Como Funciona Supply Chain Attack -PTBR
Description: 
Author: Ronaldo Modesto
Date: 2026-02-19T22:08:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>Access the english version here: <a href="https://dev.to/r9n/how-supply-chain-attacks-work-5bc">Click Here</a></p>

<p>Olá pessoal. <br>
Hoje quero trazer um pouquinho a mais de conhecimento a respeito de um tipo de ataque que tem se tornado cada vez mais frequente. O Supply Chain Attack, ou ataque de cadeia de suprimentos.<br>
Bora lá ver como isso funciona no ecossistema npm e o que podemos fazer para mitigar esse risco.<br>
<strong>Lembrando que aqui eu trouxe o npm apenas como exemplo, mas esse tipo de ataque pode ocorrer em outros gerenciadores de pacotes também!</strong></p>

<p>Aqui você encontra uma ferramenta que eu desenvolvi para mitigar esse tipo de ataque: <a href="https://github.com/R9n/safeinstall" rel="noopener noreferrer">Safeinstall</a></p>

<p>Caso queira ver como a ferramenta funciona, dá uma olhada aqui <a href="https://dev.to/r9n/safeinstall-um-aliado-no-combate-a-ataques-de-supply-chain-47mh">SafeInstall</a></p>


<h2>
  
  
  Introdução
</h2>

<p>Quantas vezes por dia você executa <code>npm install</code>? Para a maioria dos desenvolvedores JavaScript e Node.js, a resposta é: várias. Essa rotina aparentemente inocente — instalar uma dependência para resolver um problema — esconde um vetor de ataque cada vez mais explorado por cibercriminosos: o <strong>ataque de supply chain</strong> (ou ataque de cadeia de suprimentos).</p>

<p>Neste artigo, exploramos o que são esses ataques, como funcionam na prática usando um projeto de demonstração real, quais seriam as consequências em ambientes de produção e como você pode se proteger.</p>

<p>O projeto utilizado no artigo pode ser encontrado aqui: <a href="https://github.com/R9n/supply-chain-attack-example" rel="noopener noreferrer">Projeto Exemplo</a></p>


<h2>
  
  
  O que é um Supply Chain Attack?
</h2>

<p>Um <strong>ataque de supply chain</strong> ocorre quando um atacante compromete algum componente que faz parte da cadeia de suprimentos de software — ou seja, algo que os desenvolvedores ou sistemas confiam e utilizam sem questionar. No ecossistema npm, isso se materializa principalmente através de:</p>

<ol>
<li>
<strong>Pacotes maliciosos</strong> criados do zero para parecerem legítimos</li>
<li>
<strong>Pacotes legítimos comprometidos</strong> (manutenção abandonada, conta hackeada, typosquatting)</li>
<li>
<strong>Scripts de lifecycle</strong> que executam automaticamente durante <code>npm install</code>
</li>
</ol>

<p>O ponto crítico é que <strong>o desenvolvedor não precisa fazer nada além de instalar o pacote</strong>. Não é necessário abrir um arquivo suspeito, clicar em um link ou executar um binário desconhecido. O simples ato de adicionar uma dependência ao <code>package.json</code> e rodar <code>npm install</code> pode ser suficiente para comprometer a máquina, o repositório ou a infraestrutura.</p>


<h2>
  
  
  Scripts de Lifecycle: A Porta de Entrada
</h2>

<p>O npm define diversos scripts que são executados em momentos específicos do ciclo de vida de um pacote:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Script</th>
<th>Momento de execução</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>preinstall</code></td>
<td>Antes do pacote ser instalado</td>
</tr>
<tr>
<td><code>install</code></td>
<td>Durante a instalação</td>
</tr>
<tr>
<td><code>postinstall</code></td>
<td>
<strong>Imediatamente após a instalação</strong> — alvo preferido de ataques</td>
</tr>
<tr>
<td><code>preuninstall</code></td>
<td>Antes de desinstalar</td>
</tr>
<tr>
<td><code>postuninstall</code></td>
<td>Após desinstalar</td>
</tr>
<tr>
<td><code>prepublish</code></td>
<td>Antes de publicar no registro npm</td>
</tr>
</tbody>
</table></div>

<p><strong>Qualquer pessoa que execute <code>npm install</code> rodará esses scripts automaticamente</strong>, sem avisos claros. É aí que reside o perigo.</p>


<h2>
  
  
  Projeto de Demonstração: Estrutura e Código
</h2>

<p>Para ilustrar o vetor de ataque, criamos um projeto educacional composto por:</p>

<ol>
<li>
<strong>Pacote “malicioso”</strong> — um pacote que parece útil, mas executa código nas fases de instalação</li>
<li>
<strong>Projeto vítima</strong> — um projeto que simplesmente depende desse pacote</li>
</ol>
<h3>
  
  
  Estrutura do Pacote Malicioso
</h3>

<p>O <code>package.json</code> do pacote define os scripts que serão executados:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"utilidades-uteis"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Pacote útil que parece legítimo mas executa código no post-install"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"main"</span><span class="p">:</span><span class="w"> </span><span class="s2">"index.js"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"postinstall"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node postinstall.js"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"preinstall"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node preinstall.js"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"keywords"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"utility"</span><span class="p">,</span><span class="w"> </span><span class="s2">"helper"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Atacante Anônimo"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"license"</span><span class="p">:</span><span class="w"> </span><span class="s2">"MIT"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Note que <code>postinstall</code> e <code>preinstall</code> apontam para scripts Node.js. Esses scripts rodam automaticamente durante a instalação.</p>

<h3>
  
  
  Script preinstall.js
</h3>

<p>Este script executa <strong>antes</strong> da instalação do pacote:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="cm">/**
 * PREINSTALL - Roda ANTES da instalação do pacote
 * Outra fase onde código malicioso pode executar
 */</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="se">\</span><span class="s1">u001b[35m[preinstall]</span><span class="se">\</span><span class="s1">u001b[0m Este script roda antes mesmo do pacote ser instalado!</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p>Em um ataque real, aqui poderia haver coleta inicial de dados ou preparação do ambiente.</p>

<h3>
  
  
  Script postinstall.js — O Coração do Ataque
</h3>

<p>Aqui está o script que simula a exfiltração de dados sensíveis:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">fs</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">path</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">os</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">os</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// Simula o que um atacante PODERIA coletar (apenas mostra, não envia)</span>
<span class="kd">const</span> <span class="nx">dadosSensiveis</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">executadoEm</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
  <span class="na">usuario</span><span class="p">:</span> <span class="nx">os</span><span class="p">.</span><span class="nf">userInfo</span><span class="p">().</span><span class="nx">username</span><span class="p">,</span>
  <span class="na">diretorioAtual</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nf">cwd</span><span class="p">(),</span>
  <span class="na">platform</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">platform</span><span class="p">,</span>
  <span class="na">nodeVersion</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">version</span><span class="p">,</span>
  <span class="c1">// Um atacante real tentaria ler:</span>
  <span class="c1">// env: process.env,  // Tokens, senhas, API keys</span>
  <span class="c1">// arquivos: fs.readdirSync(process.env.HOME)</span>
<span class="p">};</span>

<span class="c1">// Cria arquivo de "prova" - em ataque real seria enviado para servidor</span>
<span class="kd">const</span> <span class="nx">arquivoProva</span> <span class="o">=</span> <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nf">cwd</span><span class="p">(),</span> <span class="dl">'</span><span class="s1">PROVA_ATAQUE_SUPPLY_CHAIN.json</span><span class="dl">'</span><span class="p">);</span>
<span class="nx">fs</span><span class="p">.</span><span class="nf">writeFileSync</span><span class="p">(</span><span class="nx">arquivoProva</span><span class="p">,</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">dadosSensiveis</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span> <span class="dl">'</span><span class="s1">utf8</span><span class="dl">'</span><span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`\u001b[31m[ATAQUE SIMULADO]\u001b[0m Dados coletados salvos em: </span><span class="p">${</span><span class="nx">arquivoProva</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="se">\n</span><span class="s1">Em um ataque REAL, isso seria enviado para o servidor do atacante.</span><span class="se">\n</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p>Exemplo de arquivo criado após o script executar</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnlu8u1men5p4vzrl8qr6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnlu8u1men5p4vzrl8qr6.png" alt="exfiltrated information image" width="800" height="203"></a></p>

<p>E ao executar a aplicação o usuário nem percebe o que aconteceu</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3k5rdxd80cd30px0vos6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3k5rdxd80cd30px0vos6.png" alt="Image showing that the user will not see the malware execution" width="800" height="390"></a></p>

<p>Em uma versão maliciosa real, o atacante trocaria <code>fs.writeFileSync</code> por uma chamada HTTP (por exemplo, com <code>https.request</code>) para enviar esses dados a um servidor sob seu controle. O pacote também expõe um módulo legítimo (<code>index.js</code>) que faz algo útil — tornando o pacote plausível e reduzindo a suspeita.</p>

<h3>
  
  
  Fluxo do Ataque
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Desenvolvedor: npm install utilidades-uteis
2. npm baixa o pacote
3. npm executa preinstall  → código malicioso #1
4. npm executa postinstall → código malicioso #2 (coleta dados)
5. Pacote instalado normalmente
6. Desenvolvedor não percebe que foi comprometido
</code></pre>

</div>






<h2>
  
  
  Consequências em Ambientes Reais
</h2>

<p>O que poderia acontecer se esse fosse um ataque real? As consequências variam conforme o contexto da vítima.</p>

<h3>
  
  
  1. Exfiltração de Credenciais e Segredos
</h3>

<ul>
<li>
<strong>Variáveis de ambiente</strong> (<code>process.env</code>): tokens de API (AWS, GitHub, Stripe), chaves de banco de dados, senhas</li>
<li>
<strong>Arquivos <code>.env</code></strong>: credenciais em texto plano em vários projetos</li>
<li>
<strong>Arquivos de configuração</strong>: <code>~/.npmrc</code>, <code>~/.aws/credentials</code>, <code>~/.ssh/config</code>
</li>
</ul>

<p><strong>Impacto</strong>: Acesso a contas cloud, bancos de dados, repositórios privados e sistemas de terceiros.</p>

<h3>
  
  
  2. Roubo de Chaves SSH e Certificados
</h3>

<ul>
<li>Leitura de <code>~/.ssh/</code> (chaves privadas, <code>known_hosts</code>)</li>
<li>Uso das chaves para acessar servidores, GitHub, repositórios privados</li>
</ul>

<p><strong>Impacto</strong>: Invasão de servidores, clonagem de repositórios privados, commit de código malicioso em nome da vítima.</p>

<h3>
  
  
  3. Cryptojacking
</h3>

<ul>
<li>Execução de minerador de criptomoeda em segundo plano</li>
<li>Uso de CPU e energia da máquina ou servidor da vítima</li>
</ul>

<p><strong>Impacto</strong>: Custos elevados de infraestrutura, degradação de desempenho, possível violação de políticas de uso de cloud.</p>

<h3>
  
  
  4. Backdoors e Persistência
</h3>

<ul>
<li>Instalação de ferramentas de acesso remoto</li>
<li>Adição de tarefas agendadas ou scripts de inicialização</li>
</ul>

<p><strong>Impacto</strong>: Controle prolongado da máquina, espionagem, preparação para ataques futuros.</p>

<h3>
  
  
  5. Modificação de Outros Pacotes
</h3>

<ul>
<li>Alteração de código em <code>node_modules</code> de outras dependências</li>
<li>Injeção de backdoors em bibliotecas usadas em produção</li>
</ul>

<p><strong>Impacto</strong>: Comprometimento em escala, propagação do ataque para todos os usuários da aplicação.</p>

<h3>
  
  
  6. Em Ambientes CI/CD
</h3>

<ul>
<li>Acesso a secrets de pipelines (tokens, credenciais)</li>
<li>Possibilidade de modificar artefatos de build ou imagens Docker</li>
<li>Deploy de versões comprometidas em produção</li>
</ul>

<p><strong>Impacto</strong>: Comprometimento de toda a cadeia de entrega, desde o build até produção.</p>




<h2>
  
  
  Casos Reais Documentados
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Caso</th>
<th>Ano</th>
<th>Descrição</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>event-stream</strong></td>
<td>2018</td>
<td>Pacote com ~2M downloads/semana. Código malicioso adicionado para roubar carteiras Bitcoin.</td>
</tr>
<tr>
<td><strong>ua-parser-js</strong></td>
<td>2021</td>
<td>Biblioteca popular comprometida; executava minerador de criptomoeda.</td>
</tr>
<tr>
<td>
<strong>coa</strong> e <strong>rc</strong>
</td>
<td>2021</td>
<td>Typosquatting; pacotes roubavam variáveis de ambiente e as enviavam para servidor remoto.</td>
</tr>
<tr>
<td><strong>node-ipc</strong></td>
<td>2022</td>
<td>Adicionou código que alterava arquivos em máquinas de desenvolvedores de certas regiões geográficas.</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Medidas de Proteção
</h2>

<ol>
<li>
<strong>Use <code>--ignore-scripts</code></strong> quando possível: <code>npm install --ignore-scripts</code>
</li>
<li>
<strong>Audite dependências</strong>: <code>npm audit</code>, <code>npm audit fix</code>
</li>
<li>
<strong>Verifique scripts</strong>: <code>npm view nome-do-pacote</code> antes de instalar</li>
<li>
<strong>Ferramentas especializadas</strong>: Socket.dev, Snyk para detecção de comportamentos suspeitos</li>
<li>
<strong>Mantenha <code>package-lock.json</code></strong> versionado e revise mudanças</li>
<li>
<strong>Verifique procedência</strong>: downloads, manutenção ativa, repositório aberto</li>
</ol>




<h2>
  
  
  Conclusão
</h2>

<p>O ecossistema npm é extremamente conveniente, mas essa conveniência traz riscos. O ato aparentemente trivial de <code>npm install</code> pode executar código arbitrário na sua máquina. A consciência sobre supply chain attacks e a adoção de boas práticas de segurança são fundamentais para reduzir a superfície de ataque e proteger projetos e infraestrutura.</p>

<p>O projeto de demonstração está disponível para que você possa testar o fluxo em um ambiente controlado e entender na prática como esses ataques funcionam.</p>

