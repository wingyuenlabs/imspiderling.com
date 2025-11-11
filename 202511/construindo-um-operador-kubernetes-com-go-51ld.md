---
Title: Construindo um Operador Kubernetes com Go
Description: 
Author: Gabriel francisco Alves
Date: 2025-11-11T20:54:15.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introdução
</h2>

<p>Se você já administra aplicações em Kubernetes, provavelmente sabe o quanto pode ser trabalhoso manter tudo funcionando corretamente. Criar Deployments, Services, Ingresses, aplicar patches, gerenciar atualizações… a lista não acaba. <br>
Ferramentas como o <strong>Helm</strong> ajudam  a organizar e versionar YAMLs, mas mas não lidam com lógica operacional complexa. Afinal, o Helm não “pensa”: ele '<del>apenas</del>' aplica templates. E se pudéssemos ensinar o Kubernetes a fazer esse trabalho sozinho, reagindo a mudanças e tomando decisões automaticamente?</p>

<p>Os <strong>Operadores do Kubernetes</strong> são como extensões que deixam o cluster mais inteligente. Eles permitem que você descreva, de forma simples, o que precisa, e o operador se encarrega de criar e manter todos os recursos necessários para que isso aconteça. Assim, em vez de depender de scripts ou de aplicar vários YAMLs manualmente, você trabalha com recursos customizados que o próprio Kubernetes entende — ampliando o que o cluster consegue fazer de forma nativa.</p>

<p>Neste post, quero mostrar de forma prática o que os <strong>Operadores do Kubernetes</strong> são capazes de fazer e por que podem ser tão úteis. Para isso, vou usar um exemplo em Go: um operador que implanta uma aplicação simples e já cria Deployment, Service e Ingress automaticamente. Mais do que o código em si, a ideia é demonstrar como operadores pode ser utilizados para transformam tarefas repetitivas em algo muito mais simples, deixando o Kubernetes trabalhar por você.</p>


<h2>
  
  
  O Que Vamos Construir
</h2>

<p>Para ilustrar o poder dos operadores, vamos criar um simples operador nos permitirá aplicar um  <strong>recurso customizado <code>App</code></strong> que automaticamente gera:</p>

<ul>
<li>
<strong>Deployment</strong> - Roda a aplicação (imagem, replicas, porta)</li>
<li>
<strong>Service</strong> - Expõe internamente no cluster</li>
<li>
<strong>Ingress</strong> - Expõe externamente via HTTP</li>
<li>
<strong>Status</strong> - Mostra estado atual (replicas disponíveis, URL)
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">platform.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">App</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">app-sample</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">deploy</span><span class="pi">:</span> 
    <span class="na">image</span><span class="pi">:</span> <span class="s">nginx:1.27</span>
    <span class="na">replicas</span><span class="pi">:</span> <span class="m">2</span>
    <span class="na">containerPort</span><span class="pi">:</span> <span class="m">8080</span>
  <span class="na">service</span><span class="pi">:</span>
    <span class="na">port</span><span class="pi">:</span> <span class="m">9093</span>
  <span class="na">ingress</span><span class="pi">:</span>
    <span class="na">host</span><span class="pi">:</span> <span class="s">minha-app.local</span>
    <span class="na">path</span><span class="pi">:</span> <span class="s">/</span>  

</code></pre>

</div>



<h2>
  
  
  <a id="#oquesaooperadoresnokubernetes"></a> O que são Operadores no Kubernetes?
</h2>

<p>De forma simples: <strong>um Operador é um programa que estende o Kubernetes</strong> para entender novos tipos de recurso.<br>
No dia a dia, usamos objetos nativos como <code>Pod</code>, <code>Service</code> e <code>Deployment</code>. Mas com um operador, podemos criar objetos totalmente novos — por exemplo, <code>Database</code>, <code>App</code> ou <code>BackupJob</code> — e ensinar o Kubernetes a entender o que fazer com eles.<br>
Um operador segue o mesmo ciclo de vida que um SRE ou DevOps humano faria:</p>

<ol>
<li>
<strong>Observar</strong> o cluster (ficar “de olho” nos recursos).</li>
<li>
<strong>Decidir</strong> o que precisa ser feito (comparar estado desejado com estado atual).</li>
<li>
<strong>Agir</strong> criando, atualizando ou removendo objetos.
Dessa forma, ele automatiza operações que antes precisariam ser feitas manualmente.</li>
</ol>

<p>[imagemdofluxodocontroller]</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0nkakah4pgni7h3ap925.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0nkakah4pgni7h3ap925.png" alt="reconciliationflow" width="447" height="627"></a></p>


<h2>
  
  
  Quando faz sentido usar Operadores?
</h2>

<p>Você pode estar se perguntando: “mas se já tenho Helm, por que precisaria de um operador?”.<br>
Aqui estão alguns casos típicos em que operadores fazem toda a diferença:</p>

<ul>
<li>
<strong>Aplicações com lógica operacional complexa</strong>: bancos de dados, caches e sistemas distribuídos geralmente exigem muito mais do que um simples <code>kubectl apply</code>.</li>
<li>
<strong>Automação sob medida</strong>: quando Helm não consegue lidar com a lógica de negócios ou requisitos específicos do seu time.</li>
<li>
<strong>Integrações externas</strong>: operadores podem conversar com APIs, sistemas de monitoramento, provedores de nuvem etc.</li>
<li>
<strong>Menos erro humano</strong>: nada de esquecer de aplicar um YAML ou rodar um script — o operador garante o estado correto.</li>
</ul>

<p>Um exemplo prático:</p>

<ul>
<li>Com Helm, você pode instalar o PostgreSQL.</li>
<li>Com um operador, além de instalar, o Kubernetes pode automaticamente criar backups, restaurar em caso de falha, gerenciar usuários e até escalar o banco.</li>
</ul>


<h2>
  
  
  Operadores vs Helm
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Helm</strong></th>
<th><strong>Operador</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>Trabalha com templates de YAML</td>
<td>Trabalha observando e reagindo a eventos do cluster</td>
</tr>
<tr>
<td>Ótimo para instalar aplicações</td>
<td>Ótimo para operar aplicações</td>
</tr>
<tr>
<td>Não tem lógica de negócio embutida</td>
<td>Pode conter inteligência operacional (backup, failover, integração externa)</td>
</tr>
<tr>
<td>Execução pontual (<code>helm install/upgrade</code>)</td>
<td>Execução contínua (reconciliação automática)</td>
</tr>
</tbody>
</table></div>

<p>Resumindo: Helm é como um “instalador de pacotes”, enquanto operadores são como “administradores automatizados”.</p>


<h2>
  
  
  Mãos à Obra: Criando Nosso Operador
</h2>
<h3>
  
  
  Pré-requisitos
</h3>

<ul>
<li>Go 1.23+</li>
<li>kubebuilder instalado</li>
<li>Cluster Kubernetes(local)
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Instalar Go 1.23+</span>
go version  <span class="c"># Deve mostrar go1.23 ou superior</span>

<span class="c"># 2. Instalar Kubebuilder</span>
curl <span class="nt">-L</span> <span class="nt">-o</span> kubebuilder https://go.kubebuilder.io/dl/latest/<span class="o">{</span>% katex inline %<span class="o">}</span>
 <span class="o">(</span>go <span class="nb">env </span>GOOS<span class="o">)</span>/
