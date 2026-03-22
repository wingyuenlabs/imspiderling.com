---
Title: Checkov Scan para Terraform com Azure Pipelines
Description: 
Author: Leonan Viana
Date: 2026-03-22T21:39:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>Esse post mostra como usar o Checkov para escanear seu IaC Terraform dentro de uma esteira do Azure Pipelines. O objetivo é ter um step dedicado ao scan de segurança antes de qualquer <code>plan</code> ou <code>apply</code> chegar no ambiente.</p>

<h2>
  
  
  O que é o Checkov?
</h2>

<p><a href="https://www.checkov.io/" rel="noopener noreferrer">Checkov</a> é uma ferramenta de análise estática para infraestrutura como código. Ela verifica configurações de recursos cloud e aponta misconfigurations antes que elas sejam provisionadas. Suporta Terraform, CloudFormation, Kubernetes, Helm, ARM Templates, Serverless e AWS CDK.</p>




<h2>
  
  
  Etapa 1 — Criando o template do Checkov
</h2>

<p>Como vamos usar Azure Pipelines, faz sentido isolar as tasks do Checkov em um template reutilizável. O arquivo fica em:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="s">.azuredevops/templates/terraform-build-checkov.yml</span>
</code></pre>

</div>


<p>O template usa duas variáveis predefinidas do Azure DevOps para instalar o Checkov no runner de execução:</p>


<div class="crayons-card c-embed text-styles text-styles--secondary">
    <div class="c-embed__content">
        <div class="c-embed__cover">
          <a href="https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&amp;amp%3Btabs=yaml&amp;amp%3Bsource=post_page-----045f6dd5ab9e---------------------------------------" class="c-link align-middle" rel="noopener noreferrer">
            <img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Flearn.microsoft.com%2Fen-us%2Fmedia%2Fopen-graph-image.png" height="420" class="m-0" width="800">
          </a>
        </div>
      <div class="c-embed__body">
        <h2 class="fs-xl lh-tight">
          <a href="https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&amp;amp%3Btabs=yaml&amp;amp%3Bsource=post_page-----045f6dd5ab9e---------------------------------------" rel="noopener noreferrer" class="c-link">
            Predefined variables - Azure Pipelines | Microsoft Learn
          </a>
        </h2>
          <p class="truncate-at-3">
            A comprehensive list of all available predefined variables
          </p>
        <div class="color-secondary fs-s flex items-center">
          learn.microsoft.com
        </div>
      </div>
    </div>
</div>



<ul>
<li>
<code>$(Agent.ToolsDirectory)</code> — diretório de ferramentas do agente</li>
<li>
<code>$(Agent.OS)</code> — sistema operacional do agente
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">parameters</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">CheckovVersion</span>
    <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
    <span class="na">default</span><span class="pi">:</span> <span class="s1">'</span><span class="s">'</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">WorkingDir</span>
    <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
    <span class="na">default</span><span class="pi">:</span> <span class="s1">'</span><span class="s">.'</span>

<span class="na">steps</span><span class="pi">:</span>

  <span class="c1"># Cache do Checkov para evitar reinstalação a cada run</span>
  <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">Cache@2</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s">Restore checkov $(CheckovVersion) from cache</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">checkov_restore_cache</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">key</span><span class="pi">:</span> <span class="s1">'</span><span class="s">"checkov</span><span class="nv"> </span><span class="s">$(CheckovVersion)"</span><span class="nv"> </span><span class="s">|</span><span class="nv"> </span><span class="s">$(Agent.OS)'</span>
      <span class="na">path</span><span class="pi">:</span> <span class="s">$(Agent.ToolsDirectory)/checkov</span>
      <span class="na">cacheHitVar</span><span class="pi">:</span> <span class="s">CACHE_RESTORED</span>

  <span class="c1"># Instala o Checkov apenas se o cache não existir</span>
  <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">CmdLine@2</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s">Install checkov $(CheckovVersion)</span>
    <span class="na">condition</span><span class="pi">:</span> <span class="s">and(succeeded(), ne(variables.CACHE_RESTORED, 'true'))</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">targetType</span><span class="pi">:</span> <span class="s1">'</span><span class="s">inline'</span>
      <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">CHECKOV_DIR=${AGENT_TOOLSDIRECTORY}/checkov</span>
        <span class="s">mkdir -p $CHECKOV_DIR</span>
        <span class="s">python3 -m venv $CHECKOV_DIR</span>
        <span class="s">source $CHECKOV_DIR/bin/activate</span>
        <span class="s">pip3 install checkov==${{ parameters.CheckovVersion }}</span>
        <span class="s">echo "##vso[task.prependpath]$CHECKOV_DIR/bin"</span>

  <span class="c1"># Adiciona o Checkov ao PATH do agente</span>
  <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">CmdLine@2</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s">Add checkov to PATH</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">targetType</span><span class="pi">:</span> <span class="s1">'</span><span class="s">inline'</span>
      <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">CHECKOV_DIR=${AGENT_TOOLSDIRECTORY}/checkov</span>
        <span class="s">echo "##vso[task.prependpath]$CHECKOV_DIR/bin"</span>

  <span class="c1"># Executa o scan no diretório Terraform informado</span>
  <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">CmdLine@2</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s">Run checkov</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">targetType</span><span class="pi">:</span> <span class="s1">'</span><span class="s">inline'</span>
      <span class="na">script</span><span class="pi">:</span> <span class="s">checkov --directory "${{ parameters.WorkingDir }}" --framework terraform</span>
