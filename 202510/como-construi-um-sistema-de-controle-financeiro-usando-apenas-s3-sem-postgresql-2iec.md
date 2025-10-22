---
Title: Como construí um Sistema de Controle Financeiro usando APENAS S3 (sem PostgreSQL!)
Description: 
Author: Matheus Cruvinel
Date: 2025-10-22T22:03:15.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>TL;DR:</strong> Construí um sistema completo de gestão financeira pessoal usando apenas Object Storage (S3). Sem PostgreSQL, sem MongoDB, sem Redis. Resultado: infraestrutura de ~R$1/mês com performance surpreendente.</p>




<h2>
  
  
  A Provocação
</h2>

<p>Durante uma conversa com alguns amigos, surgiu a pergunta: <em>"Você realmente precisa de um banco de dados para tudo?"</em></p>

<p>A resposta padrão seria: <em>"Claro! Como você vai fazer queries? E as transações? E os índices?"</em></p>

<p>Mas e se... não precisássemos?</p>

<p>E se pudéssemos construir um sistema funcional de controle financeiro usando <strong>apenas Object Storage</strong>?</p>

<p>Spoiler: Funcionou. E funcionou bem.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffmny54m0bw68dsq1y92j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffmny54m0bw68dsq1y92j.png" alt="Dashboard NimbusVault" width="800" height="611"></a></p>

<h2>
  
  
  O Problema Real
</h2>

<p>Como desenvolvedor, gerenciar finanças pessoais sempre foi caótico para mim:</p>

<ul>
<li>📂 Faturas de cartão espalhadas no email</li>
<li>🧾 Comprovantes em pastas desorganizadas no Drive</li>
<li>💼 Recibos de investimentos perdidos</li>
<li>📊 Documentos do IR em diversos lugares</li>
<li>🔍 Impossível encontrar algo quando precisa</li>
</ul>

<p>Pior: tentei usar apps prontos, mas todos tinham problemas:</p>

<ul>
<li>
<strong>Apps mobile</strong>: Ótimos para registrar gastos, péssimos para documentos</li>
<li>
<strong>Google Drive</strong>: Falta estrutura e automação</li>
<li>
<strong>Planilhas</strong>: Funcionam até você ter 500 linhas</li>
<li>
<strong>Apps de terceiros</strong>: Lock-in + custos recorrentes altos</li>
</ul>

<p>Eu queria algo <strong>simples, barato e que EU controlasse</strong>.</p>

<h2>
  
  
  Deploy: 1 Minuto
</h2>

<p>Usei Docker Compose para tornar o deploy trivial:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.8'</span>

<span class="na">services</span><span class="pi">:</span>
  <span class="na">backend</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span> <span class="s">./backend</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8000:8000"</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">BUCKET=${BUCKET}</span>
      <span class="pi">-</span> <span class="s">ACCESS_KEY=${ACCESS_KEY}</span>
      <span class="pi">-</span> <span class="s">SECRET_KEY=${SECRET_KEY}</span>

  <span class="na">frontend</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span> <span class="s">./frontend</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">3000:80"</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">backend</span>
</code></pre>

</div>



<p>Deploy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/mcruvinel/NimbusVault
<span class="nb">cd </span>NimbusVault
<span class="nb">cp</span> .env.example .env  <span class="c"># Configure suas credenciais</span>
docker compose up <span class="nt">-d</span>
</code></pre>

</div>



<p>Pronto. Sistema rodando.</p>

<h2>
  
  
  A Ideia Maluca
</h2>

<p>"E se eu usar S3 como 'banco de dados'?"</p>

<p>Meu primeiro pensamento foi: <em>"Isso é burrice, S3 não foi feito pra isso"</em></p>

<p>Mas então percebi algumas coisas:</p>

<ol>
<li>
<strong>S3 tem estrutura hierárquica</strong> (<code>categoria/arquivo.pdf</code>)</li>
<li>
<strong>list_objects_v2 é como um SELECT</strong> com filtros</li>
<li><strong>URLs pré-assinadas eliminam proxy de arquivos</strong></li>
<li>
<strong>Durabilidade de 99.999999999%</strong> (melhor que meu PostgreSQL caseiro)</li>
<li>
<strong>Custo ridiculamente baixo</strong> para uso pessoal</li>
</ol>