<span class="o">{</span>% endkatex %<span class="o">}(</span>go <span class="nb">env </span>GOARCH<span class="o">)</span>
<span class="nb">chmod</span> +x kubebuilder <span class="o">&amp;&amp;</span> <span class="nb">mv </span>kubebuilder /usr/local/bin/

<span class="c"># 3. Cluster Kubernetes local</span>
kind create cluster <span class="nt">--name</span> operator-demo
<span class="c"># OU</span>
minikube start
</code></pre>

</div>

<h3>
  
  
  Estrutura do Projeto
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Criar projeto</span>
kubebuilder init <span class="nt">--domain</span> platform.example.com <span class="nt">--repo</span> github.com/seu-usuario/app-operator

<span class="c"># Criar API</span>
kubebuilder create api <span class="nt">--group</span> platform <span class="nt">--version</span> v1alpha1 <span class="nt">--kind</span> App <span class="nt">--resource</span> <span class="nt">--controller</span>
</code></pre>

</div>


<p><strong>Estrutura gerada:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>app-operator/
├── api/v1alpha1/
│   └── app_types.go          # ← Define o CRD (estrutura do YAML)
├── internal/controller/
│   └── app_controller.go     # ← Lógica do operador (reconciliation)
├── config/
│   ├── crd/                  # ← Manifests do CRD
│   ├── rbac/                 # ← Permissões
│   └── samples/              # ← Exemplos de uso
└── main.go
</code></pre>

</div>



<p>Utilizaremos uma função chamada cmp, para comparar as alterações nos recursos. Então precisamos  trazer essa biblioteca para o nosso projeto, para isso utilizamos.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go get github.com/google/go-cmp/cmp 
</code></pre>

</div>



<h2>
  
  
  Definindo o CRD (Custom Resource Definition)
</h2>

<p>Nosso objetivo é que, ao criar um recurso no Kubernetes como o exemplo  acima, o operador seja capaz de gerar automaticamente os objetos necessários (Deployment, Service e Ingress)<br>
Para que isso aconteça é necessário realizar as alterações necessárias no arquivo de types do projeto gerado pelo kubebuilder.</p>


<h3>
  
  
  Editando o arquivo <code>api/v1alpha1/app_types.go</code>
</h3>

<p>Abra o arquivo e defina as structs:</p>

<p>Dentro dele, vamos criar a struct <code>AppSpec</code>, que conterá os campos definidos no <code>spec</code> do CRD. Esses campos serão utilizados pelo operador para criar os recursos no cluster.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">AppSpec</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Deploy</span>  <span class="n">AppDeploy</span>  <span class="s">`json:"deploy"`</span>
    <span class="n">Service</span> <span class="n">AppService</span> <span class="s">`json:"service"`</span>
    <span class="n">Ingress</span> <span class="n">AppIngress</span> <span class="s">`json:"ingress"`</span>
<span class="p">}</span>

<span class="k">type</span> <span class="n">AppDeploy</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Image</span>         <span class="kt">string</span> <span class="s">`json:"image"`</span>
    <span class="n">Replicas</span>      <span class="o">*</span><span class="kt">int32</span> <span class="s">`json:"replicas,omitempty"`</span>
    <span class="n">ContainerPort</span> <span class="o">*</span><span class="kt">int32</span> <span class="s">`json:"containerPort"`</span>
<span class="p">}</span>

<span class="k">type</span> <span class="n">AppService</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Port</span>       <span class="o">*</span><span class="kt">int32</span> <span class="s">`json:"port"`</span>
<span class="p">}</span>

<span class="k">type</span> <span class="n">AppIngress</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Host</span> <span class="kt">string</span> <span class="s">`json:"host,omitempty"`</span>
    <span class="n">Path</span> <span class="kt">string</span> <span class="s">`json:"path,omitempty"`</span>
<span class="p">}</span>

<span class="k">type</span> <span class="n">AppStatus</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">AvailableReplicas</span>  <span class="kt">int32</span>  <span class="s">`json:"availableReplicas,omitempty"`</span>
    <span class="n">URL</span>                <span class="kt">string</span> <span class="s">`json:"url,omitempty"`</span>
    <span class="n">ObservedGeneration</span> <span class="kt">int64</span>  <span class="s">`json:"observedGeneration,omitempty"`</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Entendendo os Campos
</h3>

<p>Nosso CRD tem três seções principais que definem como a aplicação será implantada:</p>

<p><strong>1. Deploy</strong> - Define o container:</p>

<ul>
<li>
<code>image</code>: Qual imagem Docker usar (ex: <code>nginx:1.27</code>)</li>
<li>
<code>replicas</code>: Quantas cópias rodar (padrão: 1)</li>
<li>
<code>containerPort</code>: Porta que o container expõe (padrão: 8080)</li>
</ul>

<p><strong>2. Service</strong> - Define como expor internamente:</p>

<ul>
<li>
<code>port</code>: Porta do Service no cluster (ex: 9093)</li>
</ul>

<p><strong>3. Ingress</strong> - Define acesso externo:</p>

<ul>
<li>
<code>host</code>: Domínio para acessar (ex: <code>minha-app.local</code>)</li>
<li>
<code>path</code>: Caminho da URL (padrão: <code>/</code>)</li>
</ul>

<p><strong>4. Status</strong> - Estado atual (preenchido automaticamente):</p>

<ul>
<li>
<code>availableReplicas</code> - número de réplicas do Deployment que estão realmente disponíveis no momento.</li>
<li>
<code>url</code> - URL de acesso completa</li>
<li>
<code>observedGeneration</code> - Versão reconciliada</li>
</ul>

<h3>
  
  
  Editando o arquivo  <code>internal/controller/app_controller.go</code>
</h3>

<p>Como estamos criando um operador que ira lidar com objetos nativos do kubernetes tambem precisamos fazer com que ele possa lidar com esse tipo de objeto entao precisamos aplicar as permissões necessárias as clusterrole e clusterrolebinding do kubenetes o  kubebuilder tambem lida com isso de forma nativa pra isso precisamos adicionar algumas tags no arquivo<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// +kubebuilder:rbac:groups=apps,resources=deployments,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups="",resources=services,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=networking.k8s.io,resources=ingresses,verbs=get;list;watch;create;update;patch;delete
</code></pre>

</div>



<h3>
  
  
  Gerando e instalando os artefatos
</h3>

<p>Após realizarmos essas alterações nos arquivos <code>app_types.go</code> e <code>app_controller.go</code>, precisamos executar o comando:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>make manifests
make <span class="nb">install</span>
</code></pre>

</div>



<p>Esse passo é importante porque o <strong>Kubebuilder</strong> vai:</p>

<ul>
<li>Atualizar o <strong>CRD</strong> (CustomResourceDefinition) com os novos campos definidos no <code>AppSpec</code> e <code>AppStatus</code>. </li>
<li>Ajustar as permissões de <strong>RBAC</strong> necessárias para que o operador consiga gerenciar os recursos.</li>
<li>Aplicar o crd e a clusterrole e clusterrolebidinig</li>
</ul>

<p>Assim, garantimos que o cluster Kubernetes já reconheça o novo formato do nosso recurso personalizado.</p>

<h3>
  
  
  Implementando o Controller
</h3>

<p>O próximo passo é editar o arquivo <code>internal/controller/app_controller.go</code><br>
para incluirmos a logica do que queremos que o operador faça.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0yg9n9ruch58840k4gxu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0yg9n9ruch58840k4gxu.png" alt="app_controller.go" width="800" height="389"></a></p>

