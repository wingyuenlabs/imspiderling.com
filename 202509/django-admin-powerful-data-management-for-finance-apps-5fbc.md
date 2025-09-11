---
Title: Django Admin: Powerful Data Management for Finance Apps
Description: 
Author: Joe
Date: 2025-09-11T21:59:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Django comes with a built-in admin interface—no coding required!</p>




<h2>
  
  
  1. <strong>Create a Superuser</strong>
</h2>

<p>Run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python manage.py createsuperuser
</code></pre>

</div>



<p>Follow the prompts.</p>




<h2>
  
  
  2. <strong>Register Models for Admin</strong>
</h2>

<p>Edit <code>tracker/admin.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>
<span class="kn">from</span> <span class="n">.models</span> <span class="kn">import</span> <span class="n">Category</span><span class="p">,</span> <span class="n">Transaction</span>

<span class="nd">@admin.register</span><span class="p">(</span><span class="n">Category</span><span class="p">)</span>
<span class="k">class</span> <span class="nc">CategoryAdmin</span><span class="p">(</span><span class="n">admin</span><span class="p">.</span><span class="n">ModelAdmin</span><span class="p">):</span>
    <span class="n">list_display</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">name</span><span class="sh">'</span><span class="p">]</span>

<span class="nd">@admin.register</span><span class="p">(</span><span class="n">Transaction</span><span class="p">)</span>
<span class="k">class</span> <span class="nc">TransactionAdmin</span><span class="p">(</span><span class="n">admin</span><span class="p">.</span><span class="n">ModelAdmin</span><span class="p">):</span>
    <span class="n">list_display</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">date</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">transaction_type</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">category</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">amount</span><span class="sh">'</span><span class="p">]</span>
    <span class="n">list_filter</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">transaction_type</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">category</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">date</span><span class="sh">'</span><span class="p">]</span>
    <span class="n">search_fields</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">notes</span><span class="sh">'</span><span class="p">]</span>
</code></pre>

</div>






<h2>
  
  
  3. <strong>Run the Server &amp; Use Admin</strong>
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python manage.py runserver
</code></pre>

</div>



<p>Go to <a href="http://127.0.0.1:8000/admin/" rel="noopener noreferrer">http://127.0.0.1:8000/admin/</a></p>




<p>Now you can add, edit, delete, and search transactions and categories!</p>




<p>Next: building custom views, forms, and templates.</p>




