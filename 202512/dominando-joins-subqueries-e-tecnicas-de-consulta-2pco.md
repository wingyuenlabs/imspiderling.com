---
Title: Dominando JOINs, Subqueries e Técnicas de Consulta
Description: 
Author: Kauê Matos
Date: 2025-12-20T21:22:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>O Oracle Database oferece recursos poderosos para consultar e relacionar dados. Este guia aborda em profundidade INNER JOINs, subqueries e técnicas avançadas de consulta SQL no Oracle.</p>

<h2>
  
  
  1. INNER JOIN
</h2>

<h3>
  
  
  1.1 Conceito Fundamental
</h3>

<p>O INNER JOIN retorna apenas as linhas que possuem correspondência em ambas as tabelas envolvidas na junção. É o tipo de JOIN mais comum e forma a base para relacionamentos entre tabelas.</p>

<h3>
  
  
  1.2 Sintaxe Básica
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">colunas</span>
<span class="k">FROM</span> <span class="n">tabela1</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">tabela2</span> <span class="k">ON</span> <span class="n">tabela1</span><span class="p">.</span><span class="n">coluna</span> <span class="o">=</span> <span class="n">tabela2</span><span class="p">.</span><span class="n">coluna</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  1.3 Exemplos Práticos
</h3>

<p><strong>Exemplo 1: Relacionando Funcionários e Departamentos</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">last_name</span><span class="p">,</span>
    <span class="n">d</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Exemplo 2: INNER JOIN com Múltiplas Condições</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">o</span><span class="p">.</span><span class="n">order_id</span><span class="p">,</span>
    <span class="k">c</span><span class="p">.</span><span class="n">customer_name</span><span class="p">,</span>
    <span class="n">p</span><span class="p">.</span><span class="n">product_name</span>
<span class="k">FROM</span> <span class="n">orders</span> <span class="n">o</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">customers</span> <span class="k">c</span> <span class="k">ON</span> <span class="n">o</span><span class="p">.</span><span class="n">customer_id</span> <span class="o">=</span> <span class="k">c</span><span class="p">.</span><span class="n">customer_id</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">order_items</span> <span class="n">oi</span> <span class="k">ON</span> <span class="n">o</span><span class="p">.</span><span class="n">order_id</span> <span class="o">=</span> <span class="n">oi</span><span class="p">.</span><span class="n">order_id</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">products</span> <span class="n">p</span> <span class="k">ON</span> <span class="n">oi</span><span class="p">.</span><span class="n">product_id</span> <span class="o">=</span> <span class="n">p</span><span class="p">.</span><span class="n">product_id</span>
<span class="k">WHERE</span> <span class="n">o</span><span class="p">.</span><span class="n">order_date</span> <span class="o">&gt;=</span> <span class="nb">DATE</span> <span class="s1">'2024-01-01'</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  1.4 INNER JOIN vs Sintaxe Antiga (Oracle)
</h3>

<p>O Oracle ainda suporta a sintaxe antiga de JOIN, mas a sintaxe ANSI (INNER JOIN) é recomendada:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Sintaxe antiga (não recomendada)</span>
<span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span> <span class="n">d</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span><span class="p">,</span> <span class="n">departments</span> <span class="n">d</span>
<span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span><span class="p">;</span>

<span class="c1">-- Sintaxe moderna (recomendada)</span>
<span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span> <span class="n">d</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  1.5 INNER JOIN com Aliases e Funções
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="k">UPPER</span><span class="p">(</span><span class="n">e</span><span class="p">.</span><span class="n">last_name</span><span class="p">)</span> <span class="k">AS</span> <span class="n">sobrenome</span><span class="p">,</span>
    <span class="n">d</span><span class="p">.</span><span class="n">department_name</span> <span class="k">AS</span> <span class="n">departamento</span><span class="p">,</span>
    <span class="n">TO_CHAR</span><span class="p">(</span><span class="n">e</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span> <span class="s1">'L99G999D99'</span><span class="p">)</span> <span class="k">AS</span> <span class="n">salario_formatado</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span> <span class="o">&gt;</span> <span class="mi">5000</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  2. Subqueries (Subconsultas)
</h2>

<h3>
  
  
  2.1 O Que São Subqueries?
</h3>

<p>Subqueries são consultas SQL aninhadas dentro de outra consulta. Elas permitem resolver problemas complexos dividindo-os em partes menores e mais gerenciáveis.</p>