<p>A função <code>Reconcile</code> é onde toda a mágica acontece. É ela que o Kubernetes chama automaticamente sempre que:</p>

<ul>
<li>Um recurso <code>App</code> é criado, modificado ou deletado</li>
<li>Algum recurso gerenciado pelo operador (Deployment, Service, Ingress) muda</li>
<li>O operador precisa garantir que o estado atual corresponde ao estado desejado</li>
</ul>

<p>Vamos entender o código em blocos:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">Reconcile</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">req</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Request</span><span class="p">)</span> <span class="p">(</span><span class="n">ctrl</span><span class="o">.</span><span class="n">Result</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">logger</span> <span class="o">:=</span> <span class="n">log</span><span class="o">.</span><span class="n">FromContext</span><span class="p">(</span><span class="n">ctx</span><span class="p">)</span>
    <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Starting reconciliation"</span><span class="p">,</span> <span class="s">"app"</span><span class="p">,</span> <span class="n">req</span><span class="o">.</span><span class="n">NamespacedName</span><span class="p">)</span>

    <span class="n">app</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">{}</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">req</span><span class="o">.</span><span class="n">NamespacedName</span><span class="p">,</span> <span class="n">app</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">if</span> <span class="n">apierrors</span><span class="o">.</span><span class="n">IsNotFound</span><span class="p">(</span><span class="n">err</span><span class="p">)</span> <span class="p">{</span>
            <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"App resource not found. Ignoring since object must be deleted"</span><span class="p">)</span>
            <span class="k">return</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Result</span><span class="p">{},</span> <span class="no">nil</span>
        <span class="p">}</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Error</span><span class="p">(</span><span class="n">err</span><span class="p">,</span> <span class="s">"Failed to get App resource"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Result</span><span class="p">{},</span> <span class="n">err</span>
    <span class="p">}</span>

</code></pre>

</div>



<p><strong>O que está acontecendo:</strong></p>

<ul>
<li>Cria um logger para registrar as operações</li>
<li>Tenta buscar o recurso <code>App</code> no cluster usando o nome e namespace recebidos</li>
<li>Se o recurso não for encontrado (foi deletado), retorna sem erro</li>
<li>Se houver outro tipo de erro, retorna o erro para tentar novamente</li>
</ul>

<p>Para manter o código mais organizado, vamos dividi-lo em funcoes 4 funções de suporte para tarefas distintas: uma será responsável por reconciliar uma parte da nossa app.</p>

<ul>
<li> Uma função com o nome  reconcileDeployment

<ul>
<li>Responsável por reconciliar o nosso deployment</li>
</ul>


</li>

<li> Uma função com o nome  reconcileService

<ul>
<li>Responsável por reconciliar o nosso service</li>
</ul>


</li>

<li> Uma função com o nome  reconcileIngress

<ul>
<li>Responsável por reconciliar o nosso o ingress </li>
</ul>


</li>

<li> Uma função com o nome  updateStatus 

<ul>
<li> Atualizar o Status do objeto</li>
</ul>


</li>

</ul>

<p>dessa maneira<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">reconcileDeployment</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">app</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Error</span><span class="p">(</span><span class="n">err</span><span class="p">,</span> <span class="s">"Failed to reconcile Deployment"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Result</span><span class="p">{},</span> <span class="n">err</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">reconcileService</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">app</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Error</span><span class="p">(</span><span class="n">err</span><span class="p">,</span> <span class="s">"Failed to reconcile Service"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Result</span><span class="p">{},</span> <span class="n">err</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">reconcileIngress</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">app</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Error</span><span class="p">(</span><span class="n">err</span><span class="p">,</span> <span class="s">"Failed to reconcile Ingress"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Result</span><span class="p">{},</span> <span class="n">err</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">updateStatus</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">app</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Error</span><span class="p">(</span><span class="n">err</span><span class="p">,</span> <span class="s">"Failed to update App status"</span><span class="p">)</span>
    <span class="p">}</span>

</code></pre>

</div>






<h3>
  
  
  Funções de Reconciliação
</h3>

<h4>
  
  
  Função <code>reconcileDeployment()</code> - Gerencia o Deployment
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">reconcileDeployment</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
    <span class="n">logger</span> <span class="o">:=</span> <span class="n">log</span><span class="o">.</span><span class="n">FromContext</span><span class="p">(</span><span class="n">ctx</span><span class="p">)</span>
    <span class="n">deployment</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">appsv1</span><span class="o">.</span><span class="n">Deployment</span><span class="p">{}</span>
    <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">types</span><span class="o">.</span><span class="n">NamespacedName</span><span class="p">{</span>
        <span class="n">Name</span><span class="o">:</span>      <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">,</span>
        <span class="n">Namespace</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
    <span class="p">},</span> <span class="n">deployment</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="o">&amp;&amp;</span> <span class="n">apierrors</span><span class="o">.</span><span class="n">IsNotFound</span><span class="p">(</span><span class="n">err</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">desiredDeployment</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">deploymentForApp</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to generate deployment spec: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Creating new Deployment"</span><span class="p">,</span>
            <span class="s">"namespace"</span><span class="p">,</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="s">"name"</span><span class="p">,</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Create</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">desiredDeployment</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to create deployment: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="no">nil</span>
    <span class="p">}</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to get deployment: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="n">desiredDeployment</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">deploymentForApp</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to generate deployment spec: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="k">if</span> <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">deployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Image</span><span class="p">,</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Image</span><span class="p">)</span> <span class="o">||</span>
        <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">deployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Replicas</span><span class="p">,</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Replicas</span><span class="p">)</span> <span class="o">||</span>
        <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">deployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Ports</span><span class="p">,</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Ports</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Updating existing Deployment"</span><span class="p">,</span>
            <span class="s">"namespace"</span><span class="p">,</span> <span class="n">deployment</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="s">"name"</span><span class="p">,</span> <span class="n">deployment</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>
        <span class="n">deployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Image</span> <span class="o">=</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Image</span>
        <span class="n">deployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Ports</span> <span class="o">=</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Template</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Containers</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Ports</span>
        <span class="n">deployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Replicas</span> <span class="o">=</span> <span class="n">desiredDeployment</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Replicas</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Update</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">deployment</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to update deployment: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="no">nil</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>O que essa função faz:</strong></p>

<ul>
<li>
<strong>Busca</strong> o Deployment no cluster</li>
<li> <strong>Se não existe:</strong> Cria usando <code>deploymentForApp()</code>
</li>
<li> <strong>Se existe:</strong> Compara com estado desejado</li>
<li>
<strong>Se diferente:</strong> Atualiza imagem, replicas ou portas</li>
<li> <strong>Se houver erro na criação</strong> , registra e retorna o erro se não cria o objeto e retorna sem erro</li>
</ul>

<p>Foi criada uma função de suporte chamada <code>deploymentForApp()</code> (veja detalhes na seção Funções Auxiliares)  onde passamos o  a  app e ela nos retorna um objeto do tipo deployment de Kubernetes. </p>

<p>Padrão das funções reconcile: As funções reconcileService() e reconcileIngress() seguem a mesma lógica:<br>
Buscar → Se não existe, criar → Se existe, comparar → Se diferente, atualizar<br>
Veja o código completo dessas funções na seção Funções de Reconciliação.</p>
<h4>
  
  
  Função  <code>reconcileService()</code> - Gerencia o Service
