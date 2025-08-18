---
Title: E-mails de verificação com AWS SES + Lambda (Node.js) e Terraform — do zero ao envio
Description: 
Author: Cláudio Filipe Lima Rapôso
Date: 2025-08-18T21:43:30.000Z
Robots: noindex,nofollow
Template: index
---
<p>Quero te levar do <strong>zero</strong> até o <strong>primeiro e-mail de verificação enviado</strong> pela AWS, usando <strong>Terraform</strong> para criar a infra e <strong>AWS Lambda (Node.js com AWS SDK v3)</strong> para enviar. A ideia é simples: você provisiona tudo como código, empacota uma pequena função de envio e testa em minutos.</p>

<blockquote>
<p>Observação importante sobre o SES: contas novas começam no <strong>sandbox</strong>. Nesse modo, você só consegue enviar para <strong>endereços verificados</strong> (remetente e destinatário) e com limites mais baixos. Para enviar para qualquer pessoa, é preciso <strong>solicitar saída do sandbox</strong>. Vou te mostrar como validar identidades e como testar mesmo no sandbox. (<a href="https://docs.aws.amazon.com/ses/latest/dg/verify-addresses-and-domains.html?utm_source=chatgpt.com" rel="noopener noreferrer">AWS Documentation</a>)</p>
</blockquote>




<h2>
  
  
  O problema que essa solução resolve
</h2>

<p>Quase toda app precisa confirmar se um e-mail realmente pertence ao usuário. Em vez de acoplar envio de e-mail no backend principal, vamos <strong>isolar a responsabilidade</strong> em uma <strong>Lambda</strong> enxuta, com <strong>boas práticas de engenharia</strong> (SOLID/DRY, separação de camadas) e <strong>infra como código</strong>. Resultado: baixo acoplamento, escala automática e governança simples.</p>




<h2>
  
  
  O que vamos montar
</h2>

