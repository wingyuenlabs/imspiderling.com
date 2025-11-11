---
Title: RESTful - Pílula 2 – Estrutura e hierarquia de URLs
Description: 
Author: Anderson Contreira
Date: 2025-11-11T22:04:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>Uma API RESTful bem projetada deve ter <strong>URLs organizadas e previsíveis</strong>, que representem a <strong>relação entre recursos</strong>.<br>
A estrutura das rotas comunica a hierarquia e o contexto de cada entidade dentro do domínio da aplicação.</p>
<h2>
  
  
  Representando coleções e instâncias
</h2>

<p>Como vimos na pílula anterior, recursos são representados por <strong>coleções</strong> e <strong>instâncias</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GET /users         → coleção de usuários  
GET /users/10      → um usuário específico  
</code></pre>

</div>



<p>Essa convenção pode ser aplicada a qualquer entidade do sistema (<code>posts</code>, <code>orders</code>, <code>companies</code>, <code>products</code> etc.).</p>

<h2>
  
  
  Hierarquia entre recursos
</h2>

<p>Quando um recurso depende de outro, por exemplo, <strong>pedidos de um usuário</strong>, podemos representar essa relação de forma hierárquica:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GET /users/10/orders           → lista de pedidos do usuário 10  
GET /users/10/orders/99        → pedido 99 do usuário 10  
</code></pre>

</div>



<p>Isso mostra claramente o <strong>contexto</strong> do recurso: o pedido pertence ao usuário.</p>

<p>Outra dica importante de segurança é: </p>

<blockquote>
<p>Procure usar UUIDs em vez de IDs diretamente nas URLs; isso previne ações de pessoas mal-intencionadas.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GET /users/&lt;uuid&gt;/orders           → lista de pedidos do usuário  
GET /users/&lt;uuid&gt;/orders/&lt;uuid&gt;        → pedido X do usuário Y  
</code></pre>

</div>



<h2>
  
  
  Quando (e quando não) usar hierarquia
</h2>

<p>✅ Use hierarquia quando o sub-recurso <strong>não faz sentido existir fora do contexto do pai</strong>.<br>
Exemplo:</p>

<ul>
<li>
<code>/users/10/orders</code> → o pedido pertence a um usuário específico.</li>
</ul>

<p>❌ Evite hierarquias profundas, como:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/companies/1/departments/2/employees/3/projects/4/tasks/5
</code></pre>

</div>



<blockquote>
<p>Isso dificulta manutenção e compreensão. Nesse caso, prefira algo mais direto:<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/tasks/5
</code></pre>

</div>



<p>E, se necessário, use <strong>filtros</strong> ou <strong>relacionamentos</strong> na query string:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/tasks?userId=3
</code></pre>

</div>



<h2>
  
  
  URLs limpas e consistentes
</h2>

<p>Boas práticas de formatação:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Regra</th>
<th>Exemplo</th>
</tr>
</thead>
<tbody>
<tr>
<td>Use <strong>letras minúsculas</strong>
</td>
<td>
<code>/users</code>, não <code>/Users</code>
</td>
</tr>
<tr>
<td>Separe palavras com <strong>hífen (-)</strong>
</td>
<td>
<code>/user-profiles</code>, não <code>/user_profiles</code>
</td>
</tr>
<tr>
<td>Evite sufixos técnicos</td>
<td>
<code>/users</code>, não <code>/users.json</code>
</td>
</tr>
<tr>
<td>Evite barras no final</td>
<td>
<code>/users</code>, não <code>/users/</code>
</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Resumo
</h2>

<ul>
<li>Use <strong>coleções</strong> para grupos e <strong>instâncias</strong> para itens específicos</li>
<li>Represente relações com <strong>hierarquias simples e intuitivas</strong>
</li>
<li>Evite <strong>profundidade excessiva</strong> nas URLs</li>
<li>Mantenha <strong>consistência e legibilidade</strong>
</li>
</ul>