<h3>
  
  
  2.2 Tipos de Subqueries
</h3>

<h4>
  
  
  2.2.1 Subquery Escalar (Retorna um Único Valor)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">salary</span>
<span class="k">FROM</span> <span class="n">employees</span>
<span class="k">WHERE</span> <span class="n">salary</span> <span class="o">&gt;</span> <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span><span class="p">);</span>
</code></pre>

</div>



<h4>
  
  
  2.2.2 Subquery de Múltiplas Linhas
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">department_id</span>
<span class="k">FROM</span> <span class="n">employees</span>
<span class="k">WHERE</span> <span class="n">department_id</span> <span class="k">IN</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">department_id</span> 
    <span class="k">FROM</span> <span class="n">departments</span> 
    <span class="k">WHERE</span> <span class="n">location_id</span> <span class="o">=</span> <span class="mi">1700</span>
<span class="p">);</span>
</code></pre>

</div>



<h4>
  
  
  2.2.3 Subquery Correlacionada
</h4>

<p>Uma subquery correlacionada referencia colunas da query externa:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span> <span class="o">&gt;</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">e2</span><span class="p">.</span><span class="n">salary</span><span class="p">)</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e2</span>
    <span class="k">WHERE</span> <span class="n">e2</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span>
<span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  2.3 Subqueries na Cláusula FROM (Inline Views)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">dept_avg</span><span class="p">.</span><span class="n">department_id</span><span class="p">,</span> <span class="n">dept_avg</span><span class="p">.</span><span class="n">media_salarial</span><span class="p">,</span> <span class="n">d</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">department_id</span><span class="p">,</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">AS</span> <span class="n">media_salarial</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">GROUP</span> <span class="k">BY</span> <span class="n">department_id</span>
<span class="p">)</span> <span class="n">dept_avg</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">dept_avg</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">WHERE</span> <span class="n">dept_avg</span><span class="p">.</span><span class="n">media_salarial</span> <span class="o">&gt;</span> <span class="mi">8000</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  2.4 Subqueries na Cláusula SELECT
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span>
    <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span><span class="p">)</span> <span class="k">AS</span> <span class="n">media_geral</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">salary</span> <span class="o">-</span> <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span><span class="p">)</span> <span class="k">AS</span> <span class="n">diferenca_media</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  2.5 Operadores com Subqueries
</h3>

<h4>
  
  
  EXISTS
</h4>

<p>Verifica se a subquery retorna algum resultado:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">d</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="n">departments</span> <span class="n">d</span>
<span class="k">WHERE</span> <span class="k">EXISTS</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="mi">1</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
    <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
    <span class="k">AND</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span> <span class="o">&gt;</span> <span class="mi">10000</span>
<span class="p">);</span>
</code></pre>

</div>



<h4>
  
  
  NOT EXISTS
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">d</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="n">departments</span> <span class="n">d</span>
<span class="k">WHERE</span> <span class="k">NOT</span> <span class="k">EXISTS</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="mi">1</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
    <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
<span class="p">);</span>
</code></pre>

</div>



<h4>
  
  
  ANY / SOME
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">salary</span>
<span class="k">FROM</span> <span class="n">employees</span>
<span class="k">WHERE</span> <span class="n">salary</span> <span class="o">&gt;</span> <span class="k">ANY</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">salary</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">WHERE</span> <span class="n">department_id</span> <span class="o">=</span> <span class="mi">50</span>
<span class="p">);</span>
</code></pre>

</div>



<h4>
  
  
  ALL
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">salary</span>
<span class="k">FROM</span> <span class="n">employees</span>
<span class="k">WHERE</span> <span class="n">salary</span> <span class="o">&gt;</span> <span class="k">ALL</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">salary</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">WHERE</span> <span class="n">department_id</span> <span class="o">=</span> <span class="mi">50</span>
<span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  3. Combinando INNER JOIN e Subqueries
</h2>