<ul>
<li>Um <strong>Email Identity</strong> no <strong>Amazon SES</strong> (o remetente).</li>
<li>Uma <strong>IAM Role</strong> mínima para a Lambda.</li>
<li>Uma <strong>Lambda</strong> em Node.js 20 que usa <strong>AWS SDK v3 (SESv2)</strong> para enviar.</li>
<li>Opcional: um <strong>Function URL</strong> para chamar a Lambda por HTTP e testar com <code>curl</code>/Postman. (<a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lambda_function_url.html?utm_source=chatgpt.com" rel="noopener noreferrer">Terraform Registry</a>)</li>
</ul>




<h2>
  
  
  Pré-requisitos
</h2>

<ul>
<li>Conta AWS com <code>aws configure</code> pronto.</li>
<li>Terraform ≥ 1.5.</li>
<li>Node.js 20 + npm.</li>
<li>Região do SES suportada (ex.: <code>us-east-1</code>).</li>
</ul>




<h2>
  
  
  Estrutura de pastas
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ses-verify/
  app/
    package.json
    tsconfig.json
    src/
      config.ts
      dto.ts
      email/
        SesEmailClient.ts
        EmailService.ts
      handler.ts
  build/        # gerado pelo script
  infra/
    main.tf
    variables.tf
    outputs.tf
</code></pre>

</div>






<h2>
  
  
  Passo 1 — Infra com Terraform (SES, Lambda, IAM e Function URL)
</h2>

<p>Crie os arquivos abaixo em <code>infra/</code>.</p>

<h3>
  
  
  <code>variables.tf</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"project_name"</span> <span class="p">{</span>
  <span class="nx">type</span>    <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span> <span class="p">=</span> <span class="s2">"ses-verify"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"aws_region"</span> <span class="p">{</span>
  <span class="nx">type</span>    <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span> <span class="p">=</span> <span class="s2">"us-east-1"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"sender_email"</span> <span class="p">{</span>
  <span class="nx">type</span> <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"lambda_package_path"</span> <span class="p">{</span>
  <span class="nx">type</span>    <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span> <span class="p">=</span> <span class="s2">"../build/lambda.zip"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"expose_function_url"</span> <span class="p">{</span>
  <span class="nx">type</span>    <span class="p">=</span> <span class="nx">bool</span>
  <span class="nx">default</span> <span class="p">=</span> <span class="kc">false</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"function_url_auth_type"</span> <span class="p">{</span>
  <span class="nx">type</span>    <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span> <span class="p">=</span> <span class="s2">"NONE"</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  <code>main.tf</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">required_version</span> <span class="p">=</span> <span class="s2">"&gt;= 1.5.0"</span>
  <span class="nx">required_providers</span> <span class="p">{</span>
    <span class="nx">aws</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">source</span>  <span class="p">=</span> <span class="s2">"hashicorp/aws"</span>
      <span class="nx">version</span> <span class="p">=</span> <span class="s2">"~&gt; 5.0"</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">provider</span> <span class="s2">"aws"</span> <span class="p">{</span>
  <span class="nx">region</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">aws_region</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_sesv2_email_identity"</span> <span class="s2">"sender"</span> <span class="p">{</span>
  <span class="nx">email_identity</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">sender_email</span>
<span class="p">}</span>

<span class="nx">data</span> <span class="s2">"aws_iam_policy_document"</span> <span class="s2">"lambda_assume"</span> <span class="p">{</span>
  <span class="nx">statement</span> <span class="p">{</span>
    <span class="nx">actions</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"sts:AssumeRole"</span><span class="p">]</span>
    <span class="nx">principals</span> <span class="p">{</span>
      <span class="nx">type</span>        <span class="p">=</span> <span class="s2">"Service"</span>
      <span class="nx">identifiers</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"lambda.amazonaws.com"</span><span class="p">]</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_iam_role"</span> <span class="s2">"lambda"</span> <span class="p">{</span>
  <span class="nx">name</span>               <span class="p">=</span> <span class="s2">"${var.project_name}-role"</span>
  <span class="nx">assume_role_policy</span> <span class="p">=</span> <span class="nx">data</span><span class="p">.</span><span class="nx">aws_iam_policy_document</span><span class="p">.</span><span class="nx">lambda_assume</span><span class="p">.</span><span class="nx">json</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_iam_role_policy_attachment"</span> <span class="s2">"basic_logs"</span> <span class="p">{</span>
  <span class="nx">role</span>       <span class="p">=</span> <span class="nx">aws_iam_role</span><span class="p">.</span><span class="nx">lambda</span><span class="p">.</span><span class="nx">name</span>
  <span class="nx">policy_arn</span> <span class="p">=</span> <span class="s2">"arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"</span>
<span class="p">}</span>

<span class="nx">data</span> <span class="s2">"aws_iam_policy_document"</span> <span class="s2">"ses_send"</span> <span class="p">{</span>
  <span class="nx">statement</span> <span class="p">{</span>
    <span class="nx">effect</span>    <span class="p">=</span> <span class="s2">"Allow"</span>
    <span class="nx">actions</span>   <span class="p">=</span> <span class="p">[</span><span class="s2">"ses:SendEmail"</span><span class="p">]</span>
    <span class="nx">resources</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"*"</span><span class="p">]</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_iam_role_policy"</span> <span class="s2">"ses_send"</span> <span class="p">{</span>
  <span class="nx">name</span>   <span class="p">=</span> <span class="s2">"${var.project_name}-ses-send"</span>
  <span class="nx">role</span>   <span class="p">=</span> <span class="nx">aws_iam_role</span><span class="p">.</span><span class="nx">lambda</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">policy</span> <span class="p">=</span> <span class="nx">data</span><span class="p">.</span><span class="nx">aws_iam_policy_document</span><span class="p">.</span><span class="nx">ses_send</span><span class="p">.</span><span class="nx">json</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_lambda_function"</span> <span class="s2">"send_verification"</span> <span class="p">{</span>
  <span class="nx">function_name</span>    <span class="p">=</span> <span class="s2">"${var.project_name}-send"</span>
  <span class="nx">role</span>             <span class="p">=</span> <span class="nx">aws_iam_role</span><span class="p">.</span><span class="nx">lambda</span><span class="p">.</span><span class="nx">arn</span>
  <span class="nx">handler</span>          <span class="p">=</span> <span class="s2">"dist/handler.handler"</span>
  <span class="nx">filename</span>         <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">lambda_package_path</span>
  <span class="nx">source_code_hash</span> <span class="p">=</span> <span class="nx">filebase64sha256</span><span class="p">(</span><span class="nx">var</span><span class="p">.</span><span class="nx">lambda_package_path</span><span class="p">)</span>
  <span class="nx">runtime</span>          <span class="p">=</span> <span class="s2">"nodejs20.x"</span>
  <span class="nx">timeout</span>          <span class="p">=</span> <span class="mi">10</span>
  <span class="nx">environment</span> <span class="p">{</span>
    <span class="nx">variables</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">SENDER_EMAIL</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">sender_email</span>
      <span class="nx">SES_REGION</span>   <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">aws_region</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_lambda_function_url"</span> <span class="s2">"this"</span> <span class="p">{</span>
  <span class="nx">count</span>               <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">expose_function_url</span> <span class="err">?</span> <span class="mi">1</span> <span class="err">:</span> <span class="mi">0</span>
  <span class="nx">function_name</span>       <span class="p">=</span> <span class="nx">aws_lambda_function</span><span class="p">.</span><span class="nx">send_verification</span><span class="p">.</span><span class="nx">function_name</span>
  <span class="nx">authorization_type</span>  <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">function_url_auth_type</span>
  <span class="nx">cors</span> <span class="p">{</span>
    <span class="nx">allow_origins</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"*"</span><span class="p">]</span>
    <span class="nx">allow_methods</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"POST"</span><span class="p">,</span> <span class="s2">"OPTIONS"</span><span class="p">]</span>
    <span class="nx">allow_headers</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"content-type"</span><span class="p">]</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>Refs úteis: <code>aws_lambda_function</code>, <code>aws_lambda_function_url</code> e política básica de logs para Lambda. (<a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lambda_function?utm_source=chatgpt.com" rel="noopener noreferrer">Terraform Registry</a>, <a href="https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AWSLambdaBasicExecutionRole.html?utm_source=chatgpt.com" rel="noopener noreferrer">AWS Documentation</a>)</p>
</blockquote>

<h3>
  
  
  <code>outputs.tf</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">output</span> <span class="s2">"lambda_function_name"</span> <span class="p">{</span>
  <span class="nx">value</span> <span class="p">=</span> <span class="nx">aws_lambda_function</span><span class="p">.</span><span class="nx">send_verification</span><span class="p">.</span><span class="nx">function_name</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"lambda_function_arn"</span> <span class="p">{</span>
  <span class="nx">value</span> <span class="p">=</span> <span class="nx">aws_lambda_function</span><span class="p">.</span><span class="nx">send_verification</span><span class="p">.</span><span class="nx">arn</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"function_url"</span> <span class="p">{</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">try</span><span class="p">(</span><span class="nx">aws_lambda_function_url</span><span class="p">.</span><span class="nx">this</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">function_url</span><span class="p">,</span> <span class="kc">null</span><span class="p">)</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"URL pública opcional para testes"</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Passo 2 — Lambda em Node.js (AWS SDK v3 / SESv2)
</h2>

<p>Abaixo, um design simples, mas com <strong>separação de responsabilidades</strong>:</p>

<ul>
<li>
<code>SesEmailClient</code> encapsula o client do SESv2.</li>
<li>
<code>EmailService</code> define o caso de uso de envio de verificação.</li>
<li>
<code>config</code> centraliza variáveis de ambiente.</li>
<li>
<code>handler</code> expõe a função Lambda.</li>
</ul>

<h3>
  
  
  <code>app/package.json</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ses-verify-lambda"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"private"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"main"</span><span class="p">:</span><span class="w"> </span><span class="s2">"dist/handler.js"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"build"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tsc -p tsconfig.json"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"package"</span><span class="p">:</span><span class="w"> </span><span class="s2">"mkdir -p ../build &amp;&amp; zip -r ../build/lambda.zip dist node_modules package.json"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"dependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"@aws-sdk/client-sesv2"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^3.600.0"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"devDependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"@types/aws-lambda"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^8.10.136"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"typescript"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^5.5.4"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  <code>app/tsconfig.json</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"compilerOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"target"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ES2020"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"module"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CommonJS"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"moduleResolution"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Node"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"outDir"</span><span class="p">:</span><span class="w"> </span><span class="s2">"dist"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"strict"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"esModuleInterop"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"forceConsistentCasingInFileNames"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"skipLibCheck"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"include"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"src"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  <code>app/src/config.ts</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">env</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">region</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SES_REGION</span> <span class="o">||</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">AWS_REGION</span> <span class="o">||</span> <span class="dl">"</span><span class="s2">us-east-1</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">sender</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SENDER_EMAIL</span> <span class="o">||</span> <span class="dl">""</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  <code>app/src/dto.ts</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">type</span> <span class="nx">SendVerificationInput</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">to</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">code</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">subject</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">link</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  <code>app/src/email/SesEmailClient.ts</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">SESv2Client</span><span class="p">,</span> <span class="nx">SendEmailCommand</span><span class="p">,</span> <span class="nx">SendEmailCommandInput</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/client-sesv2</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">SesEmailClient</span> <span class="p">{</span>
  <span class="k">private</span> <span class="k">readonly</span> <span class="nx">client</span><span class="p">:</span> <span class="nx">SESv2Client</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="nx">region</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SESv2Client</span><span class="p">({</span> <span class="nx">region</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">send</span><span class="p">(</span><span class="nx">input</span><span class="p">:</span> <span class="nx">SendEmailCommandInput</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">client</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="k">new</span> <span class="nc">SendEmailCommand</span><span class="p">(</span><span class="nx">input</span><span class="p">));</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  <code>app/src/email/EmailService.ts</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">SesEmailClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./SesEmailClient</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SendVerificationInput</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">../dto</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">EmailService</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="k">readonly</span> <span class="nx">ses</span><span class="p">:</span> <span class="nx">SesEmailClient</span><span class="p">,</span> <span class="k">private</span> <span class="k">readonly</span> <span class="nx">sender</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{}</span>

  <span class="k">async</span> <span class="nf">sendVerification</span><span class="p">({</span> <span class="nx">to</span><span class="p">,</span> <span class="nx">code</span><span class="p">,</span> <span class="nx">subject</span><span class="p">,</span> <span class="nx">link</span> <span class="p">}:</span> <span class="nx">SendVerificationInput</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="k">this</span><span class="p">.</span><span class="nx">sender</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Missing sender</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">to</span> <span class="o">||</span> <span class="o">!</span><span class="nx">code</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Invalid payload</span><span class="dl">"</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">subj</span> <span class="o">=</span> <span class="nx">subject</span> <span class="o">||</span> <span class="dl">"</span><span class="s2">Verifique seu e-mail</span><span class="dl">"</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">html</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">buildHtml</span><span class="p">(</span><span class="nx">code</span><span class="p">,</span> <span class="nx">link</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">buildText</span><span class="p">(</span><span class="nx">code</span><span class="p">,</span> <span class="nx">link</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">ses</span><span class="p">.</span><span class="nf">send</span><span class="p">({</span>
      <span class="na">FromEmailAddress</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">sender</span><span class="p">,</span>
      <span class="na">Destination</span><span class="p">:</span> <span class="p">{</span> <span class="na">ToAddresses</span><span class="p">:</span> <span class="p">[</span><span class="nx">to</span><span class="p">]</span> <span class="p">},</span>
      <span class="na">Content</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">Simple</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">Subject</span><span class="p">:</span> <span class="p">{</span> <span class="na">Data</span><span class="p">:</span> <span class="nx">subj</span><span class="p">,</span> <span class="na">Charset</span><span class="p">:</span> <span class="dl">"</span><span class="s2">UTF-8</span><span class="dl">"</span> <span class="p">},</span>
          <span class="na">Body</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">Html</span><span class="p">:</span> <span class="p">{</span> <span class="na">Data</span><span class="p">:</span> <span class="nx">html</span><span class="p">,</span> <span class="na">Charset</span><span class="p">:</span> <span class="dl">"</span><span class="s2">UTF-8</span><span class="dl">"</span> <span class="p">},</span>
            <span class="na">Text</span><span class="p">:</span> <span class="p">{</span> <span class="na">Data</span><span class="p">:</span> <span class="nx">text</span><span class="p">,</span> <span class="na">Charset</span><span class="p">:</span> <span class="dl">"</span><span class="s2">UTF-8</span><span class="dl">"</span> <span class="p">}</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">private</span> <span class="nf">buildHtml</span><span class="p">(</span><span class="nx">code</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">link</span><span class="p">?:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">callToAction</span> <span class="o">=</span> <span class="nx">link</span> <span class="p">?</span> <span class="s2">`&lt;p&gt;&lt;a href="</span><span class="p">${</span><span class="nx">link</span><span class="p">}</span><span class="s2">"&gt;Confirmar conta&lt;/a&gt;&lt;/p&gt;`</span> <span class="p">:</span> <span class="dl">""</span><span class="p">;</span>
    <span class="k">return</span> <span class="s2">`&lt;h1&gt;Seu código&lt;/h1&gt;&lt;p&gt;</span><span class="p">${</span><span class="nx">code</span><span class="p">}</span><span class="s2">&lt;/p&gt;</span><span class="p">${</span><span class="nx">callToAction</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">private</span> <span class="nf">buildText</span><span class="p">(</span><span class="nx">code</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">link</span><span class="p">?:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">callToAction</span> <span class="o">=</span> <span class="nx">link</span> <span class="p">?</span> <span class="s2">`\n\nConfirmar: </span><span class="p">${</span><span class="nx">link</span><span class="p">}</span><span class="s2">`</span> <span class="p">:</span> <span class="dl">""</span><span class="p">;</span>
    <span class="k">return</span> <span class="s2">`Seu código: </span><span class="p">${</span><span class="nx">code</span><span class="p">}${</span><span class="nx">callToAction</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  <code>app/src/handler.ts</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">env</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./config</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">EmailService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./email/EmailService</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SesEmailClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./email/SesEmailClient</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SendVerificationInput</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./dto</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">service</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">EmailService</span><span class="p">(</span><span class="k">new</span> <span class="nc">SesEmailClient</span><span class="p">(</span><span class="nx">env</span><span class="p">.</span><span class="nx">region</span><span class="p">),</span> <span class="nx">env</span><span class="p">.</span><span class="nx">sender</span><span class="p">);</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">body</span> <span class="o">=</span> <span class="k">typeof</span> <span class="nx">event</span><span class="p">?.</span><span class="nx">body</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">string</span><span class="dl">"</span> <span class="p">?</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">body</span><span class="p">)</span> <span class="p">:</span> <span class="nx">event</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">payload</span> <span class="o">=</span> <span class="nx">body</span> <span class="k">as</span> <span class="nx">SendVerificationInput</span><span class="p">;</span>
  <span class="k">await</span> <span class="nx">service</span><span class="p">.</span><span class="nf">sendVerification</span><span class="p">(</span><span class="nx">payload</span><span class="p">);</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="na">statusCode</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">content-type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span> <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">ok</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
  <span class="p">};</span>
<span class="p">};</span>
</code></pre>

</div>



<blockquote>
<p>A operação usada é <code>SendEmail</code> do <strong>SESv2</strong>; no SDK v3, montamos <code>Destination</code>, <code>Content.Simple</code> e <code>FromEmailAddress</code> como no exemplo acima. (<a href="https://docs.aws.amazon.com/goto/SdkForJavaScriptV3/sesv2-2019-09-27/SendEmail?utm_source=chatgpt.com" rel="noopener noreferrer">AWS Documentation</a>)</p>
</blockquote>




<h2>
  
  
  Passo 3 — Build do código e empacotamento
</h2>

<p>Dentro de <code>app/</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm ci
npm run build
npm run package
</code></pre>

</div>



<p>Isso gera <code>build/lambda.zip</code>, que o Terraform referenciará.</p>




<h2>
  
  
  Passo 4 — Deploy com Terraform
</h2>

<p>Na raiz do projeto:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>terraform <span class="nt">-chdir</span><span class="o">=</span>infra init
terraform <span class="nt">-chdir</span><span class="o">=</span>infra apply <span class="se">\</span>
  <span class="nt">-var</span><span class="o">=</span><span class="s1">'aws_region=us-east-1'</span> <span class="se">\</span>
  <span class="nt">-var</span><span class="o">=</span><span class="s1">'sender_email=seu-remetente@exemplo.com'</span> <span class="se">\</span>
  <span class="nt">-var</span><span class="o">=</span><span class="s1">'lambda_package_path=../build/lambda.zip'</span> <span class="se">\</span>
  <span class="nt">-var</span><span class="o">=</span><span class="s1">'expose_function_url=true'</span>
</code></pre>

</div>



<p>O SES vai enviar um <strong>e-mail de verificação</strong> para o remetente informado. Clique no link recebido para confirmar o <strong>Email Identity</strong> antes de testar o envio. Em <strong>sandbox</strong>, o <strong>destinatário também precisa estar verificado</strong> (ou use o <strong>mailbox simulator</strong>). Para liberar envio geral, faça o pedido de <strong>produção</strong>. (<a href="https://docs.aws.amazon.com/ses/latest/dg/verify-addresses-and-domains.html?utm_source=chatgpt.com" rel="noopener noreferrer">AWS Documentation</a>)</p>




<h2>
  
  
  Passo 5 — Testes
</h2>

<h3>
  
  
  Opção A: invocar direto a Lambda (AWS CLI)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">FUNC_NAME</span><span class="o">=</span><span class="si">$(</span>terraform <span class="nt">-chdir</span><span class="o">=</span>infra output <span class="nt">-raw</span> lambda_function_name<span class="si">)</span>

aws lambda invoke <span class="se">\</span>
  <span class="nt">--function-name</span> <span class="s2">"</span><span class="nv">$FUNC_NAME</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">--cli-binary-format</span> raw-in-base64-out <span class="se">\</span>
  <span class="nt">--payload</span> <span class="s1">'{"to":"destinatario@exemplo.com","code":"123456","subject":"Verifique seu e-mail","link":"https://minhaapp/verify?code=123456"}'</span> <span class="se">\</span>
  /dev/stdout
</code></pre>

</div>



<h3>
  
  
  Opção B: chamar via Function URL (curl/Postman)
</h3>

<p>Se você ativou <code>expose_function_url=true</code>, capture a URL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">URL</span><span class="o">=</span><span class="si">$(</span>terraform <span class="nt">-chdir</span><span class="o">=</span>infra output <span class="nt">-raw</span> function_url<span class="si">)</span>
curl <span class="nt">-s</span> <span class="nt">-X</span> POST <span class="s2">"</span><span class="nv">$URL</span><span class="s2">"</span> <span class="nt">-H</span> <span class="s1">'content-type: application/json'</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"to":"destinatario@exemplo.com","code":"789012"}'</span>
</code></pre>

</div>



<blockquote>
<p>Function URLs são um atalho elegante para testar HTTP sem API Gateway. Para produção, considere autenticação (AWS_IAM), WAF/CloudFront e políticas mais restritivas. (<a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lambda_function_url.html?utm_source=chatgpt.com" rel="noopener noreferrer">Terraform Registry</a>, <a href="https://hedrange.com/2024/05/03/develop-lightweight-and-secure-rest-apis-with-aws-lambda-function-url-and-terraform/?utm_source=chatgpt.com" rel="noopener noreferrer">Håkon Eriksen Drange - Perspectives</a>)</p>
</blockquote>

<p>Se tudo deu certo, o e-mail chegará no destino permitido pelo seu status do SES.</p>




<h2>
  
  
  Próximos passos e boas práticas
</h2>

<ul>
<li>
<strong>Sair do sandbox</strong> quando estiver pronto para enviar a qualquer domínio. (<a href="https://docs.aws.amazon.com/ses/latest/dg/request-production-access.html?utm_source=chatgpt.com" rel="noopener noreferrer">AWS Documentation</a>)</li>
<li>
<strong>Domínio verificado + DKIM/DMARC</strong> para reputação e entregabilidade melhores (em vez de só endereço). (<a href="https://docs.aws.amazon.com/ses/latest/dg/verify-addresses-and-domains.html?utm_source=chatgpt.com" rel="noopener noreferrer">AWS Documentation</a>)</li>
<li>
<strong>Templates</strong> e métricas com Configuration Sets, IP pools gerenciados, etc., se precisar escalar e observar. (<a href="https://stackoverflow.com/questions/77821006/use-a-managed-ses-ip-pool-in-terraform-by-default?utm_source=chatgpt.com" rel="noopener noreferrer">Stack Overflow</a>)</li>
<li>
<strong>Segurança</strong>: se usar Function URL em produção, avalie <code>AWS_IAM</code>, WAF e CDN com origem na URL. (<a href="https://hedrange.com/2024/05/03/develop-lightweight-and-secure-rest-apis-with-aws-lambda-function-url-and-terraform/?utm_source=chatgpt.com" rel="noopener noreferrer">Håkon Eriksen Drange - Perspectives</a>)</li>
</ul>