<p>Decidi testar. O pior que poderia acontecer era perder um fim de semana.</p>

<h2>
  
  
  A Solução: NimbusVault
</h2>

<p>Construí o <strong>NimbusVault</strong> em um fim de semana. O conceito é simples:</p>

<p><strong>Object Storage organizado = Banco de dados funcional</strong></p>

<h3>
  
  
  Arquitetura
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────┐      ┌──────────────┐      ┌─────────────────┐
│   React     │─────▶│   FastAPI    │─────▶│  MagaluCloud    │
│  Frontend   │      │   Backend    │      │  Object Storage │
└─────────────┘      └──────────────┘      └─────────────────┘
  Tailwind UI          boto3 + S3            11 nines durability
</code></pre>

</div>



<h3>
  
  
  Estrutura "Tabelas"
</h3>

<p>Em vez de tabelas SQL, uso <strong>prefixos como categorias</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>finance-bucket/
├── despesas-mensais/
│   ├── fatura-cartao-out-2025.pdf
│   ├── recibo-aluguel-out.pdf
│   └── recibo-energia-outubro.pdf
├── receitas/
│   ├── holerite-outubro-2025.pdf
│   └── freelance-projeto-x.pdf
├── investimentos/
│   ├── tesouro-direto-setembro.pdf
│   └── extrato-corretora-out.pdf
└── impostos/
    ├── darf-irpf-2024.pdf
    └── comprovante-iptu.pdf
</code></pre>

</div>



<p>Simples. Óbvio. <strong>Funcional</strong>.</p>

<h2>
  
  
  Implementação
</h2>

<h3>
  
  
  Backend Minimalista (FastAPI)
</h3>

<p>O backend tem <strong>menos de 200 linhas</strong>. Não é exagero.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">fastapi</span> <span class="kn">import</span> <span class="n">FastAPI</span><span class="p">,</span> <span class="n">UploadFile</span><span class="p">,</span> <span class="n">Form</span><span class="p">,</span> <span class="n">File</span>
<span class="kn">import</span> <span class="n">boto3</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">FastAPI</span><span class="p">(</span><span class="n">title</span><span class="o">=</span><span class="sh">"</span><span class="s">NimbusVault</span><span class="sh">"</span><span class="p">)</span>