<h3>
  
  
  3.1 Exemplo Avançado
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">first_name</span> <span class="o">||</span> <span class="s1">' '</span> <span class="o">||</span> <span class="n">e</span><span class="p">.</span><span class="n">last_name</span> <span class="k">AS</span> <span class="n">nome_completo</span><span class="p">,</span>
    <span class="n">d</span><span class="p">.</span><span class="n">department_name</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span>
    <span class="n">dept_stats</span><span class="p">.</span><span class="n">media_dept</span><span class="p">,</span>
    <span class="n">dept_stats</span><span class="p">.</span><span class="n">max_dept</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="p">(</span>
    <span class="k">SELECT</span> 
        <span class="n">department_id</span><span class="p">,</span>
        <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">AS</span> <span class="n">media_dept</span><span class="p">,</span>
        <span class="k">MAX</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">AS</span> <span class="n">max_dept</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">GROUP</span> <span class="k">BY</span> <span class="n">department_id</span>
<span class="p">)</span> <span class="n">dept_stats</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">dept_stats</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span> <span class="o">&gt;</span> <span class="n">dept_stats</span><span class="p">.</span><span class="n">media_dept</span> <span class="o">*</span> <span class="mi">1</span><span class="p">.</span><span class="mi">2</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  3.2 Subquery com JOIN Múltiplo
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span>
    <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span>
    <span class="n">d</span><span class="p">.</span><span class="n">department_name</span><span class="p">,</span>
    <span class="n">l</span><span class="p">.</span><span class="n">city</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">locations</span> <span class="n">l</span> <span class="k">ON</span> <span class="n">d</span><span class="p">.</span><span class="n">location_id</span> <span class="o">=</span> <span class="n">l</span><span class="p">.</span><span class="n">location_id</span>
<span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span> <span class="o">&gt;</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">e2</span><span class="p">.</span><span class="n">salary</span><span class="p">)</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e2</span>
    <span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d2</span> <span class="k">ON</span> <span class="n">e2</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d2</span><span class="p">.</span><span class="n">department_id</span>
    <span class="k">WHERE</span> <span class="n">d2</span><span class="p">.</span><span class="n">location_id</span> <span class="o">=</span> <span class="n">l</span><span class="p">.</span><span class="n">location_id</span>
<span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  4. Common Table Expressions (WITH Clause)
</h2>

<p>As CTEs oferecem uma alternativa mais legível às subqueries complexas:</p>

<h3>
  
  
  4.1 CTE Simples
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WITH</span> <span class="n">dept_salaries</span> <span class="k">AS</span> <span class="p">(</span>
    <span class="k">SELECT</span> 
        <span class="n">department_id</span><span class="p">,</span>
        <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_salary</span><span class="p">,</span>
        <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">AS</span> <span class="n">num_employees</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">GROUP</span> <span class="k">BY</span> <span class="n">department_id</span>
<span class="p">)</span>
<span class="k">SELECT</span> 
    <span class="n">d</span><span class="p">.</span><span class="n">department_name</span><span class="p">,</span>
    <span class="n">ds</span><span class="p">.</span><span class="n">avg_salary</span><span class="p">,</span>
    <span class="n">ds</span><span class="p">.</span><span class="n">num_employees</span>
<span class="k">FROM</span> <span class="n">dept_salaries</span> <span class="n">ds</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">ds</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">WHERE</span> <span class="n">ds</span><span class="p">.</span><span class="n">avg_salary</span> <span class="o">&gt;</span> <span class="mi">7000</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  4.2 Múltiplas CTEs
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WITH</span> 
<span class="n">high_earners</span> <span class="k">AS</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">salary</span><span class="p">,</span> <span class="n">department_id</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">WHERE</span> <span class="n">salary</span> <span class="o">&gt;</span> <span class="mi">10000</span>
<span class="p">),</span>
<span class="n">dept_info</span> <span class="k">AS</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">department_id</span><span class="p">,</span> <span class="n">department_name</span><span class="p">,</span> <span class="n">location_id</span>
    <span class="k">FROM</span> <span class="n">departments</span>
