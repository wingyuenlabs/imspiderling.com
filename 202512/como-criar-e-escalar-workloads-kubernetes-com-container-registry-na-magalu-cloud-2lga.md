---
Title: Como criar e escalar workloads Kubernetes com Container Registry na Magalu Cloud
Description: 
Author: Monica Hillman
Date: 2025-12-18T21:22:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Imagine um cenário bem comum: seu time mantém uma <strong>API</strong> (por exemplo, <code>checkout-api</code>) e um <strong>worker</strong> de processamento assíncrono (por exemplo, <code>order-worker</code>). Em dias normais, 2 réplicas da API dão conta. Mas em promoções, campanhas ou picos sazonais, o tráfego sobe rápido, e você precisa:</p>

<ul>
<li><p><strong>publicar uma nova versão</strong> da aplicação com segurança (sem subir binário em VM na mão),</p></li>
<li><p><strong>garantir que o cluster puxe a imagem correta</strong> (privada) com controle de acesso,</p></li>
<li><p><strong>aumentar réplicas</strong> rapidamente para absorver carga,</p></li>
<li><p>e manter um fluxo simples para repetir isso toda semana (ou todo dia).</p></li>
</ul>

<p>É exatamente aí que entram <strong>Container Registry</strong> + <strong>Kubernetes</strong>:</p>

<ul>
<li><p>o <strong><a href="https://docs.magalu.cloud/docs/containers-manager/container-registry/overview/?utm_source=devto" rel="noopener noreferrer">Container Registry</a></strong> vira o “repositório oficial” de imagens da sua aplicação (com versionamento e permissão),</p></li>
<li><p>o <strong><a href="https://docs.magalu.cloud/docs/containers-manager/kubernetes/overview/?utm_source=devto" rel="noopener noreferrer">Kubernetes</a></strong> executa essas imagens como workloads (Deployments/Pods) e permite <strong>escalar</strong> o número de réplicas de forma controlada.</p></li>
</ul>

<p>Neste artigo, você vai aprender:</p>

<ul>
<li><p>Como criar e usar o <strong>Container Registry</strong> da <a href="https://magalu.cloud/?utm_source=site&amp;utm_medium=organico&amp;utm_campaign=artigos+comunidade&amp;utm_id=artigos+comunidade" rel="noopener noreferrer">Magalu Cloud</a></p></li>
<li><p>Como autenticar <strong>Docker</strong> e <strong>Kubernetes</strong> no registry</p></li>
<li><p>Como criar workloads Kubernetes usando <strong>imagens privadas</strong></p></li>
<li><p>Como escalar workloads Kubernetes de forma simples</p></li>
</ul>




<h2>
  
  
  Pré-requisitos
</h2>

<ul>
<li>Ter uma conta ativa na Magalu Cloud
</li>
<li>Ter a CLI <code>mgc</code> instalada e configurada com suas credenciais
</li>
<li>Ter um cluster Kubernetes criado na Magalu Cloud
</li>
<li>Ter acesso ao <code>kubectl</code> configurado para o cluster
</li>
<li>Ter o Docker (ou ferramenta compatível) instalado localmente
</li>
</ul>




<h2>
  
  
  <strong>Visão geral da arquitetura</strong>
</h2>

<p>O fluxo básico é o seguinte:</p>

<ol>
<li>Criar um <strong>Container Registry</strong>
</li>
<li>Construir e publicar imagens no registry
</li>
<li>Executar workloads no Kubernetes usando essas imagens
</li>
<li>Escalar a aplicação conforme a necessidade</li>
</ol>

<p>Esse modelo separa claramente:</p>

<ul>
<li>
<strong>build e versionamento</strong> (Container Registry)
</li>
<li>
<strong>execução e escalabilidade</strong> (Kubernetes)</li>
</ul>




<h2>
  
  
  <strong>O que é o Container Registry</strong>
</h2>

<p>O <strong>Container Registry da Magalu Cloud</strong> é o serviço responsável por armazenar e distribuir imagens de container utilizadas pelas aplicações. Ele funciona como um repositório central, compatível com o padrão <strong>OCI/Docker</strong>, onde o time publica as imagens geradas durante o processo de build e a partir do qual outros serviços, como o Kubernetes, conseguem consumi-las.</p>

<p>Na prática, o registry passa a ser a <strong>fonte oficial das imagens da aplicação</strong>. Cada versão gerada pode ser identificada por uma tag, permitindo controlar exatamente qual versão será executada em cada ambiente. Isso facilita o versionamento, o rollback e a padronização do processo de entrega.</p>

