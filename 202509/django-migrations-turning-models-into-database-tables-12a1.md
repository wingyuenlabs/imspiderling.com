---
Title: Django Migrations: Turning Models Into Database Tables
Description: 
Author: Joe
Date: 2025-09-11T21:58:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>Django migrations take your Python models and create real database tables.</p>




<h2>
  
  
  1. <strong>Create Migrations</strong>
</h2>

<p>Run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python manage.py makemigrations tracker
</code></pre>

</div>






<h2>
  
  
  2. <strong>Apply Migrations</strong>
</h2>

<p>Run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python manage.py migrate
</code></pre>

</div>






<h2>
  
  
  3. <strong>What Just Happened?</strong>
</h2>

<ul>
<li>The <code>Category</code> and <code>Transaction</code> tables are now in your database!</li>
<li>Django tracks changes for future updates.</li>
</ul>




<p>Next: manage your data easily with Django admin.</p>