<span class="p">)</span>
<span class="k">SELECT</span> 
    <span class="n">he</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span>
    <span class="n">he</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span>
    <span class="n">di</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="n">high_earners</span> <span class="n">he</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">dept_info</span> <span class="n">di</span> <span class="k">ON</span> <span class="n">he</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">di</span><span class="p">.</span><span class="n">department_id</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">he</span><span class="p">.</span><span class="n">salary</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  4.3 CTE Recursiva
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WITH</span> <span class="n">employee_hierarchy</span> <span class="p">(</span><span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">manager_id</span><span class="p">,</span> <span class="n">level_num</span><span class="p">)</span> <span class="k">AS</span> <span class="p">(</span>
    <span class="c1">-- Anchor member</span>
    <span class="k">SELECT</span> <span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">manager_id</span><span class="p">,</span> <span class="mi">1</span> <span class="k">AS</span> <span class="n">level_num</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">WHERE</span> <span class="n">manager_id</span> <span class="k">IS</span> <span class="k">NULL</span>

    <span class="k">UNION</span> <span class="k">ALL</span>

    <span class="c1">-- Recursive member</span>
    <span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span><span class="p">,</span> <span class="n">eh</span><span class="p">.</span><span class="n">level_num</span> <span class="o">+</span> <span class="mi">1</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
    <span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">employee_hierarchy</span> <span class="n">eh</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span> <span class="o">=</span> <span class="n">eh</span><span class="p">.</span><span class="n">employee_id</span>
<span class="p">)</span>
<span class="k">SELECT</span> 
    <span class="n">LPAD</span><span class="p">(</span><span class="s1">' '</span><span class="p">,</span> <span class="p">(</span><span class="n">level_num</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span> <span class="o">*</span> <span class="mi">2</span><span class="p">)</span> <span class="o">||</span> <span class="n">first_name</span> <span class="k">AS</span> <span class="k">hierarchy</span><span class="p">,</span>
    <span class="n">level_num</span>
<span class="k">FROM</span> <span class="n">employee_hierarchy</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">level_num</span><span class="p">,</span> <span class="n">first_name</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  5. Técnicas Avançadas
</h2>

<h3>
  
  
  5.1 LATERAL JOIN (Oracle 12c+)
</h3>

<p>O LATERAL permite que a subquery à direita referencie colunas da tabela à esquerda:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">d</span><span class="p">.</span><span class="n">department_name</span><span class="p">,</span>
    <span class="n">top_earners</span><span class="p">.</span><span class="o">*</span>
<span class="k">FROM</span> <span class="n">departments</span> <span class="n">d</span>
<span class="k">CROSS</span> <span class="n">APPLY</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">employee_id</span><span class="p">,</span> <span class="n">first_name</span><span class="p">,</span> <span class="n">salary</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
    <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span>
    <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">salary</span> <span class="k">DESC</span>
    <span class="k">FETCH</span> <span class="k">FIRST</span> <span class="mi">3</span> <span class="k">ROWS</span> <span class="k">ONLY</span>
<span class="p">)</span> <span class="n">top_earners</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  5.2 Subquery com CASE
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">employee_id</span><span class="p">,</span>
    <span class="n">first_name</span><span class="p">,</span>
    <span class="n">salary</span><span class="p">,</span>
    <span class="k">CASE</span> 
        <span class="k">WHEN</span> <span class="n">salary</span> <span class="o">&gt;</span> <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span><span class="p">)</span> 
        <span class="k">THEN</span> <span class="s1">'Acima da Média'</span>
        <span class="k">WHEN</span> <span class="n">salary</span> <span class="o">=</span> <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span><span class="p">)</span> 
        <span class="k">THEN</span> <span class="s1">'Na Média'</span>
        <span class="k">ELSE</span> <span class="s1">'Abaixo da Média'</span>
    <span class="k">END</span> <span class="k">AS</span> <span class="n">classificacao_salarial</span>
<span class="k">FROM</span> <span class="n">employees</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  5.3 Self-Join com Subquery
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> 
    <span class="n">e1</span><span class="p">.</span><span class="n">employee_id</span> <span class="k">AS</span> <span class="n">funcionario_id</span><span class="p">,</span>
    <span class="n">e1</span><span class="p">.</span><span class="n">first_name</span> <span class="k">AS</span> <span class="n">funcionario_nome</span><span class="p">,</span>
    <span class="n">e2</span><span class="p">.</span><span class="n">first_name</span> <span class="k">AS</span> <span class="n">gerente_nome</span><span class="p">,</span>
    <span class="n">e1</span><span class="p">.</span><span class="n">salary</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e1</span>
