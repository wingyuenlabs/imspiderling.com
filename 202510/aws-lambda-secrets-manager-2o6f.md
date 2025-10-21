---
Title: AWS Lambda + Secrets Manager
Description: 
Author: Eduardo Garcia
Date: 2025-10-21T21:29:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>Funções <strong>AWS Lambda</strong> são opções extremamente práticas que simplificam todo o processo de disponibilização de aplicações <em>serverless</em>.<br>
Ao mesmo tempo que geram praticidade, também podem criar péssimos hábitos em nós, desenvolvedores.</p>

<p>Um desses maus costumes é a exposição de <strong>chaves de API</strong> externas ou <strong>conexões de banco de dados</strong> diretamente dentro do corpo de uma função Lambda. São incontáveis as vezes em que encontrei códigos semelhantes ao mostrado abaixo:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpba40w502myql3h3unta.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpba40w502myql3h3unta.png" alt="Bad practice" width="510" height="177"></a></p>

<p>Nem é preciso dizer o quão ruim é manter suas configurações assim! Essa exposição de chaves é extremamente prejudicial por alguns fatores, dentre eles:</p>

<ul>
<li>
<strong>Vulnerabilidade da aplicação:</strong> qualquer vazamento de código, seja em um repositório público ou por um ataque, expõe suas credenciais ao mundo externo;</li>
<li>
<strong>Falta de escalabilidade:</strong> caso seja necessário trocar de conexão com a base ou atualizar um par de API Keys, será preciso publicar novamente o código para que ele funcione. Em uma única Lambda isso é simples, mas quando falamos de funções críticas com múltiplas conexões, a situação complica.</li>
</ul>

<p>Pensando nesses pontos, a melhor forma de manter suas chaves seguras é <strong>externalizar o armazenamento e o gerenciamento</strong> delas.<br>
E é exatamente para isso que a AWS disponibiliza o <strong>AWS Secrets Manager</strong>, um serviço voltado para o controle e a centralização de credenciais de forma segura.</p>




<h3>
  
  
  1. Configurando um conjunto de chaves na AWS
</h3>

<ol>
<li>Em sua conta da AWS, acesse o serviço <a href="https://us-east-1.console.aws.amazon.com/secretsmanager/landing" rel="noopener noreferrer">AWS Secrets Manager</a> e escolha a opção <strong>“Store a new secret”</strong>;</li>
<li>Em <strong>Secret Type</strong>, selecione <strong>“Other type of secret”</strong> e preencha todas as chaves necessárias. Para este exemplo, foram criadas apenas duas. Em seguida, pressione <strong>Next</strong>.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhusii4h2n8t62b5xfp5b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhusii4h2n8t62b5xfp5b.png" alt="Secret Configuration" width="800" height="304"></a></p>

<p>Na próxima tela, escolha um nome descritivo para o conjunto de chaves.<br><br>
Depois, basta confirmar as etapas seguintes. O conjunto de <em>secrets</em> ficará disponível instantaneamente e pode ser acessado no console em <strong>Overview → Retrieve Secret Value</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9cx41aby5sbctdhiubev.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9cx41aby5sbctdhiubev.png" alt="Secret Informations" width="800" height="178"></a></p>

<h2>
  
  
  Simples assim: suas <em>secrets</em> estão criadas e já podem ser usadas — seja em aplicações via <strong>SDK</strong>, seja através do <strong>CLI</strong>.
</h2>

<h3>
  
  
  2. Buscando as chaves dentro da Lambda
</h3>

<p>Dentro de uma função Lambda, o uso das chaves é muito simples. No exemplo abaixo, estou utilizando o framework <strong>AWS SAM</strong>, configurado com <strong>Python 3.10</strong> — uma versão que já inclui o SDK da AWS.</p>

<p>Com o pequeno bloco de código a seguir, é possível acessar as chaves configuradas:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyf60q834hv0eckq5d8s0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyf60q834hv0eckq5d8s0.png" alt="Secret Usage on AWS Lambda" width="800" height="480"></a></p>

<p>O parâmetro principal da função <code>parameters.get_secret()</code> é o <strong>nome</strong> atribuído ao conjunto de chaves. Os outros dois parâmetros são opcionais: um define o formato dos dados retornados e o outro o tempo de cache em segundos.</p>

<p>Tudo certo, certo? Quase!<br>
Provavelmente, ao rodar sua Lambda — seja no ambiente de desenvolvimento ou em produção — você receberá um erro de <strong>permissão negada</strong>.<br><br>
Isso acontece porque é necessário conceder à função permissão para <strong>ler chaves no Secrets Manager</strong>.</p>

<p>Essa permissão pode ser adicionada diretamente no <strong>template.yaml</strong> (no caso do AWS SAM), ou atribuída ao usuário/role que executa a função localmente via <strong>CLI</strong>. Na imagem abaixo, temos um exemplo de política que permite à função acessar o Secrets Manager:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy3aamy1x0nj2kipq3m74.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy3aamy1x0nj2kipq3m74.png" alt="Secrets Manager policy" width="660" height="340"></a></p>




<h3>
  
  
  3. Conclusão
</h3>

<p>Com esse setup feito, você está pronto para usar suas chaves de forma segura em funções Lambda! O uso do <strong>AWS Secrets Manager</strong> traz um ganho significativo de <strong>controle e escalabilidade</strong>, centralizando as credenciais dentro de um ambiente seguro e fácil de manter.  </p>

<p>Além disso, o serviço permite configurar <strong>rotação automática de chaves</strong>, reduzindo o risco de vazamentos e simplificando ainda mais o gerenciamento de credenciais sensíveis.  </p>

<p>Vale lembrar que, como todo serviço da AWS, o Secrets Manager possui um custo.<br><br>
Antes de implementá-lo em larga escala, consulte os valores <a href="https://aws.amazon.com/pt/secrets-manager/pricing/" rel="noopener noreferrer">nesta página</a>.</p>

<p>Abaixo está o link para meu repositório no GitHub, onde disponibilizo um projeto de exemplo com o uso do Secrets Manager em uma função Lambda criada com AWS SAM:</p>

<p>🔗 <a href="https://github.com/Eudu4rdo/sam-lambda-pipeline" rel="noopener noreferrer">Eudu4rdo/AWS-SAM-Projects</a></p>




<p><strong>Em resumo:</strong><br><br>
Ao adotar o Secrets Manager, você não apenas protege suas credenciais, mas também eleva o nível de segurança, profissionalismo e maturidade do seu ambiente <em>serverless</em>.</p>

