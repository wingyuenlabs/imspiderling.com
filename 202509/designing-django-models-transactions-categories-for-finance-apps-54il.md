---
Title: Designing Django Models: Transactions & Categories for Finance Apps
Description: 
Author: Joe
Date: 2025-09-11T21:58:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>To track finances, we need two main data types—categories (like "Food") and transactions (money in or out).</p>




<h2>
  
  
  1. <strong>Plan Your Models</strong>
</h2>

<ul>
<li>
<strong>Category</strong>: e.g. Food, Rent, Salary</li>
<li>
<strong>Transaction</strong>: amount, date, category, type, notes</li>
</ul>




<h2>
  
  
  2. <strong>Code the Models</strong>
</h2>

<p>Edit <code>tracker/models.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">django.db</span> <span class="kn">import</span> <span class="n">models</span>

<span class="k">class</span> <span class="nc">Category</span><span class="p">(</span><span class="n">models</span><span class="p">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">name</span> <span class="o">=</span> <span class="n">models</span><span class="p">.</span><span class="nc">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">50</span><span class="p">,</span> <span class="n">unique</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">name</span>

<span class="k">class</span> <span class="nc">Transaction</span><span class="p">(</span><span class="n">models</span><span class="p">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">INCOME</span> <span class="o">=</span> <span class="sh">'</span><span class="s">IN</span><span class="sh">'</span>
    <span class="n">EXPENSE</span> <span class="o">=</span> <span class="sh">'</span><span class="s">EX</span><span class="sh">'</span>
    <span class="n">TRANSACTION_TYPES</span> <span class="o">=</span> <span class="p">[</span>
        <span class="p">(</span><span class="n">INCOME</span><span class="p">,</span> <span class="sh">'</span><span class="s">Income</span><span class="sh">'</span><span class="p">),</span>
        <span class="p">(</span><span class="n">EXPENSE</span><span class="p">,</span> <span class="sh">'</span><span class="s">Expense</span><span class="sh">'</span><span class="p">),</span>
    <span class="p">]</span>
    <span class="n">category</span> <span class="o">=</span> <span class="n">models</span><span class="p">.</span><span class="nc">ForeignKey</span><span class="p">(</span><span class="n">Category</span><span class="p">,</span> <span class="n">on_delete</span><span class="o">=</span><span class="n">models</span><span class="p">.</span><span class="n">CASCADE</span><span class="p">)</span>
    <span class="n">amount</span> <span class="o">=</span> <span class="n">models</span><span class="p">.</span><span class="nc">DecimalField</span><span class="p">(</span><span class="n">max_digits</span><span class="o">=</span><span class="mi">10</span><span class="p">,</span> <span class="n">decimal_places</span><span class="o">=</span><span class="mi">2</span><span class="p">)</span>
    <span class="n">date</span> <span class="o">=</span> <span class="n">models</span><span class="p">.</span><span class="nc">DateField</span><span class="p">()</span>
    <span class="n">transaction_type</span> <span class="o">=</span> <span class="n">models</span><span class="p">.</span><span class="nc">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">2</span><span class="p">,</span> <span class="n">choices</span><span class="o">=</span><span class="n">TRANSACTION_TYPES</span><span class="p">)</span>
    <span class="n">notes</span> <span class="o">=</span> <span class="n">models</span><span class="p">.</span><span class="nc">TextField</span><span class="p">(</span><span class="n">blank</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="nf">get_transaction_type_display</span><span class="p">()</span><span class="si">}</span><span class="s"> - </span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">category</span><span class="p">.</span><span class="n">name</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">amount</span><span class="si">}</span><span class="s"> on </span><span class="si">{</span><span class="n">self</span><span class="p">.</span><span class="n">date</span><span class="si">}</span><span class="sh">"</span>
</code></pre>

</div>






<h2>
  
  
  3. <strong>Why This Structure?</strong>
</h2>

<ul>
<li>
<strong>ForeignKey</strong> connects transactions to categories</li>
<li>
<strong>Choices</strong> enforce transaction type</li>
</ul>




<p>Let’s turn these models into database tables next!</p>




