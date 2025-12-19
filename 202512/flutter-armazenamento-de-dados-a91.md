---
Title: Flutter: Armazenamento de dados
Description: 
Author: Willane Paiva
Date: 2025-12-19T21:41:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>O flutter permite armazenar dados no dispositivo em que o aplicativo está instalado, podendo esse dado ter diferentes finalidades, como mostrar ou não um pop-up promocional, salvar preferências (como tema a usar). Este artigo explicará a diferença entre dados simples e criptografados, além de apresentar os principais pacotes que podem ser usado para cada tipo.</p>

<h2>
  
  
  Tipos de dados
</h2>

<h3>
  
  
  Dados simples
</h3>

<p>Dados simples seriam dados que não necessitam de segurança ao armazenar, podendo ser texto simples, exemplo seriam preferências do usuário, visualização de determinado dado. Esses dados são facilmente acessados, como no android que é possível acessar ao rodar o app em modo debug.</p>

<h3>
  
  
  Dados criptografados
</h3>

<blockquote>
<p>Criptografia seria codificar o dado de forma humanamente ilegível, usando um algoritmo</p>
</blockquote>

<p>Quando é usado dados sensíveis, é recomendado usar um armazenamento que criptografe os dados antes de salvar no dispositivo, de forma a manter o dado seguro, exemplo desse uso seriam informações do usuário. Para isso pode criptografar o dado e armazenar ou usar um pacote que faça isso, como o <code>flutter_secure_storage</code>.</p>

<h2>
  
  
  Principais pacotes
</h2>

<h3>
  
  
  SharedPreferences
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fszlwmlvhgwnkgtjgies0.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fszlwmlvhgwnkgtjgies0.jpg" alt=" " width="800" height="287"></a></p>

<p>O SharedPreferences permite o armazenado de dados de diferentes tipos primitivos sem criptografia, é um pacote recomendado pelo time do flutter. A instalação permite configurar propriedades durante sua implementação.<br>
As operações com os dados é feita tanto de forma síncrona como assíncrona, a depender do construtor usado na configuração.<br>
Um ponto de atenção é que não existe garantia de que o dado salvo será persistido, não sendo assim recomendado para dados críticos.</p>

<p><a href="https://pub.dev/packages/shared_preferences" rel="noopener noreferrer">Link do pacote</a></p>

<h3>
  
  
  FlutterSecureStorage
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fikfjiiq52mkhcrewo5f5.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fikfjiiq52mkhcrewo5f5.jpg" alt=" " width="800" height="362"></a></p>

<p>O FlutterSecureStorage armazena os dados de forma segura, usando o keychain (iOS) e keyStore (android), assim os dados são salvos criptografados no dispositivo.<br>
As operações com os dados é feita de forma assíncrona.</p>

<p><a href="https://pub.dev/packages/flutter_secure_storage" rel="noopener noreferrer">Link do pacote</a></p>

<h2>
  
  
  Considerações finais
</h2>

<p>Nesse artigo foi abordado o conceito de dados não sensíveis e criptografados, apresentando os pacotes <code>SharedPreferences</code> e FlutterSecureStorage` como alternativas para salvar os dados em dispositivos. Além desses pacotes, também existe a possibilidade de guardar dados mais complexos usando banco de dados, o qual a escolha vai depender das especificações do tipo e uso do dado.</p>