<span class="n">s3</span> <span class="o">=</span> <span class="n">boto3</span><span class="p">.</span><span class="nf">client</span><span class="p">(</span>
    <span class="sh">"</span><span class="s">s3</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">endpoint_url</span><span class="o">=</span><span class="n">ENDPOINT_URL</span><span class="p">,</span>
    <span class="n">aws_access_key_id</span><span class="o">=</span><span class="n">ACCESS_KEY</span><span class="p">,</span>
    <span class="n">aws_secret_access_key</span><span class="o">=</span><span class="n">SECRET_KEY</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@app.post</span><span class="p">(</span><span class="sh">"</span><span class="s">/upload</span><span class="sh">"</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">upload_file</span><span class="p">(</span><span class="n">category</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="nc">Form</span><span class="p">(...),</span> <span class="nb">file</span><span class="p">:</span> <span class="n">UploadFile</span> <span class="o">=</span> <span class="nc">File</span><span class="p">(...)):</span>
    <span class="sh">"""</span><span class="s">Upload de documento financeiro</span><span class="sh">"""</span>
    <span class="n">key</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">category</span><span class="si">}</span><span class="s">/</span><span class="si">{</span><span class="nb">file</span><span class="p">.</span><span class="n">filename</span><span class="si">}</span><span class="sh">"</span>
    <span class="n">content</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">file</span><span class="p">.</span><span class="nf">read</span><span class="p">()</span>

    <span class="n">s3</span><span class="p">.</span><span class="nf">put_object</span><span class="p">(</span>
        <span class="n">Bucket</span><span class="o">=</span><span class="n">BUCKET</span><span class="p">,</span>
        <span class="n">Key</span><span class="o">=</span><span class="n">key</span><span class="p">,</span>
        <span class="n">Body</span><span class="o">=</span><span class="n">content</span><span class="p">,</span>
        <span class="n">ContentType</span><span class="o">=</span><span class="nb">file</span><span class="p">.</span><span class="n">content_type</span><span class="p">,</span>
    <span class="p">)</span>

    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">ok</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">key</span><span class="sh">"</span><span class="p">:</span> <span class="n">key</span><span class="p">}</span>

<span class="nd">@app.get</span><span class="p">(</span><span class="sh">"</span><span class="s">/files</span><span class="sh">"</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">list_files</span><span class="p">(</span><span class="n">category</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="bp">None</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Lista documentos (opcionalmente filtrados por categoria)</span><span class="sh">"""</span>
    <span class="n">params</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">Bucket</span><span class="sh">"</span><span class="p">:</span> <span class="n">BUCKET</span><span class="p">}</span>
    <span class="k">if</span> <span class="n">category</span><span class="p">:</span>
        <span class="n">params</span><span class="p">[</span><span class="sh">"</span><span class="s">Prefix</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">category</span><span class="si">}</span><span class="s">/</span><span class="sh">"</span>

    <span class="n">response</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="nf">list_objects_v2</span><span class="p">(</span><span class="o">**</span><span class="n">params</span><span class="p">)</span>

    <span class="n">files</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">obj</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">Contents</span><span class="sh">'</span><span class="p">,</span> <span class="p">[]):</span>
        <span class="c1"># Gera URL pré-assinada (válida por 1 hora)
</span>        <span class="n">url</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="nf">generate_presigned_url</span><span class="p">(</span>
            <span class="sh">'</span><span class="s">get_object</span><span class="sh">'</span><span class="p">,</span>
            <span class="n">Params</span><span class="o">=</span><span class="p">{</span><span class="sh">'</span><span class="s">Bucket</span><span class="sh">'</span><span class="p">:</span> <span class="n">BUCKET</span><span class="p">,</span> <span class="sh">'</span><span class="s">Key</span><span class="sh">'</span><span class="p">:</span> <span class="n">obj</span><span class="p">[</span><span class="sh">'</span><span class="s">Key</span><span class="sh">'</span><span class="p">]},</span>
            <span class="n">ExpiresIn</span><span class="o">=</span><span class="mi">3600</span>
        <span class="p">)</span>

        <span class="n">files</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
            <span class="sh">"</span><span class="s">key</span><span class="sh">"</span><span class="p">:</span> <span class="n">obj</span><span class="p">[</span><span class="sh">"</span><span class="s">Key</span><span class="sh">"</span><span class="p">],</span>
            <span class="sh">"</span><span class="s">size</span><span class="sh">"</span><span class="p">:</span> <span class="n">obj</span><span class="p">[</span><span class="sh">"</span><span class="s">Size</span><span class="sh">"</span><span class="p">],</span>
            <span class="sh">"</span><span class="s">modified</span><span class="sh">"</span><span class="p">:</span> <span class="n">obj</span><span class="p">[</span><span class="sh">"</span><span class="s">LastModified</span><span class="sh">"</span><span class="p">],</span>
            <span class="sh">"</span><span class="s">url</span><span class="sh">"</span><span class="p">:</span> <span class="n">url</span>
        <span class="p">})</span>

    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">files</span><span class="sh">"</span><span class="p">:</span> <span class="n">files</span><span class="p">}</span>
</code></pre>

</div>



<p>Isso é tudo. Sério.</p>

<h3>
  
  
  Frontend Responsivo (React + Tailwind)
</h3>