</h4>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">reconcileService</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
    <span class="n">logger</span> <span class="o">:=</span> <span class="n">log</span><span class="o">.</span><span class="n">FromContext</span><span class="p">(</span><span class="n">ctx</span><span class="p">)</span>

    <span class="n">serviceName</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"service-%s"</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

    <span class="n">service</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">corev1</span><span class="o">.</span><span class="n">Service</span><span class="p">{}</span>
    <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">types</span><span class="o">.</span><span class="n">NamespacedName</span><span class="p">{</span>
        <span class="n">Name</span><span class="o">:</span>      <span class="n">serviceName</span><span class="p">,</span>
        <span class="n">Namespace</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
    <span class="p">},</span> <span class="n">service</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="o">&amp;&amp;</span> <span class="n">apierrors</span><span class="o">.</span><span class="n">IsNotFound</span><span class="p">(</span><span class="n">err</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">desiredService</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">serviceForApp</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to generate service spec: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Creating new Service"</span><span class="p">,</span>
            <span class="s">"namespace"</span><span class="p">,</span> <span class="n">desiredService</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="s">"name"</span><span class="p">,</span> <span class="n">desiredService</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Create</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">desiredService</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to create service: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="no">nil</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to get service: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="n">desiredService</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">serviceForApp</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to generate service spec: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">service</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Ports</span><span class="p">,</span> <span class="n">desiredService</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Ports</span><span class="p">)</span> <span class="o">||</span>
        <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">service</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Selector</span><span class="p">,</span> <span class="n">desiredService</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Selector</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Updating existing Service"</span><span class="p">,</span>
            <span class="s">"namespace"</span><span class="p">,</span> <span class="n">service</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="s">"name"</span><span class="p">,</span> <span class="n">service</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

        <span class="n">desiredService</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">ClusterIP</span> <span class="o">=</span> <span class="n">service</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">ClusterIP</span>
        <span class="n">service</span><span class="o">.</span><span class="n">Spec</span> <span class="o">=</span> <span class="n">desiredService</span><span class="o">.</span><span class="n">Spec</span>

        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Update</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">service</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to update service: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="no">nil</span>
<span class="p">}</span>
</code></pre>

</div>

<h4>
  
  
  Função <code>reconcileIngress()</code> - Gerencia o Ingress
</h4>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">reconcileIngress</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
    <span class="n">logger</span> <span class="o">:=</span> <span class="n">log</span><span class="o">.</span><span class="n">FromContext</span><span class="p">(</span><span class="n">ctx</span><span class="p">)</span>

    <span class="n">ingressName</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"ingress-%s"</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

    <span class="n">ingress</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">Ingress</span><span class="p">{}</span>
    <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">types</span><span class="o">.</span><span class="n">NamespacedName</span><span class="p">{</span>
        <span class="n">Name</span><span class="o">:</span>      <span class="n">ingressName</span><span class="p">,</span>
        <span class="n">Namespace</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
    <span class="p">},</span> <span class="n">ingress</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="o">&amp;&amp;</span> <span class="n">apierrors</span><span class="o">.</span><span class="n">IsNotFound</span><span class="p">(</span><span class="n">err</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">desiredIngress</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">ingressForApp</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to generate ingress spec: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Creating new Ingress"</span><span class="p">,</span>
            <span class="s">"namespace"</span><span class="p">,</span> <span class="n">desiredIngress</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="s">"name"</span><span class="p">,</span> <span class="n">desiredIngress</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Create</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">desiredIngress</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to create ingress: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="no">nil</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to get ingress: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="n">desiredIngress</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">ingressForApp</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to generate ingress spec: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="k">if</span> <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">ingress</span><span class="o">.</span><span class="n">Spec</span><span class="p">,</span> <span class="n">desiredIngress</span><span class="o">.</span><span class="n">Spec</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Updating existing Ingress"</span><span class="p">,</span>
            <span class="s">"namespace"</span><span class="p">,</span> <span class="n">ingress</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="s">"name"</span><span class="p">,</span> <span class="n">ingress</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

        <span class="n">ingress</span><span class="o">.</span><span class="n">Spec</span> <span class="o">=</span> <span class="n">desiredIngress</span><span class="o">.</span><span class="n">Spec</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Update</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">ingress</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to update ingress: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="no">nil</span>
<span class="p">}</span>
</code></pre>

</div>

<h4>
  
  
  Função <code>updateStatus()</code> - Atualiza o Status do App
