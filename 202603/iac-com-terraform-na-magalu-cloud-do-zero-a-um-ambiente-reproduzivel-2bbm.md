---
Title: IaC com Terraform na Magalu Cloud: do zero a um ambiente reproduzível
Description: 
Author: Monica Hillman
Date: 2026-03-05T21:32:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>Infraestrutura em nuvem permite criar recursos rapidamente, mas também traz um desafio comum: <strong>como garantir que ambientes sejam consistentes e reproduzíveis?</strong></p>

<p>Criar recursos manualmente pelo console funciona no início. Porém, conforme a infraestrutura cresce, manter ambientes idênticos entre desenvolvimento, homologação e produção se torna difícil.</p>

<p>É nesse cenário que entra <strong>Infraestrutura como Código (IaC)</strong>.</p>

<p>Neste artigo vamos construir um exemplo prático usando <strong><a href="https://registry.terraform.io/providers/MagaluCloud/mgc/latest/docs" rel="noopener noreferrer">Terraform</a> na <a href="https://magalu.cloud/?utm_source=site&amp;utm_medium=organico&amp;utm_campaign=artigos+comunidade&amp;utm_id=artigos+comunidade" rel="noopener noreferrer">Magalu Cloud</a></strong>, criando um ambiente reproduzível com:</p>

<ul>
<li>Compute (máquina virtual)</li>
<li>Storage persistente (Block Storage)</li>
<li>Object Storage</li>
<li>State remoto para colaboração entre equipes</li>
</ul>

<p>Ao final, você terá um projeto capaz de <strong>recriar toda a infraestrutura em poucos comandos</strong>.</p>




<h2>
  
  
  O que é Infraestrutura como Código (IaC)
</h2>

<p>Infraestrutura como Código é a prática de <strong>descrever a infraestrutura em arquivos de configuração versionáveis</strong>.</p>

<p>Em vez de criar recursos manualmente, você escreve definições declarativas que descrevem o estado desejado do ambiente.</p>

<p>O Terraform interpreta essas definições e executa as mudanças necessárias.</p>

<p>Esse modelo traz benefícios importantes:</p>

<ul>
<li>
<strong>Reprodutibilidade</strong> — qualquer pessoa pode recriar o ambiente</li>
<li>
<strong>Versionamento</strong> — mudanças ficam registradas em Git</li>
<li>
<strong>Previsibilidade</strong> — <code>terraform plan</code> mostra o que será alterado</li>
<li>
<strong>Automação</strong> — integração simples com pipelines de CI/CD</li>
</ul>

<p>Na Magalu Cloud, a infraestrutura pode ser gerenciada por <strong>Console, APIs, CLI ou <a href="https://docs.magalu.cloud/docs/devops-tools/terraform/overview/?utm_source=site&amp;utm_medium=organico&amp;utm_campaign=artigos+comunidade+documentacao&amp;utm_id=artigos+comunidade+documentacao" rel="noopener noreferrer">Terraform</a></strong>, utilizando o provider oficial:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
magalucloud/mgc

</code></pre>

</div>






<h2>
  
  
  Arquitetura que vamos criar
</h2>

<p>Neste artigo vamos montar uma infraestrutura simples e reproduzível.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
VM (Compute)
├─ IP público
└─ Volume Block Storage

Object Storage
└─ Bucket para dados e Terraform state

</code></pre>

</div>



<p>Esse ambiente já permite executar aplicações e armazenar dados persistentes.</p>




<h2>
  
  
  Configurando o provider da Magalu Cloud
</h2>

<p>No arquivo <code>main.tf</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">required_providers</span> <span class="p">{</span>
    <span class="nx">mgc</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">source</span> <span class="p">=</span> <span class="s2">"magalucloud/mgc"</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">provider</span> <span class="s2">"mgc"</span> <span class="p">{</span>
  <span class="nx">region</span>  <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">mgc_region</span>
  <span class="nx">api_key</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">mgc_api_key</span>
<span class="p">}</span>
</code></pre>

</div>



<p>A autenticação pode ser feita usando variáveis de ambiente como:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>MGC_API_KEY
MGC_REGION
MGC_OBJ_KEY_ID
MGC_OBJ_KEY_SECRET
MGC_ENV
</code></pre>

</div>



<p>Isso evita expor credenciais diretamente no código.</p>




<h2>
  
  
  Inicializando o Terraform
</h2>

<p>Depois de configurar o provider:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>terraform init
</code></pre>

</div>



<p>Visualize o que será criado:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>terraform plan
</code></pre>

</div>



<p>E aplique a infraestrutura:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>terraform apply
</code></pre>

</div>



<p>Esses comandos fazem parte do fluxo básico de trabalho com Terraform.</p>




<h2>
  
  
  Criando a máquina virtual