<p>Interface clean com drag-and-drop:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="kd">const</span> <span class="nx">NimbusVault</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">category</span><span class="p">,</span> <span class="nx">setCategory</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">'</span><span class="s1">despesas-mensais</span><span class="dl">'</span><span class="p">);</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">files</span><span class="p">,</span> <span class="nx">setFiles</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">([]);</span>

  <span class="kd">const</span> <span class="nx">handleUpload</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">formData</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">FormData</span><span class="p">();</span>
    <span class="nx">formData</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="dl">'</span><span class="s1">category</span><span class="dl">'</span><span class="p">,</span> <span class="nx">category</span><span class="p">);</span>
    <span class="nx">formData</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="dl">'</span><span class="s1">file</span><span class="dl">'</span><span class="p">,</span> <span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">files</span><span class="p">[</span><span class="mi">0</span><span class="p">]);</span>

    <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/upload</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">body</span><span class="p">:</span> <span class="nx">formData</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="nf">loadFiles</span><span class="p">();</span>
  <span class="p">};</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">CategorySelector</span> <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">category</span><span class="si">}</span> <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="nx">setCategory</span><span class="si">}</span> <span class="p">/&gt;</span>
      <span class="p">&lt;</span><span class="nc">UploadZone</span> <span class="na">onUpload</span><span class="p">=</span><span class="si">{</span><span class="nx">handleUpload</span><span class="si">}</span> <span class="p">/&gt;</span>
      <span class="p">&lt;</span><span class="nc">FileList</span> <span class="na">files</span><span class="p">=</span><span class="si">{</span><span class="nx">files</span><span class="si">}</span> <span class="p">/&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  Features Surpreendentes
</h2>

<h3>
  
  
  1. URLs Pré-assinadas (Game Changer)
</h3>

<p>Em vez de fazer o backend servir arquivos (gastando banda e CPU), gero <strong>URLs temporárias que apontam direto pro S3</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">presigned_url</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="nf">generate_presigned_url</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">get_object</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">Params</span><span class="o">=</span><span class="p">{</span><span class="sh">'</span><span class="s">Bucket</span><span class="sh">'</span><span class="p">:</span> <span class="n">BUCKET</span><span class="p">,</span> <span class="sh">'</span><span class="s">Key</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">receitas/holerite.pdf</span><span class="sh">'</span><span class="p">},</span>
    <span class="n">ExpiresIn</span><span class="o">=</span><span class="mi">3600</span>  <span class="c1"># 1 hora
</span><span class="p">)</span>
</code></pre>

</div>



<p>Vantagens:</p>

<ul>
<li>✅ Download direto do S3 (super rápido)</li>
<li>✅ Backend não processa tráfego</li>
<li>✅ Segurança: URLs expiram automaticamente</li>
<li>✅ Funciona com <code>&lt;a download&gt;</code> nativamente</li>
</ul>

<h3>
  
  
  2. Busca "Good Enough"
</h3>

<p>Não tenho Elasticsearch, mas funciona:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Backend lista TODOS os arquivos (rápido até ~10k objetos)
</span><span class="n">all_files</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="nf">list_objects_v2</span><span class="p">(</span><span class="n">Bucket</span><span class="o">=</span><span class="n">BUCKET</span><span class="p">)</span>

<span class="c1"># Frontend filtra com JavaScript
</span><span class="n">const</span> <span class="n">filteredFiles</span> <span class="o">=</span> <span class="n">files</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="n">f</span> <span class="o">=&gt;</span> 
  <span class="n">f</span><span class="p">.</span><span class="n">key</span><span class="p">.</span><span class="nf">toLowerCase</span><span class="p">().</span><span class="nf">includes</span><span class="p">(</span><span class="n">searchQuery</span><span class="p">.</span><span class="nf">toLowerCase</span><span class="p">())</span>
<span class="p">);</span>
</code></pre>

</div>



<p>Para 99% dos casos de uso pessoal (&lt; 5000 documentos), <strong>isso é mais que suficiente</strong>.</p>

<h3>
  
  
  3. "Particionamento" Natural
</h3>