</h4>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">updateStatus</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
    <span class="n">logger</span> <span class="o">:=</span> <span class="n">log</span><span class="o">.</span><span class="n">FromContext</span><span class="p">(</span><span class="n">ctx</span><span class="p">)</span>

    <span class="n">deployment</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">appsv1</span><span class="o">.</span><span class="n">Deployment</span><span class="p">{}</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">types</span><span class="o">.</span><span class="n">NamespacedName</span><span class="p">{</span>
        <span class="n">Name</span><span class="o">:</span>      <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">,</span>
        <span class="n">Namespace</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
    <span class="p">},</span> <span class="n">deployment</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to get deployment for status update: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="n">url</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"http://%s%s"</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Ingress</span><span class="o">.</span><span class="n">Host</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Ingress</span><span class="o">.</span><span class="n">Path</span><span class="p">)</span>

    <span class="n">statusChanged</span> <span class="o">:=</span> <span class="no">false</span>
    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">AvailableReplicas</span> <span class="o">!=</span> <span class="n">deployment</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">AvailableReplicas</span> <span class="p">{</span>
        <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">AvailableReplicas</span> <span class="o">=</span> <span class="n">deployment</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">AvailableReplicas</span>
        <span class="n">statusChanged</span> <span class="o">=</span> <span class="no">true</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">URL</span> <span class="o">!=</span> <span class="n">url</span> <span class="p">{</span>
        <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">URL</span> <span class="o">=</span> <span class="n">url</span>
        <span class="n">statusChanged</span> <span class="o">=</span> <span class="no">true</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">ObservedGeneration</span> <span class="o">!=</span> <span class="n">app</span><span class="o">.</span><span class="n">Generation</span> <span class="p">{</span>
        <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">ObservedGeneration</span> <span class="o">=</span> <span class="n">app</span><span class="o">.</span><span class="n">Generation</span>
        <span class="n">statusChanged</span> <span class="o">=</span> <span class="no">true</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">statusChanged</span> <span class="p">{</span>
        <span class="n">logger</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"Updating App status"</span><span class="p">,</span>
            <span class="s">"availableReplicas"</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">AvailableReplicas</span><span class="p">,</span>
            <span class="s">"url"</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Status</span><span class="o">.</span><span class="n">URL</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Status</span><span class="p">()</span><span class="o">.</span><span class="n">Update</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">app</span><span class="p">);</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"failed to update status: %w"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="no">nil</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Funções Auxiliares
</h3>

<p>A função getLabelForApp foi criada pra facilitar a adição de labels nos objetos criados pelo controller, ela é utilizada em todos os objetos. </p>

<p>Todas as funções auxiliares (<code>deploymentForApp</code>, <code>serviceForApp</code>, <code>ingressForApp</code>) usam <code>ctrl.SetControllerReference()</code>. </p>

<p><strong>O que isso faz?</strong></p>

<ul>
<li>Cria um vínculo de <strong>propriedade</strong> entre o recurso <code>App</code> e os objetos criados (Deployment, Service, Ingress)</li>
<li>Quando o <code>App</code> é deletado, o <strong>Garbage Collector do Kubernetes</strong> remove automaticamente todos os recursos associados</li>
<li>Isso evita "recursos órfãos" no cluster</li>
</ul>

<p><strong>Sem isso:</strong> Você teria que deletar manualmente cada Deployment, Service e Ingress.<br>
<strong>Com isso:</strong> Um único <code>kubectl delete app app-sample</code> limpa tudo!</p>
<h4>
  
  
  <code>getLabelsForApp()</code> - Labels Padronizadas
</h4>

<p>Retorna um mapa/dicionario com as labels definidas<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="n">getLabelsForApp</span><span class="p">(</span><span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="kt">string</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="kt">string</span><span class="p">{</span>
        <span class="s">"app.kubernetes.io/name"</span><span class="o">:</span>       <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">,</span>
        <span class="s">"app.kubernetes.io/managed-by"</span><span class="o">:</span> <span class="s">"platform-app-operator"</span><span class="p">,</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<h4>
  
  
  <code>deploymentForApp()</code> - Constrói o Deployment()
</h4>

<p>Ao iniciar essa função ela realiza algumas validações  como: </p>

<ul>
<li>Se a porta foi passada na spec da app, senão ele define a porta padrao para o container (8080) </li>
<li>Se o numero de replicas foram definidas.  senão ele define a quantidade minima de replicas para 1</li>
</ul>

<p>Depois inicia-se a criação do deployment, utilizando como nome e namespace do  deploy os mesmo das app e para as labels utilizamos a <code>getLabelsForApp()</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// deploymentForApp returns a App Deployment object</span>
<span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">deploymentForApp</span><span class="p">(</span><span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="p">(</span><span class="o">*</span><span class="n">appsv1</span><span class="o">.</span><span class="n">Deployment</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>

    <span class="n">replicas</span> <span class="o">:=</span> <span class="kt">int32</span><span class="p">(</span><span class="m">1</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">Replicas</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">replicas</span> <span class="o">=</span> <span class="o">*</span><span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">Replicas</span>
    <span class="p">}</span>

    <span class="n">port</span> <span class="o">:=</span> <span class="kt">int32</span><span class="p">(</span><span class="m">8080</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">ContainerPort</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">port</span> <span class="o">=</span> <span class="o">*</span><span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">ContainerPort</span>
    <span class="p">}</span>

    <span class="n">dep</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">appsv1</span><span class="o">.</span><span class="n">Deployment</span><span class="p">{</span>
        <span class="n">ObjectMeta</span><span class="o">:</span> <span class="n">metav1</span><span class="o">.</span><span class="n">ObjectMeta</span><span class="p">{</span>
            <span class="n">Name</span><span class="o">:</span>      <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">,</span>
            <span class="n">Namespace</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="n">Labels</span><span class="o">:</span>    <span class="n">getLabelsForApp</span><span class="p">(</span><span class="n">app</span><span class="p">),</span>
        <span class="p">},</span>
        <span class="n">Spec</span><span class="o">:</span> <span class="n">appsv1</span><span class="o">.</span><span class="n">DeploymentSpec</span><span class="p">{</span>
            <span class="n">Replicas</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">replicas</span><span class="p">,</span>
            <span class="n">Selector</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">metav1</span><span class="o">.</span><span class="n">LabelSelector</span><span class="p">{</span>
                <span class="n">MatchLabels</span><span class="o">:</span> <span class="n">getLabelsForApp</span><span class="p">(</span><span class="n">app</span><span class="p">),</span>
            <span class="p">},</span>
            <span class="n">Template</span><span class="o">:</span> <span class="n">corev1</span><span class="o">.</span><span class="n">PodTemplateSpec</span><span class="p">{</span>
                <span class="n">ObjectMeta</span><span class="o">:</span> <span class="n">metav1</span><span class="o">.</span><span class="n">ObjectMeta</span><span class="p">{</span>
                    <span class="n">Labels</span><span class="o">:</span> <span class="n">getLabelsForApp</span><span class="p">(</span><span class="n">app</span><span class="p">),</span>
                <span class="p">},</span>
                <span class="n">Spec</span><span class="o">:</span> <span class="n">corev1</span><span class="o">.</span><span class="n">PodSpec</span><span class="p">{</span>
                    <span class="n">Containers</span><span class="o">:</span> <span class="p">[]</span><span class="n">corev1</span><span class="o">.</span><span class="n">Container</span><span class="p">{{</span>
                        <span class="n">Name</span><span class="o">:</span>  <span class="s">"app"</span><span class="p">,</span>
                        <span class="n">Image</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">Image</span><span class="p">,</span>
                        <span class="n">Ports</span><span class="o">:</span> <span class="p">[]</span><span class="n">corev1</span><span class="o">.</span><span class="n">ContainerPort</span><span class="p">{</span>
                            <span class="p">{</span>
                                <span class="n">Name</span><span class="o">:</span>          <span class="s">"http"</span><span class="p">,</span>
                                <span class="n">ContainerPort</span><span class="o">:</span> <span class="n">port</span><span class="p">,</span>
                                <span class="n">Protocol</span><span class="o">:</span>      <span class="n">corev1</span><span class="o">.</span><span class="n">ProtocolTCP</span><span class="p">,</span>
                            <span class="p">},</span>
                        <span class="p">},</span>
                    <span class="p">},</span>
                    <span class="p">},</span>
                <span class="p">},</span>
            <span class="p">},</span>
        <span class="p">},</span>
    <span class="p">}</span>
    <span class="n">err</span> <span class="o">:=</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">SetControllerReference</span><span class="p">(</span><span class="n">app</span><span class="p">,</span> <span class="n">dep</span><span class="p">,</span> <span class="n">r</span><span class="o">.</span><span class="n">Scheme</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="no">nil</span><span class="p">,</span> <span class="n">err</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="n">dep</span><span class="p">,</span> <span class="no">nil</span>
<span class="p">}</span>


</code></pre>

</div>



<h4>
  
  
  <code>serviceForApp()</code> - Constrói o Service
</h4>

<p>Ao iniciar essa função ela realiza algumas validações  como: </p>

<ul>
<li>Se a porta foi passada na spec da app, senão ele define a porta padrao para o container (8080) </li>
</ul>

<p>Depois inicia-se a criação do service, utilizando como nome e namespace do service os mesmos da app e para as labels utilizamos a função <code>getLabelsForApp()</code>.</p>