</code></pre>

</div>



<blockquote>
<p><strong>Por que usar cache?</strong> O Checkov tem algumas dependências Python que levam tempo para instalar. Com a task <code>Cache@2</code>, a instalação só acontece uma vez por versão — nas runs seguintes o agente restaura direto do cache.</p>
</blockquote>




<h2>
  
  
  Etapa 2 — Configurando a Pipeline principal
</h2>

<p>Na raiz do repositório, o <code>azure-pipelines.yaml</code> referencia o template criado acima. A variável <code>$(System.DefaultWorkingDirectory)</code> aponta para o diretório dos arquivos Terraform.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">trigger</span><span class="pi">:</span>
  <span class="na">branches</span><span class="pi">:</span>
    <span class="na">include</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">main</span>
  <span class="na">paths</span><span class="pi">:</span>
    <span class="na">exclude</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">**/*.md'</span>

<span class="na">pool</span><span class="pi">:</span>
  <span class="na">vmImage</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>

<span class="na">variables</span><span class="pi">:</span>
  <span class="na">CheckovVersion</span><span class="pi">:</span> <span class="s1">'</span><span class="s">2.3.110'</span>

<span class="na">steps</span><span class="pi">:</span>

  <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">TerraformTaskV4@4</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s">Terraform Init</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">provider</span><span class="pi">:</span> <span class="s1">'</span><span class="s">aws'</span>
      <span class="na">command</span><span class="pi">:</span> <span class="s1">'</span><span class="s">init'</span>
      <span class="na">workingDirectory</span><span class="pi">:</span> <span class="s1">'</span><span class="s">$(System.DefaultWorkingDirectory)/tf'</span>

  <span class="pi">-</span> <span class="na">template</span><span class="pi">:</span> <span class="s">.azuredevops/templates/terraform-build-checkov.yaml</span>
</code></pre>

</div>



<blockquote>
<p>O exemplo acima tem apenas o <code>Init</code> e o template do Checkov para manter a demonstração simples. Na prática, você vai querer adicionar <code>Validate</code>, <code>TfLint</code>, <code>Plan</code> e <code>Apply</code> conforme a maturidade da sua esteira.</p>
</blockquote>




<h2>
  
  
  Etapa 3 — Executando e interpretando o resultado
</h2>

<p>Ao rodar a pipeline, o log vai mostrar:</p>

<ul>
<li>Quantos recursos foram verificados</li>
<li>Quantos checks passaram, falharam ou foram pulados (skipped)</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmpnzrzd4tvdnajz5kelj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmpnzrzd4tvdnajz5kelj.png" alt=" " width="800" height="408"></a></p>

<p><strong>Se um check falhar, a pipeline quebra.</strong> Esse é o comportamento esperado — a ideia é bloquear o deploy até que a configuração seja corrigida ou explicitamente ignorada.</p>




<h2>
  
  
  Como fazer skip de um check específico
</h2>

<p>Quando um item é avaliado como falso positivo ou está dentro de uma exceção aceitável para o contexto, você pode declarar o skip diretamente no bloco do resource Terraform:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aws_eks_cluster"</span> <span class="s2">"cluster-eks"</span> <span class="p">{</span>

  <span class="c1">#checkov:skip=CKV_AWS_39: "Ensure Amazon EKS public endpoint disabled"</span>
  <span class="c1">#checkov:skip=CKV_AWS_37: "Ensure Amazon EKS control plane logging enabled for all log types"</span>
  <span class="c1">#checkov:skip=CKV_AWS_38: "Ensure Amazon EKS public endpoint not accessible to 0.0.0.0/0"</span>
  <span class="c1">#checkov:skip=CKV_AWS_58: "Ensure EKS Cluster has Secrets Encryption Enabled"</span>

  <span class="c1"># ... restante da configuração</span>

<span class="p">}</span>
</code></pre>

</div>



<p>O comentário segue o padrão <code>#checkov:skip=&lt;CHECK_ID&gt;: "&lt;justificativa&gt;"</code>. A justificativa é opcional mas recomendada para rastreabilidade — especialmente em auditorias.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqu5entt41r48ui3iai6t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqu5entt41r48ui3iai6t.png" alt=" " width="800" height="418"></a></p>




<h2>
  
  
  Conclusão
</h2>

<p>Com esse template, o scan de segurança vira parte do ciclo normal de desenvolvimento. Qualquer misconfiguration no Terraform é bloqueada antes de chegar no <code>plan</code>, o que reduz bastante o risco de provisionar recursos fora de conformidade.</p>

<p>O Checkov suporta os seguintes frameworks:</p>

<ul>
<li>Terraform e Terraform Plan</li>
<li>CloudFormation</li>
<li>Kubernetes</li>
<li>ARM Templates</li>
<li>Serverless</li>
<li>Helm</li>
<li>AWS CDK</li>
</ul>