<p>Usar prefixos como categorias é essencialmente particionamento:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Lista APENAS despesas (rápido)
</span><span class="n">s3</span><span class="p">.</span><span class="nf">list_objects_v2</span><span class="p">(</span><span class="n">Bucket</span><span class="o">=</span><span class="n">BUCKET</span><span class="p">,</span> <span class="n">Prefix</span><span class="o">=</span><span class="sh">"</span><span class="s">despesas-mensais/</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Lista APENAS investimentos (rápido)
</span><span class="n">s3</span><span class="p">.</span><span class="nf">list_objects_v2</span><span class="p">(</span><span class="n">Bucket</span><span class="o">=</span><span class="n">BUCKET</span><span class="p">,</span> <span class="n">Prefix</span><span class="o">=</span><span class="sh">"</span><span class="s">investimentos/</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>Mesmo com 10.000 arquivos, listar uma categoria específica é <strong>instantâneo</strong>.</p>

<h3>
  
  
  4. Versionamento Grátis
</h3>

<p>S3 tem versionamento nativo. Ativei e agora tenho <strong>audit trail de graça</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws s3api list-object-versions <span class="se">\</span>
  <span class="nt">--bucket</span> finance-bucket <span class="se">\</span>
  <span class="nt">--prefix</span> despesas-mensais/fatura.pdf
</code></pre>

</div>



<p>Alguém deletou um documento importante? <code>s3.get_object(VersionId=...)</code> e recuperei.</p>

<h2>
  
  
  Por que MagaluCloud?
</h2>

<p>Testei AWS S3, mas os custos no Brasil são altos (transferência de dados internacional).</p>

<p><strong>MagaluCloud foi uma revelação:</strong></p>

<h3>
  
  
  Vantagens
</h3>

<ol>
<li>
<strong>Datacenters no Brasil</strong> → latência &lt;50ms (vs 150ms+ AWS EUA)</li>
<li>
<strong>Preços acessíveis</strong> → R$0,06/GiB/mês de storage (Cold Instant)</li>
<li>
<strong>API S3-compatível</strong> → boto3 funciona sem alteração</li>
<li>
<strong>Transferência barata</strong> → R$0,20/GiB de saída, R$0,01/GiB de entrada</li>
<li>
<strong>Suporte em português</strong> → respondem em minutos</li>
</ol>

<h3>
  
  
  Custos Reais (3 meses de uso)
</h3>

<p>Meu uso pessoal:</p>

<ul>
<li>
<strong>5 GB de documentos</strong> (≈4,66 GiB)</li>
<li><strong>~1000 requisições/mês</strong></li>
</ul>

<p><strong>Fatura mensal: ≈R$0,50/mês</strong> 🤯</p>

<p>Compare com:</p>

<ul>
<li>PostgreSQL gerenciado: R$50-200/mês</li>
<li>MongoDB Atlas: R$80+/mês</li>
<li>Heroku Postgres: $50/mês (~R$250)</li>
</ul>

<h2>
  
  
  Quando Usar
</h2>

<p>✅ <strong>Storage de documentos/comprovantes/arquivos</strong><br><br>
✅ <strong>Logs e eventos</strong> (time-series)<br><br>
✅ <strong>Prototipagem rápida</strong><br><br>
✅ <strong>Side projects</strong> (custos baixos)<br><br>
✅ <strong>Sistemas WORM</strong> (Write Once, Read Many)<br><br>
✅ <strong>Dados não-relacionais</strong> (não precisa de joins)</p>

<p>Basicamente: se você está pensando em "salvar arquivos no servidor", <strong>considere S3 diretamente</strong>.</p>
<h2>
  
  
  Lições Aprendidas
</h2>
<h3>
  
  
  1. Simplicidade Vence
</h3>

<p>Gastei <strong>5 horas</strong> construindo isso. Gastaria <strong>5 semanas</strong> com PostgreSQL + migrations + ORM + backups + monitoramento.</p>
<h3>
  
  
  2. Object Storage é Subestimado
</h3>

<p>Desenvolvedores pensam "S3 = arquivos grandes". Errado. S3 é <strong>key-value store distribuído</strong> com storage barato.</p>
<h3>
  
  
  3. Nem Tudo Precisa de ACID
</h3>

<p>95% das minhas queries são:</p>

<ul>
<li>"Me dá esse arquivo"</li>
<li>"Lista arquivos dessa categoria"</li>
<li>"Deleta isso"</li>
</ul>

<p><strong>Nada disso precisa de transações complexas.</strong></p>
<h3>
  
  
  4. Prefixos = Índices Grátis
</h3>

<p>Organizar por <code>categoria/subcategoria/arquivo</code> é essencialmente criar índices.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>despesas-mensais/2024/10/fatura.pdf
</code></pre>

</div>



<p>Isso me permite:</p>

<ul>
<li>Listar tudo de outubro: <code>Prefix="despesas-mensais/2024/10/"</code>
</li>
<li>Listar tudo de 2024: <code>Prefix="despesas-mensais/2024/"</code>
</li>
<li>Listar todas despesas: <code>Prefix="despesas-mensais/"</code>
</li>
</ul>

<h3>
  
  
  5. Infraestrutura Brasileira Importa
</h3>

<p>MagaluCloud com datacenter em SP = <strong>&lt;50ms de latência</strong>.<br><br>
AWS US East = <strong>180ms de latência</strong>.</p>

<p>Para usuário final, isso é <strong>perceptível</strong>.</p>

<h2>
  
  
  Conclusão
</h2>

<p>Construir NimbusVault me ensinou que <strong>nem sempre precisamos da solução mais complexa</strong>.</p>

<p><strong>Antes:</strong></p>

<ul>
<li>PostgreSQL (R$80/mês)</li>
<li>Redis para cache (R$40/mês)
</li>
<li>VM com 2GB RAM (R$60/mês)</li>
<li><strong>Total: R$180/mês</strong></li>
</ul>

<p><strong>Agora:</strong></p>

<ul>
<li>MagaluCloud Object Storage (~R$1/mês)</li>
<li><strong>Total: R$1/mês</strong></li>
</ul>

<p><strong>Economia: R$2.148/ano</strong> 💰</p>

<p>E o melhor: <strong>menos coisas para quebrar</strong>.</p>

<p>Não preciso me preocupar com:</p>

<ul>
<li>Backups (S3 é 11 noves de durabilidade)</li>
<li>Escalabilidade (S3 escala infinito)</li>
<li>Manutenção (zero administração)</li>
<li>Atualizações (API não muda)</li>
</ul>

<h2>
  
  
  Código Aberto
</h2>

<p>Todo o código está no GitHub:<br>
<strong><a href="https://github.com/mcruvinel/NimbusVault" rel="noopener noreferrer">github.com/mcruvinel/NimbusVault</a></strong></p>

<p>Sinta-se livre para:</p>

<ul>
<li>⭐ Dar uma estrela</li>
<li>🍴 Fazer fork</li>
<li>🐛 Reportar bugs</li>
<li>💡 Sugerir features</li>
</ul>

<h2>
  
  
  Experimente Você Mesmo
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Clone</span>
git clone https://github.com/mcruvinel/NimbusVault
<span class="nb">cd </span>NimbusVault

<span class="c"># 2. Configure</span>
<span class="nb">cp</span> .env.example .env
<span class="c"># Edite .env com suas credenciais MagaluCloud</span>

<span class="c"># 3. Run</span>
docker compose up <span class="nt">-d</span>

<span class="c"># 4. Acesse</span>
open http://localhost:8080
</code></pre>

</div>



<p><strong>Como obter credenciais:</strong></p>

<ol>
<li>Acesse <a href="https://console.magalu.cloud" rel="noopener noreferrer">console.magalu.cloud</a>
</li>
<li>Crie um bucket em Object Storage</li>
<li>Gere um par de chaves de acesso</li>
</ol>

<h2>
  
  
  Pensamentos Finais
</h2>

<p>Este projeto começou como um experimento de fim de semana e virou minha solução real de gestão financeira.</p>

<p>Prova que <strong>ferramentas simples, usadas criativamente, resolvem problemas complexos</strong>.</p>

<p>Às vezes, você só precisa de <strong>um bucket bem organizado</strong>.</p>




<p><strong>Gostou? Compartilhe este post!</strong></p>

<ul>
<li>
<strong>GitHub:</strong> <a href="https://github.com/mcruvinel" rel="noopener noreferrer">@mcruvinel</a>
</li>
<li>
<strong>Blog:</strong> <a href="https://mount-log.hashnode.dev/" rel="noopener noreferrer">mount.log</a>
</li>
</ul>

<p><strong>Documentação</strong> <a href="https://docs.magalu.cloud/docs/storage/object-storage/overview" rel="noopener noreferrer">MagaluCloud ObjectStorage</a></p>

<p><strong>Tags:</strong> #Python #FastAPI #React #S3 #MagaluCloud #ObjectStorage #CloudStorage #FinTech #OpenSource #Serverless</p>

