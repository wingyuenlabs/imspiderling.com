---
Title: Controlando cache através do navegador
Description: 
Author: Matheus Silva
Date: 2025-12-12T21:22:10.000Z
Robots: noindex,nofollow
Template: index
---
<p><code>Cache</code> você já deve ter ouvido esse termo várias vezes e recentemente resolvi aprender como o navegador implementa esse mecanismo e então resolvi escrever esse post sobre o que aprendi.</p>

<p>Contextualizando, <code>Cache</code> é um mecanismo utilizado com a finalidade de trazer velocidade ao acesso um conteúdo que é frequentemente utilizado.</p>

<p>Quando estamos implementando cache em nosso sistema é importante mantê-lo o mais próximo possível do cliente e para isso podemos usar o sistema de cache dos navegadores assim mantendo velocidade no acesso ao conteúdo e salvando recursos de nossos servidores.</p>

<h2>
  
  
  Tipos de cache
</h2>

<p>Para gerenciar cache no navegador usamos o header <code>Cache-Control</code> e podemos trabalhar com os seguintes valores para ele:</p>

<p><code>default</code></p>

<p>Por padrão o seguinte fluxo será seguido:</p>

<ul>
<li>Se for encontrada em cache uma versão atualizada dos dados essa será retornada.</li>
<li>Caso seja obsoleta será feita uma espécie de consulta ao servidor e verificar se os dados do cache sofreram alterações, se sofreram alteração os dados atualizados vão ser retornados e atualizados no cache, caso contrário o dado do cache será retornado.</li>
<li>Se não for encontrado nada em cache será feita a requisição ao servidor e o cache vai ser atualizado com os dados retornados.</li>
</ul>

<p><code>no-store</code></p>

<p>O navegador vai ignorar o cache, efetuar a requisição para o servidor e não irá armazenar o retorno no cache.</p>

<p><code>reload</code></p>

<p>O navegador vai ignorar o cache, efetuar a requisição para o servidor e vai atualizar o cache com o retorno da requisição.</p>

<p><code>no-cache</code></p>

<p>O navegador verifica no cache se existe alguma versão atualizada dos dados ou obsoleta depois disso é verificado com o servidor se há uma nova versão dos dados, se houver será retornado a nova versão pelo servidor e o cache será atualizado, se não a versão do cache vai ser retornada.</p>

<p><code>force-cache</code></p>

<p>O navegador verifica o cache, se encontrar uma versão atualizada ou obsoleta dos dados ela vai ser retornada se não encontrar nada no cache ele vai fazer a requisição normalmente no servidor retornando os dados e salvando no cache.</p>

<p><code>max-age=&lt;valor em segundos&gt;</code></p>

<p>Pode ser usado em conjunto com os valores acima e define por quanto tempo aquele dado é definido como “atualizado”.</p>

<p>Também e possível gerenciar o cache usando o header <code>Expires</code> que pode ser definido com um tempo específico, exemplo: <code>Expires: Thu, 25 Dec 2025 18:18:18 GMT</code> ou seja a partir do tempo especificado aquele dado passa a ser obsoleto.</p>

<p>Esse foi o conteúdo que estudei espero que tenha agregado de alguma forma. 😊 </p>

<p>Referências: <br>
<a href="https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Guides/Caching" rel="noopener noreferrer">https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Guides/Caching</a><br>
<a href="https://developer.mozilla.org/en-US/docs/Web/API/Request/cache" rel="noopener noreferrer">https://developer.mozilla.org/en-US/docs/Web/API/Request/cache</a></p>