<p>Além disso, o Container Registry permite manter <strong>imagens privadas</strong>, garantindo que apenas usuários e serviços autorizados consigam acessá-las. Essa característica é essencial em ambientes de produção, onde o código da aplicação não deve ficar exposto em registries públicos.</p>

<p>Por fim, o serviço é integrado de forma direta ao <strong>Kubernetes da Magalu Cloud</strong>, possibilitando que os clusters façam o <em>pull</em> das imagens privadas de maneira segura e automatizada, sem a necessidade de soluções externas ou configurações complexas.</p>




<h2>
  
  
  <strong>Criando um Container Registry</strong>
</h2>

<p>Antes de autenticar o Docker ou publicar imagens, é necessário <strong>criar um registry</strong>.<br><br>
Esse recurso funciona como um <strong>namespace</strong> para organização das imagens.</p>
<h3>
  
  
  <strong>Criar o registry via CLI</strong>
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc container-registry registries create <span class="nt">--name</span><span class="o">=</span><span class="s2">"meu-registry"</span>
</code></pre>

</div>


<p>Esse comando cria o registry e o associa à sua conta e região.</p>
<h3>
  
  
  <strong>Listar registries existentes</strong>
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc container-registry registries list
</code></pre>

</div>


<p>Se já existir um registry, ele pode ser reutilizado.</p>


<h2>
  
  
  <strong>Endpoints do Container Registry</strong>
</h2>

<p>O endpoint depende da região:</p>

<ul>
<li>
<strong>Brasil – Sudeste (br-se1)</strong>
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>container-registry.br-se1.magalu.cloud
</code></pre>

</div>


<ul>
<li>
<strong>Brasil – Nordeste (br-ne1)</strong>
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>container-registry.br-ne1.magalu.cloud
</code></pre>

</div>


<p>Esse endpoint será usado em:</p>

<ul>
<li>
<code>docker login</code>
</li>
<li>
<code>docker push</code>
</li>
<li>manifests do Kubernetes</li>
</ul>


<h2>
  
  
  <strong>Autenticação no Container Registry</strong>
</h2>

<p>A autenticação utiliza <strong>credenciais específicas do serviço</strong>, distintas da conta principal da Magalu Cloud.</p>


<h3>
  
  
  <strong>1. Listar credenciais do registry</strong>
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mgc container-registry credentials list
</code></pre>

</div>


<p>Esse comando retorna o usuário e a senha/token do Container Registry.</p>


<h3>
  
  
  <strong>2. Autenticar o Docker localmente</strong>
</h3>

<p>Exemplo para <strong>br-se1</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker login https://container-registry.br-se1.magalu.cloud
</code></pre>

</div>



<p>Informe:</p>

<ul>
<li>
<strong>Usuário</strong> → retornado pela CLI
</li>
<li>
<strong>Senha</strong> → token retornado pela CLI</li>
</ul>

<p>Após isso, o Docker estará autenticado para push e pull.</p>




<h2>
  
  
  <strong>Publicando uma imagem no Registry</strong>
</h2>

<p>Fluxo típico de publicação:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Construir a imagem</span>
docker build <span class="nt">-t</span> meu-app:1.0.0 <span class="nb">.</span>
<span class="c"># Marcar a imagem com o registry</span>
docker tag meu-app:1.0.0 <span class="se">\</span>
container-registry.br-se1.magalu.cloud/meu-registry/meu-app:1.0.0
<span class="c"># Enviar a imagem</span>
docker push container-registry.br-se1.magalu.cloud/meu-registry/meu-app:1.0.0
</code></pre>

</div>



<p>Boas práticas:</p>

<ul>
<li>use tags versionadas
</li>
<li>evite sobrescrever <code>latest</code> em produção
</li>
</ul>




<h2>
  
  
  <strong>Kubernetes na Magalu Cloud</strong>
</h2>

<h3>
  
  
  <strong>O que é o Kubernetes gerenciado</strong>
</h3>

<p>O <strong>Kubernetes da Magalu Cloud</strong> é um serviço gerenciado que fornece um <strong>cluster pronto para execução de workloads containerizados</strong>, eliminando grande parte da complexidade operacional envolvida na administração de um ambiente Kubernetes.</p>

<p>Em vez de lidar diretamente com servidores, sistemas operacionais e componentes internos do cluster, o desenvolvedor passa a interagir com o Kubernetes de forma declarativa, por meio de <strong>manifests</strong> que descrevem como a aplicação deve ser executada.</p>