<span class="k">LEFT</span> <span class="k">JOIN</span> <span class="n">employees</span> <span class="n">e2</span> <span class="k">ON</span> <span class="n">e1</span><span class="p">.</span><span class="n">manager_id</span> <span class="o">=</span> <span class="n">e2</span><span class="p">.</span><span class="n">employee_id</span>
<span class="k">WHERE</span> <span class="n">e1</span><span class="p">.</span><span class="n">salary</span> <span class="o">&gt;</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e3</span>
    <span class="k">WHERE</span> <span class="n">e3</span><span class="p">.</span><span class="n">manager_id</span> <span class="o">=</span> <span class="n">e1</span><span class="p">.</span><span class="n">manager_id</span>
<span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  6. Otimização e Boas Práticas
</h2>

<h3>
  
  
  6.1 Índices
</h3>

<p>Crie índices nas colunas usadas em JOINs e WHERE:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_emp_dept</span> <span class="k">ON</span> <span class="n">employees</span><span class="p">(</span><span class="n">department_id</span><span class="p">);</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_dept_loc</span> <span class="k">ON</span> <span class="n">departments</span><span class="p">(</span><span class="n">location_id</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  6.2 Uso do EXPLAIN PLAN
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">EXPLAIN</span> <span class="n">PLAN</span> <span class="k">FOR</span>
<span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span> <span class="n">d</span><span class="p">.</span><span class="n">department_name</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span><span class="p">;</span>

<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="k">TABLE</span><span class="p">(</span><span class="n">DBMS_XPLAN</span><span class="p">.</span><span class="n">DISPLAY</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  6.3 Evite Subqueries Desnecessárias
</h3>

<p><strong>Ruim:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">employees</span>
<span class="k">WHERE</span> <span class="n">department_id</span> <span class="k">IN</span> <span class="p">(</span><span class="k">SELECT</span> <span class="n">department_id</span> <span class="k">FROM</span> <span class="n">departments</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Melhor:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="o">*</span> <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">departments</span> <span class="n">d</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  6.4 Use EXISTS ao Invés de IN Quando Apropriado
</h3>

<p>Para grandes conjuntos de dados, EXISTS pode ser mais eficiente:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Menos eficiente</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">WHERE</span> <span class="n">department_id</span> <span class="k">IN</span> <span class="p">(</span><span class="k">SELECT</span> <span class="n">department_id</span> <span class="k">FROM</span> <span class="n">departments</span> <span class="k">WHERE</span> <span class="n">location_id</span> <span class="o">=</span> <span class="mi">1700</span><span class="p">);</span>

<span class="c1">-- Mais eficiente</span>
<span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">WHERE</span> <span class="k">EXISTS</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="mi">1</span> <span class="k">FROM</span> <span class="n">departments</span> <span class="n">d</span> 
    <span class="k">WHERE</span> <span class="n">d</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> 
    <span class="k">AND</span> <span class="n">d</span><span class="p">.</span><span class="n">location_id</span> <span class="o">=</span> <span class="mi">1700</span>
<span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  6.5 Evite Subqueries Correlacionadas Quando Possível
</h3>

<p><strong>Menos eficiente (subquery correlacionada):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span>
    <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e2</span> <span class="k">WHERE</span> <span class="n">e2</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_dept_salary</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Mais eficiente (JOIN com subquery):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span> <span class="n">dept_avg</span><span class="p">.</span><span class="n">avg_salary</span>
<span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
<span class="k">INNER</span> <span class="k">JOIN</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">department_id</span><span class="p">,</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_salary</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">GROUP</span> <span class="k">BY</span> <span class="n">department_id</span>
<span class="p">)</span> <span class="n">dept_avg</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">department_id</span> <span class="o">=</span> <span class="n">dept_avg</span><span class="p">.</span><span class="n">department_id</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  7. Exemplos Práticos Completos
</h2>

<h3>
  
  
  7.1 Análise de Vendas
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WITH</span> <span class="n">monthly_sales</span> <span class="k">AS</span> <span class="p">(</span>
    <span class="k">SELECT</span> 
        <span class="n">TRUNC</span><span class="p">(</span><span class="n">order_date</span><span class="p">,</span> <span class="s1">'MM'</span><span class="p">)</span> <span class="k">AS</span> <span class="k">month</span><span class="p">,</span>
        <span class="k">SUM</span><span class="p">(</span><span class="n">total_amount</span><span class="p">)</span> <span class="k">AS</span> <span class="n">total_sales</span><span class="p">,</span>
        <span class="k">COUNT</span><span class="p">(</span><span class="k">DISTINCT</span> <span class="n">customer_id</span><span class="p">)</span> <span class="k">AS</span> <span class="n">unique_customers</span>
    <span class="k">FROM</span> <span class="n">orders</span>
    <span class="k">WHERE</span> <span class="n">order_date</span> <span class="o">&gt;=</span> <span class="n">ADD_MONTHS</span><span class="p">(</span><span class="n">SYSDATE</span><span class="p">,</span> <span class="o">-</span><span class="mi">12</span><span class="p">)</span>
    <span class="k">GROUP</span> <span class="k">BY</span> <span class="n">TRUNC</span><span class="p">(</span><span class="n">order_date</span><span class="p">,</span> <span class="s1">'MM'</span><span class="p">)</span>
<span class="p">),</span>
<span class="n">product_performance</span> <span class="k">AS</span> <span class="p">(</span>
    <span class="k">SELECT</span> 
        <span class="n">p</span><span class="p">.</span><span class="n">product_id</span><span class="p">,</span>
        <span class="n">p</span><span class="p">.</span><span class="n">product_name</span><span class="p">,</span>
        <span class="k">SUM</span><span class="p">(</span><span class="n">oi</span><span class="p">.</span><span class="n">quantity</span> <span class="o">*</span> <span class="n">oi</span><span class="p">.</span><span class="n">unit_price</span><span class="p">)</span> <span class="k">AS</span> <span class="n">revenue</span>
    <span class="k">FROM</span> <span class="n">products</span> <span class="n">p</span>
    <span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">order_items</span> <span class="n">oi</span> <span class="k">ON</span> <span class="n">p</span><span class="p">.</span><span class="n">product_id</span> <span class="o">=</span> <span class="n">oi</span><span class="p">.</span><span class="n">product_id</span>
    <span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">orders</span> <span class="n">o</span> <span class="k">ON</span> <span class="n">oi</span><span class="p">.</span><span class="n">order_id</span> <span class="o">=</span> <span class="n">o</span><span class="p">.</span><span class="n">order_id</span>
    <span class="k">WHERE</span> <span class="n">o</span><span class="p">.</span><span class="n">order_date</span> <span class="o">&gt;=</span> <span class="n">ADD_MONTHS</span><span class="p">(</span><span class="n">SYSDATE</span><span class="p">,</span> <span class="o">-</span><span class="mi">12</span><span class="p">)</span>
    <span class="k">GROUP</span> <span class="k">BY</span> <span class="n">p</span><span class="p">.</span><span class="n">product_id</span><span class="p">,</span> <span class="n">p</span><span class="p">.</span><span class="n">product_name</span>
<span class="p">)</span>
<span class="k">SELECT</span> 
    <span class="n">ms</span><span class="p">.</span><span class="k">month</span><span class="p">,</span>
    <span class="n">ms</span><span class="p">.</span><span class="n">total_sales</span><span class="p">,</span>
    <span class="n">ms</span><span class="p">.</span><span class="n">unique_customers</span><span class="p">,</span>
    <span class="n">pp</span><span class="p">.</span><span class="n">product_name</span> <span class="k">AS</span> <span class="n">top_product</span><span class="p">,</span>
    <span class="n">pp</span><span class="p">.</span><span class="n">revenue</span> <span class="k">AS</span> <span class="n">top_product_revenue</span>
<span class="k">FROM</span> <span class="n">monthly_sales</span> <span class="n">ms</span>
<span class="k">CROSS</span> <span class="k">JOIN</span> <span class="k">LATERAL</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">product_name</span><span class="p">,</span> <span class="n">revenue</span>
    <span class="k">FROM</span> <span class="n">product_performance</span>
    <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">revenue</span> <span class="k">DESC</span>
    <span class="k">FETCH</span> <span class="k">FIRST</span> <span class="mi">1</span> <span class="k">ROW</span> <span class="k">ONLY</span>
<span class="p">)</span> <span class="n">pp</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">ms</span><span class="p">.</span><span class="k">month</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  7.2 Relatório de Hierarquia com Métricas
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WITH</span> <span class="k">RECURSIVE</span> <span class="n">emp_hierarchy</span> <span class="k">AS</span> <span class="p">(</span>
    <span class="k">SELECT</span> 
        <span class="n">employee_id</span><span class="p">,</span>
        <span class="n">first_name</span><span class="p">,</span>
        <span class="n">manager_id</span><span class="p">,</span>
        <span class="n">salary</span><span class="p">,</span>
        <span class="mi">1</span> <span class="k">AS</span> <span class="n">level_depth</span><span class="p">,</span>
        <span class="k">CAST</span><span class="p">(</span><span class="n">first_name</span> <span class="k">AS</span> <span class="n">VARCHAR2</span><span class="p">(</span><span class="mi">1000</span><span class="p">))</span> <span class="k">AS</span> <span class="n">path</span>
    <span class="k">FROM</span> <span class="n">employees</span>
    <span class="k">WHERE</span> <span class="n">manager_id</span> <span class="k">IS</span> <span class="k">NULL</span>

    <span class="k">UNION</span> <span class="k">ALL</span>

    <span class="k">SELECT</span> 
        <span class="n">e</span><span class="p">.</span><span class="n">employee_id</span><span class="p">,</span>
        <span class="n">e</span><span class="p">.</span><span class="n">first_name</span><span class="p">,</span>
        <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span><span class="p">,</span>
        <span class="n">e</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span>
        <span class="n">eh</span><span class="p">.</span><span class="n">level_depth</span> <span class="o">+</span> <span class="mi">1</span><span class="p">,</span>
        <span class="n">eh</span><span class="p">.</span><span class="n">path</span> <span class="o">||</span> <span class="s1">' &gt; '</span> <span class="o">||</span> <span class="n">e</span><span class="p">.</span><span class="n">first_name</span>
    <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span>
    <span class="k">INNER</span> <span class="k">JOIN</span> <span class="n">emp_hierarchy</span> <span class="n">eh</span> <span class="k">ON</span> <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span> <span class="o">=</span> <span class="n">eh</span><span class="p">.</span><span class="n">employee_id</span>
<span class="p">)</span>
<span class="k">SELECT</span> 
    <span class="n">eh</span><span class="p">.</span><span class="n">path</span><span class="p">,</span>
    <span class="n">eh</span><span class="p">.</span><span class="n">salary</span><span class="p">,</span>
    <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_company_salary</span><span class="p">,</span>
    <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span> <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span> <span class="o">=</span> <span class="n">eh</span><span class="p">.</span><span class="n">manager_id</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_peer_salary</span><span class="p">,</span>
    <span class="k">CASE</span> 
        <span class="k">WHEN</span> <span class="n">eh</span><span class="p">.</span><span class="n">salary</span> <span class="o">&gt;</span> <span class="p">(</span><span class="k">SELECT</span> <span class="k">AVG</span><span class="p">(</span><span class="n">salary</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">employees</span> <span class="n">e</span> <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">manager_id</span> <span class="o">=</span> <span class="n">eh</span><span class="p">.</span><span class="n">manager_id</span><span class="p">)</span>
        <span class="k">THEN</span> <span class="s1">'Acima dos Pares'</span>
        <span class="k">ELSE</span> <span class="s1">'Abaixo dos Pares'</span>
    <span class="k">END</span> <span class="k">AS</span> <span class="n">comparacao</span>
<span class="k">FROM</span> <span class="n">emp_hierarchy</span> <span class="n">eh</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">eh</span><span class="p">.</span><span class="n">level_depth</span><span class="p">,</span> <span class="n">eh</span><span class="p">.</span><span class="n">first_name</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  8. Resumo e Conclusão
</h2>

<p>Este guia cobriu extensivamente:</p>

<ul>
<li>
<strong>INNER JOINs</strong>: Relacionamento entre tabelas, sintaxes antigas vs modernas, múltiplos JOINs</li>
<li>
<strong>Subqueries</strong>: Escalares, múltiplas linhas, correlacionadas, inline views</li>
<li>
<strong>Operadores</strong>: EXISTS, NOT EXISTS, IN, ANY, ALL</li>
<li>
<strong>CTEs</strong>: Expressões de tabela comuns, incluindo recursivas</li>
<li>
<strong>Técnicas Avançadas</strong>: LATERAL JOIN, self-joins, combinações complexas</li>
<li>
<strong>Otimização</strong>: Índices, EXPLAIN PLAN, boas práticas de performance</li>
</ul>

<p>O domínio dessas técnicas permite escrever consultas SQL complexas, eficientes e manuteníveis no Oracle Database. Pratique regularmente e sempre considere a performance ao escolher entre diferentes abordagens.</p>