<p>essa funcão retorna as labels para os objetos criados pelo controller.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// serviceForApp returns a App service object</span>
<span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">serviceForApp</span><span class="p">(</span><span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="p">(</span><span class="o">*</span><span class="n">corev1</span><span class="o">.</span><span class="n">Service</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>

    <span class="n">serviceName</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"service-%s"</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>

    <span class="n">port</span> <span class="o">:=</span> <span class="kt">int32</span><span class="p">(</span><span class="m">8080</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Service</span><span class="o">.</span><span class="n">Port</span> <span class="o">==</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">port</span> <span class="o">=</span> <span class="o">*</span><span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">ContainerPort</span>
    <span class="p">}</span>
    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Service</span><span class="o">.</span><span class="n">Port</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">port</span> <span class="o">=</span> <span class="o">*</span><span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Service</span><span class="o">.</span><span class="n">Port</span>
    <span class="p">}</span>
    <span class="n">targetPort</span> <span class="o">:=</span> <span class="kt">int32</span><span class="p">(</span><span class="m">8080</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">ContainerPort</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">targetPort</span> <span class="o">=</span> <span class="o">*</span><span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Deploy</span><span class="o">.</span><span class="n">ContainerPort</span>
    <span class="p">}</span>

    <span class="n">svc</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">corev1</span><span class="o">.</span><span class="n">Service</span><span class="p">{</span>
        <span class="n">ObjectMeta</span><span class="o">:</span> <span class="n">metav1</span><span class="o">.</span><span class="n">ObjectMeta</span><span class="p">{</span>
            <span class="n">Name</span><span class="o">:</span>      <span class="n">serviceName</span><span class="p">,</span>
            <span class="n">Namespace</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="n">Labels</span><span class="o">:</span>    <span class="n">getLabelsForApp</span><span class="p">(</span><span class="n">app</span><span class="p">),</span>
        <span class="p">},</span>
        <span class="n">Spec</span><span class="o">:</span> <span class="n">corev1</span><span class="o">.</span><span class="n">ServiceSpec</span><span class="p">{</span>
            <span class="n">Ports</span><span class="o">:</span> <span class="p">[]</span><span class="n">corev1</span><span class="o">.</span><span class="n">ServicePort</span><span class="p">{</span>
                <span class="p">{</span>
                    <span class="n">Name</span><span class="o">:</span>       <span class="s">"app"</span><span class="p">,</span>
                    <span class="n">Port</span><span class="o">:</span>       <span class="n">port</span><span class="p">,</span>
                    <span class="n">TargetPort</span><span class="o">:</span> <span class="n">intstr</span><span class="o">.</span><span class="n">FromInt32</span><span class="p">(</span><span class="n">targetPort</span><span class="p">),</span>
                    <span class="n">Protocol</span><span class="o">:</span>   <span class="s">"TCP"</span><span class="p">,</span>
                <span class="p">},</span>
            <span class="p">},</span>
            <span class="n">Selector</span><span class="o">:</span> <span class="n">getLabelsForApp</span><span class="p">(</span><span class="n">app</span><span class="p">),</span>
        <span class="p">},</span>
    <span class="p">}</span>
    <span class="n">err</span> <span class="o">:=</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">SetControllerReference</span><span class="p">(</span><span class="n">app</span><span class="p">,</span> <span class="n">svc</span><span class="p">,</span> <span class="n">r</span><span class="o">.</span><span class="n">Scheme</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="no">nil</span><span class="p">,</span> <span class="n">err</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="n">svc</span><span class="p">,</span> <span class="no">nil</span>
<span class="p">}</span>

</code></pre>

</div>



<h4>
  
  
  <code>ingressForApp()</code> - Constrói o Ingress
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">ingressForApp</span><span class="p">(</span><span class="n">app</span> <span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="p">(</span><span class="o">*</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">Ingress</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>

    <span class="n">ingressName</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"ingress-%s"</span><span class="p">,</span> <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>


    <span class="n">pathTypeImplementationSpecific</span> <span class="o">:=</span> <span class="n">networkingv1</span><span class="o">.</span><span class="n">PathTypeImplementationSpecific</span>
    <span class="n">path</span> <span class="o">:=</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Ingress</span><span class="o">.</span><span class="n">Path</span>
    <span class="k">if</span> <span class="n">path</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
        <span class="n">path</span> <span class="o">=</span> <span class="s">"/"</span>
    <span class="p">}</span>
    <span class="n">ingress</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">Ingress</span><span class="p">{</span>
        <span class="n">ObjectMeta</span><span class="o">:</span> <span class="n">metav1</span><span class="o">.</span><span class="n">ObjectMeta</span><span class="p">{</span>
            <span class="n">Name</span><span class="o">:</span>      <span class="n">ingressName</span><span class="p">,</span>
            <span class="n">Namespace</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Namespace</span><span class="p">,</span>
            <span class="n">Labels</span><span class="o">:</span>    <span class="n">getLabelsForApp</span><span class="p">(</span><span class="n">app</span><span class="p">),</span>
        <span class="p">},</span>
        <span class="n">Spec</span><span class="o">:</span> <span class="n">networkingv1</span><span class="o">.</span><span class="n">IngressSpec</span><span class="p">{</span>
            <span class="n">Rules</span><span class="o">:</span> <span class="p">[]</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">IngressRule</span><span class="p">{</span>
                <span class="n">networkingv1</span><span class="o">.</span><span class="n">IngressRule</span><span class="p">{</span>
                    <span class="n">Host</span><span class="o">:</span> <span class="n">app</span><span class="o">.</span><span class="n">Spec</span><span class="o">.</span><span class="n">Ingress</span><span class="o">.</span><span class="n">Host</span><span class="p">,</span>
                    <span class="n">IngressRuleValue</span><span class="o">:</span> <span class="n">networkingv1</span><span class="o">.</span><span class="n">IngressRuleValue</span><span class="p">{</span>
                        <span class="n">HTTP</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">HTTPIngressRuleValue</span><span class="p">{</span>
                            <span class="n">Paths</span><span class="o">:</span> <span class="p">[]</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">HTTPIngressPath</span><span class="p">{</span>
                                <span class="n">networkingv1</span><span class="o">.</span><span class="n">HTTPIngressPath</span><span class="p">{</span>
                                    <span class="n">Path</span><span class="o">:</span>     <span class="n">path</span><span class="p">,</span>
                                    <span class="n">PathType</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">pathTypeImplementationSpecific</span><span class="p">,</span>
                                    <span class="n">Backend</span><span class="o">:</span> <span class="n">networkingv1</span><span class="o">.</span><span class="n">IngressBackend</span><span class="p">{</span>
                                        <span class="n">Service</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">IngressServiceBackend</span><span class="p">{</span>
                                            <span class="n">Name</span><span class="o">:</span> <span class="s">"service-"</span> <span class="o">+</span> <span class="n">app</span><span class="o">.</span><span class="n">Name</span><span class="p">,</span>
                                            <span class="n">Port</span><span class="o">:</span> <span class="n">networkingv1</span><span class="o">.</span><span class="n">ServiceBackendPort</span><span class="p">{</span>
                                                <span class="n">Name</span><span class="o">:</span> <span class="s">"app"</span><span class="p">,</span>
                                            <span class="p">},</span>
                                        <span class="p">},</span>
                                    <span class="p">},</span>
                                <span class="p">},</span>
                            <span class="p">},</span>
                        <span class="p">},</span>
                    <span class="p">},</span>
                <span class="p">},</span>
            <span class="p">},</span>
        <span class="p">},</span>
    <span class="p">}</span>
    <span class="n">err</span> <span class="o">:=</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">SetControllerReference</span><span class="p">(</span><span class="n">app</span><span class="p">,</span> <span class="n">ingress</span><span class="p">,</span> <span class="n">r</span><span class="o">.</span><span class="n">Scheme</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="no">nil</span><span class="p">,</span> <span class="n">err</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="n">ingress</span><span class="p">,</span> <span class="no">nil</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  Função SetupWithManager
</h3>