<p>A Magalu Cloud é responsável por gerenciar os principais componentes da plataforma, incluindo:</p>

<ul>
<li><p><strong>Plano de controle</strong>, que engloba serviços como API Server, agendador e controladores do Kubernetes</p></li>
<li><p><strong>Infraestrutura subjacente</strong>, como nós de trabalho, capacidade computacional e disponibilidade do cluster</p></li>
<li><p><strong>Integração com rede</strong>, permitindo que pods e serviços se comuniquem de forma padronizada dentro do ambiente</p></li>
</ul>

<p>Com isso, o time não precisa se preocupar com tarefas como instalação do Kubernetes, atualizações de componentes, configuração inicial de rede ou manutenção do plano de controle.</p>

<p>O foco do desenvolvedor fica restrito ao que realmente importa no dia a dia:</p>

<ul>
<li><p>definir <strong>Deployments</strong>, <strong>Services</strong> e outros recursos via YAML</p></li>
<li><p>versionar e atualizar aplicações com segurança</p></li>
<li><p>escalar workloads conforme a demanda</p></li>
<li><p>integrar o cluster com serviços da própria plataforma, como o Container Registry</p></li>
</ul>

<p>Esse modelo reduz o esforço operacional e torna o uso do Kubernetes mais acessível, especialmente para times que querem adotar containers e orquestração sem assumir a complexidade completa de operar um cluster por conta própria.</p>




<h2>
  
  
  <strong>Autenticação do Kubernetes no Container Registry</strong>
</h2>

<p>Para que o Kubernetes consiga baixar imagens privadas, é necessário criar um <strong>Secret do tipo <code>docker-registry</code></strong>.</p>

<h3>
  
  
  <strong>Criando o Secret</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl create secret docker-registry magalu-registry-secret <span class="se">\-</span><span class="nt">-docker-server</span><span class="o">=</span>container-registry.br-se1.magalu.cloud <span class="se">\ </span><span class="nt">--docker-username</span><span class="o">=</span>&lt;USUARIO_DO_REGISTRY&gt; <span class="se">\ </span><span class="nt">--docker-password</span><span class="o">=</span>&lt;SENHA_DO_REGISTRY&gt; <span class="se">\ </span><span class="nt">--docker-email</span><span class="o">=</span>&lt;SEU_EMAIL&gt;
</code></pre>

</div>






<h2>
  
  
  <strong>Criando um workload Kubernetes</strong>
</h2>

<h3>
  
  
  <strong>Exemplo de Deployment</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>apiVersion: apps/v1
kind: Deployment
metadata:
  name: meu-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: meu-app
  template:
    metadata:
      labels:
        app: meu-app
    spec:
      imagePullSecrets:
        - name: magalu-registry-secret
      containers:
        - name: app
          image: container-registry.br-se1.magalu.cloud/meu-registry/meu-app:1.0.0
          ports:
            - containerPort: 8080
</code></pre>

</div>



<p>Aplicando o manifesto:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> deployment.yaml
</code></pre>

</div>






<h2>
  
  
  <strong>Escalando workloads Kubernetes</strong>
</h2>

<p>Escalonamento manual via CLI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl scale deployment meu-app <span class="nt">--replicas</span><span class="o">=</span>4
</code></pre>

</div>



<p>O Kubernetes cria ou remove pods automaticamente.</p>




<h2>
  
  
  <strong>Atualizando versões da aplicação</strong>
</h2>

<p>Para atualizar a aplicação:</p>

<ol>
<li>Gere uma nova imagem
</li>
<li>Envie ao registry
</li>
<li>Atualize a tag no Deployment
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>image: container-registry.br-se1.magalu.cloud/meu-registry/meu-app:1.1.0
</code></pre>

</div>



<p>Ao aplicar o manifesto, o Kubernetes executa um <strong>rolling update</strong>.</p>




<h2>
  
  
  Conclusão
</h2>

<p>A integração entre <strong>Container Registry</strong> e <strong>Kubernetes</strong> na Magalu Cloud permite criar aplicações containerizadas de forma <strong>segura, escalável e padronizada</strong>.</p>

<p>Com esse modelo, você consegue:</p>

<ul>
<li>Criar e organizar registries privados
</li>
<li>Publicar imagens versionadas
</li>
<li>Executar workloads Kubernetes
</li>
<li>Escalar aplicações conforme a demanda</li>
</ul>

<p>Tudo isso utilizando serviços gerenciados, integrados e operando sobre infraestrutura no Brasil.</p>