</h2>

<p>A criação de uma instância exige alguns parâmetros básicos:</p>

<ul>
<li>Availability Zone</li>
<li>imagem do sistema operacional</li>
<li>tipo da máquina</li>
<li>chave SSH</li>
</ul>

<p>Exemplo simplificado:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"mgc_virtual_machine_instances"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">name</span>              <span class="p">=</span> <span class="s2">"app-1"</span>
  <span class="nx">availability_zone</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">az</span>

  <span class="nx">image_id</span>        <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">image_id</span>
  <span class="nx">machine_type_id</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">machine_type_id</span>

  <span class="nx">ssh_key_name</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">ssh_key_name</span>

  <span class="nx">network</span> <span class="p">{</span>
    <span class="nx">associate_public_ip</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Depois da execução do Terraform, a VM estará pronta para acesso via SSH.</p>




<h2>
  
  
  Adicionando armazenamento persistente
</h2>

<p>Para aplicações que precisam armazenar dados de forma persistente, podemos anexar um volume de Block Storage à instância.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"mgc_block_storage_volume"</span> <span class="s2">"data"</span> <span class="p">{</span>
  <span class="nx">name</span> <span class="p">=</span> <span class="s2">"volume-via-terraform"</span>
  <span class="nx">size</span> <span class="p">=</span> <span class="mi">30</span>

  <span class="nx">type</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">name</span> <span class="p">=</span> <span class="s2">"cloud_nvme20k"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"mgc_block_storage_attachment"</span> <span class="s2">"data_attach"</span> <span class="p">{</span>
  <span class="nx">volume_id</span>   <span class="p">=</span> <span class="nx">mgc_block_storage_volume</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">instance_id</span> <span class="p">=</span> <span class="nx">mgc_virtual_machine_instances</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Esse volume pode ser utilizado para bancos de dados, arquivos de aplicação ou dados de usuário.</p>

<p>Um ponto importante: <strong>volumes precisam estar na mesma Availability Zone da VM para serem anexados.</strong></p>




<h2>
  
  
  Usando Object Storage para Terraform state
</h2>

<p>Quando várias pessoas trabalham no mesmo projeto Terraform, é importante armazenar o <strong>Terraform state remotamente</strong>.</p>

<p>Na Magalu Cloud, o Object Storage é compatível com S3, permitindo usar o backend <code>s3</code> do Terraform.</p>

<p>Exemplo conceitual:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">backend</span> <span class="s2">"s3"</span> <span class="p">{</span>
    <span class="nx">bucket</span>   <span class="p">=</span> <span class="s2">"terraform-state"</span>
    <span class="nx">key</span>      <span class="p">=</span> <span class="s2">"iac/terraform.tfstate"</span>
    <span class="nx">region</span>   <span class="p">=</span> <span class="s2">"br-ne1"</span>
    <span class="nx">endpoint</span> <span class="p">=</span> <span class="s2">"https://br-ne1.magaluobjects.com"</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Isso evita conflitos quando múltiplos desenvolvedores executam o Terraform.</p>




<h2>
  
  
  Conclusão
</h2>

<p>Infraestrutura como Código muda completamente a forma de gerenciar ambientes em nuvem.</p>

<p>Com Terraform na Magalu Cloud é possível:</p>

<ul>
<li>automatizar provisionamento</li>
<li>reproduzir ambientes completos</li>
<li>reduzir erros manuais</li>
<li>integrar infraestrutura ao fluxo de desenvolvimento</li>
</ul>

<p>Ao definir compute e storage em código, sua infraestrutura deixa de ser um conjunto de recursos criados manualmente e passa a ser <strong>um sistema previsível, versionável e automatizado</strong>.</p>

<p>Links interessantes:</p>

<ul>
<li>
<a href="https://magalu.cloud/blog/magalu-cloud-e-certificada-pela-hashicorp-para-fornecer-recursos-da-ferramenta-terraform-registry/?utm_source=site&amp;utm_medium=organico&amp;utm_campaign=artigos+comunidade&amp;utm_id=artigos+comunidade" rel="noopener noreferrer">Magalu Cloud é certificada pela HashiCorp para fornecer recursos da ferramenta Terraform Registry</a>;</li>
<li>
<a href="https://www.youtube.com/watch?v=6FH14RMvC9c" rel="noopener noreferrer">Usando Object Storage como Backend para Terraform State</a>;</li>
<li>
<a href="https://www.youtube.com/watch?v=8U6sN0nHj_Q" rel="noopener noreferrer">Testando o terraform da Magalu Cloud‬</a>;</li>
<li>
<a href="https://www.youtube.com/watch?v=DMb2FsoD-H8" rel="noopener noreferrer">Como criar uma VM via Terraform no Magalu Cloud?</a>.</li>
</ul>