<p>O que a função <code>SetupWithManager</code> faz: ela registra um controller no cluster de Kubernetes e também é responsável por configurar qual recurso ele vai monitorar, quando ele vai reagir e quais recursos ele gerencia.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// SetupWithManager sets up the controller with the Manager.</span>
<span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">AppReconciler</span><span class="p">)</span> <span class="n">SetupWithManager</span><span class="p">(</span><span class="n">mgr</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Manager</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
    <span class="k">return</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">NewControllerManagedBy</span><span class="p">(</span><span class="n">mgr</span><span class="p">)</span><span class="o">.</span>
        <span class="n">For</span><span class="p">(</span><span class="o">&amp;</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">{},</span> <span class="n">builder</span><span class="o">.</span><span class="n">WithPredicates</span><span class="p">(</span><span class="n">predicate</span><span class="o">.</span><span class="n">Funcs</span><span class="p">{</span>
            <span class="n">UpdateFunc</span><span class="o">:</span> <span class="k">func</span><span class="p">(</span><span class="n">e</span> <span class="n">event</span><span class="o">.</span><span class="n">TypedUpdateEvent</span><span class="p">[</span><span class="n">client</span><span class="o">.</span><span class="n">Object</span><span class="p">])</span> <span class="kt">bool</span> <span class="p">{</span>
                <span class="n">log</span> <span class="o">:=</span> <span class="n">ctrl</span><span class="o">.</span><span class="n">Log</span><span class="o">.</span><span class="n">WithName</span><span class="p">(</span><span class="s">"Predicate Update"</span><span class="p">)</span>
                <span class="n">appOld</span><span class="p">,</span> <span class="n">oldok</span> <span class="o">:=</span> <span class="n">e</span><span class="o">.</span><span class="n">ObjectOld</span><span class="o">.</span><span class="p">(</span><span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span>
                <span class="n">appNew</span><span class="p">,</span> <span class="n">newok</span> <span class="o">:=</span> <span class="n">e</span><span class="o">.</span><span class="n">ObjectNew</span><span class="o">.</span><span class="p">(</span><span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span>
                <span class="k">if</span> <span class="o">!</span><span class="n">oldok</span> <span class="o">||</span> <span class="o">!</span><span class="n">newok</span> <span class="p">{</span>
                    <span class="k">return</span> <span class="no">false</span>
                <span class="p">}</span>
                <span class="n">specsChanged</span> <span class="o">:=</span> <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">appOld</span><span class="o">.</span><span class="n">Spec</span><span class="p">,</span> <span class="n">appNew</span><span class="o">.</span><span class="n">Spec</span><span class="p">)</span>
                <span class="k">if</span> <span class="n">specsChanged</span> <span class="p">{</span>
                    <span class="n">diffSpecs</span> <span class="o">:=</span> <span class="n">cmp</span><span class="o">.</span><span class="n">Diff</span><span class="p">(</span><span class="n">appOld</span><span class="o">.</span><span class="n">Spec</span><span class="p">,</span> <span class="n">appNew</span><span class="o">.</span><span class="n">Spec</span><span class="p">)</span>
                    <span class="n">log</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"platform specs update"</span><span class="p">,</span> <span class="s">"app"</span><span class="p">,</span> <span class="n">appNew</span><span class="o">.</span><span class="n">Name</span><span class="p">,</span> <span class="s">"specs"</span><span class="p">,</span> <span class="n">diffSpecs</span><span class="p">)</span>
                <span class="p">}</span>
                <span class="k">return</span> <span class="n">specsChanged</span>
            <span class="p">},</span>
        <span class="p">}))</span><span class="o">.</span>
        <span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">appsv1</span><span class="o">.</span><span class="n">Deployment</span><span class="p">{})</span><span class="o">.</span>
        <span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">corev1</span><span class="o">.</span><span class="n">Service</span><span class="p">{})</span><span class="o">.</span>
        <span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">Ingress</span><span class="p">{})</span><span class="o">.</span>
        <span class="n">Named</span><span class="p">(</span><span class="s">"app"</span><span class="p">)</span><span class="o">.</span>
        <span class="n">Complete</span><span class="p">(</span><span class="n">r</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  Evitando Reconciliações Desnecessárias
</h4>

<p>Para evitar reconciliações desnecessárias, o predicate <code>UpdateFunc</code> faz com que o controller só processe alterações no campo <code>.spec</code> do recurso <code>App</code>.</p>

<p><strong>Como funciona:</strong></p>

<ol>
<li>
<strong>Captura o evento de update</strong>: Quando um objeto <code>App</code> é atualizado no cluster, o predicate recebe o estado antigo (<code>appOld</code>) e o novo (<code>appNew</code>)</li>
<li>
<strong>Valida os objetos</strong>: Faz o type assertion para garantir que são objetos do tipo <code>App</code> válidos
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>   <span class="n">appOld</span><span class="p">,</span> <span class="n">oldok</span> <span class="o">:=</span> <span class="n">e</span><span class="o">.</span><span class="n">ObjectOld</span><span class="o">.</span><span class="p">(</span><span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span>
   <span class="n">appNew</span><span class="p">,</span> <span class="n">newok</span> <span class="o">:=</span> <span class="n">e</span><span class="o">.</span><span class="n">ObjectNew</span><span class="o">.</span><span class="p">(</span><span class="o">*</span><span class="n">platformv1alpha1</span><span class="o">.</span><span class="n">App</span><span class="p">)</span>
   <span class="k">if</span> <span class="o">!</span><span class="n">oldok</span> <span class="o">||</span> <span class="o">!</span><span class="n">newok</span> <span class="p">{</span>
       <span class="k">return</span> <span class="no">false</span>  <span class="c">// Se inválido, ignora o evento</span>
   <span class="p">}</span>
</code></pre>

</div>



<ol>
<li>
<strong>Compara apenas o Spec</strong>: Usa a biblioteca <code>cmp</code> para comparar a spec atual com a anterior
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>   <span class="n">specsChanged</span> <span class="o">:=</span> <span class="o">!</span><span class="n">cmp</span><span class="o">.</span><span class="n">Equal</span><span class="p">(</span><span class="n">appOld</span><span class="o">.</span><span class="n">Spec</span><span class="p">,</span> <span class="n">appNew</span><span class="o">.</span><span class="n">Spec</span><span class="p">)</span>
</code></pre>

</div>



<ol>
<li>
<strong>Loga as diferenças</strong>: Se houver mudanças, registra no log exatamente o que foi alterado
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>   <span class="k">if</span> <span class="n">specsChanged</span> <span class="p">{</span>
       <span class="n">diffSpecs</span> <span class="o">:=</span> <span class="n">cmp</span><span class="o">.</span><span class="n">Diff</span><span class="p">(</span><span class="n">appOld</span><span class="o">.</span><span class="n">Spec</span><span class="p">,</span> <span class="n">appNew</span><span class="o">.</span><span class="n">Spec</span><span class="p">)</span>
       <span class="n">log</span><span class="o">.</span><span class="n">Info</span><span class="p">(</span><span class="s">"platform specs update"</span><span class="p">,</span> <span class="s">"app"</span><span class="p">,</span> <span class="n">appNew</span><span class="o">.</span><span class="n">Name</span><span class="p">,</span> <span class="s">"specs"</span><span class="p">,</span> <span class="n">diffSpecs</span><span class="p">)</span>
   <span class="p">}</span>
</code></pre>

</div>



<ol>
<li>
<strong>Retorna o resultado</strong>: Se a spec mudou, retorna <code>true</code> sinalizando para o controller iniciar a reconciliação. Se não mudou (por exemplo, apenas o <code>.status</code> foi atualizado), retorna <code>false</code> e evita processamento desnecessário</li>
</ol>

<h4>
  
  
  Gerenciando Recursos Dependentes
</h4>

<p>O método <code>Owns()</code> declara quais recursos Kubernetes este controller gerencia:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">appsv1</span><span class="o">.</span><span class="n">Deployment</span><span class="p">{})</span><span class="o">.</span>   <span class="c">// Gerencia os Deployments criados pela App</span>
<span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">corev1</span><span class="o">.</span><span class="n">Service</span><span class="p">{})</span><span class="o">.</span>      <span class="c">// Gerencia os Services criados pela App</span>
<span class="n">Owns</span><span class="p">(</span><span class="o">&amp;</span><span class="n">networkingv1</span><span class="o">.</span><span class="n">Ingress</span><span class="p">{})</span><span class="o">.</span> <span class="c">// Gerencia os Ingresses criados pela App</span>
</code></pre>

</div>



<p>Quando qualquer um desses recursos é modificado ou deletado, o controller é notificado e pode reconciliar o estado desejado.</p>




<h3>
  
  
  Testando o Operador
</h3>

<h4>
  
  
  Build e Deploy
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># gerar/manter manifests</span>
make manifests

<span class="c"># instalar CRD no cluster atual</span>
make <span class="nb">install</span>

<span class="c"># rodar o controller localmente (usa kubeconfig atual)</span>
make run

</code></pre>

</div>



<p><strong>O que você verá:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>2025-01-15T10:30:00.000Z    INFO    controller-runtime.metrics  Metrics server is starting to listen    {"addr": ":8080"}
2025-01-15T10:30:00.001Z    INFO    setup   starting manager
2025-01-15T10:30:00.002Z    INFO    Starting server {"kind": "health probe", "addr": "[::]:8081"}
2025-01-15T10:30:00.003Z    INFO    Starting EventSource    {"controller": "app", "source": "kind source: *v1alpha1.App"}
2025-01-15T10:30:00.104Z    INFO    Starting Controller {"controller": "app"}
2025-01-15T10:30:00.204Z    INFO    Starting workers    {"controller": "app", "worker count": 1}

</code></pre>

</div>



<h3>
  
  
  Criar um Exemplo de App
</h3>

<p>Edite <code>config/samples/platform_v1alpha1_app.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">platform.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">App</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">app-sample</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">deploy</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">nginx:1.27</span>
    <span class="na">replicas</span><span class="pi">:</span> <span class="m">2</span>
    <span class="na">containerPort</span><span class="pi">:</span> <span class="m">8080</span>
  <span class="na">service</span><span class="pi">:</span>
    <span class="na">targetPort</span><span class="pi">:</span> <span class="m">8080</span>
    <span class="na">port</span><span class="pi">:</span> <span class="m">9093</span>
  <span class="na">ingress</span><span class="pi">:</span>
    <span class="na">host</span><span class="pi">:</span> <span class="s">minha-app.local</span>
    <span class="na">path</span><span class="pi">:</span> <span class="s">/</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># em outro terminal, aplicar o sample</span>
kubectl apply <span class="nt">-f</span> config/samples/platform_v1alpha1_app.yaml

</code></pre>

</div>



<h4>
  
  
  Verificando os Recursos Criados
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># verificar recursos</span>
kubectl get app,deploy,svc,ing <span class="nt">-n</span> default <span class="nt">-o</span> wide
kubectl get app minha-app <span class="nt">-n</span> default <span class="nt">-o</span> <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status}\n'</span>

</code></pre>

</div>



<p><strong>Logs do operador:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>2025-01-15T10:35:00.000Z    INFO    Starting reconciliation {"app": "default/app-sample"}
2025-01-15T10:35:00.050Z    INFO    Creating new Deployment {"namespace": "default", "name": "app-sample"}
2025-01-15T10:35:00.150Z    INFO    Creating new Service    {"namespace": "default", "name": "service-app-sample"}
2025-01-15T10:35:00.250Z    INFO    Creating new Ingress    {"namespace": "default", "name": "ingress-app-sample"}
2025-01-15T10:35:00.350Z    INFO    Updating App status {"availableReplicas": 0, "url": "http://minha-app.local/"}
</code></pre>

</div>



<h4>
  
  
  Output Esperado
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>kubectl get app,deploy,svc,ing <span class="nt">-n</span> default

NAME                           
app.platform.example.com/app-sample 
NAME                         READY   UP-TO-DATE   AVAILABLE
deployment.apps/app-sample   2/2     2            2

NAME                      TYPE        CLUSTER-IP      PORT<span class="o">(</span>S<span class="o">)</span>
service/service-app-sample   ClusterIP   10.96.123.45   9093/TCP

NAME                                    HOSTS              PORTS
ingress.networking.k8s.io/ingress-app-sample   minha-app.local   80

</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ kubectl get app app-sample -o jsonpath='{.status}' | jq

{
  "availableReplicas": 2,
  "url": "http://minha-app.local/",
  "observedGeneration": 1
}

</code></pre>

</div>






<h2>
  
  
  O que aprendemos com esse exemplo
</h2>

<p>Esse operador é propositalmente simples, mas já mostra o poder da abordagem:</p>

<p>Ao aplicar o manifesto de app, o operador entra em ação:</p>

<ol>
<li>Cria o Deployment do NGINX.</li>
<li>Cria o Service apontando para o Deployment.</li>
<li>Cria o Ingress configurado para o host definido.
Ou seja: em vez de escrever 3 YAMLs diferentes, bastou um único recurso customizado.</li>
</ol>

<ul>
<li>Ele <strong>reduz YAML</strong> — de três arquivos para um só.</li>
<li>Ele <strong>se atualiza sozinho</strong>: se você trocar a imagem ou o host, os recursos são reconciliados automaticamente.</li>
</ul>

<p>O código completo do operador esta nesse <strong><a href="https://github.com/gabrielalves87/sample-operator" rel="noopener noreferrer">repositorio</a></strong></p>




<h2>
  
  
  Próximos passos e melhorias
</h2>

<ul>
<li>
<strong>TLS automático</strong> com cert-manager (adicionar annotations e emissão de certificados).</li>
<li>
<strong>Annotations e classes de Ingress</strong>: permitir configurações avançadas de roteamento.</li>
<li>
<strong>Probes e HPA</strong> (readiness/liveness, HorizontalPodAutoscaler).</li>
<li>
<strong>Configuração</strong> via ConfigMaps/Secrets e variáveis de ambiente.</li>
<li>
<strong>Finalizers</strong> para limpeza/rotinas antes do delete.</li>
<li>
<strong>Webhooks</strong> para validações/mutações avançadas (ex: defaults dinâmicos, políticas internas).</li>
</ul>




<h2>
  
  
  Conclusão
</h2>

<p>Operadores são uma forma poderosa de ensinar o Kubernetes a <strong>trabalhar por você</strong>. Eles vão além do que Helm pode oferecer, porque conseguem reagir a mudanças, aplicar lógica de negócio e integrar com sistemas externos.<br>
O exemplo que mostrei é apenas uma porta de entrada. A ideia não é que você use exatamente esse operador em produção, mas que veja como é possível começar pequeno e, aos poucos, evoluir para casos mais complexos.<br>
Se você já se sente confortável escrevendo manifestos ou usando Helm, o próximo passo natural é experimentar escrever o seu próprio operador. Quem sabe aquele processo chato e repetitivo que você faz todos os dias não pode virár um CRD com automação total?</p>

