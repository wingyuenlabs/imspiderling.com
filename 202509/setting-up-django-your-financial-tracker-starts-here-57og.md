---
Title: Setting Up Django: Your Financial Tracker Starts Here
Description: 
Author: Joe
Date: 2025-09-11T21:57:31.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Setting Up Django: Your Financial Tracker Starts Here
</h2>

<p>Welcome to Part 2 of my Django financial tracker series! In this post, we'll go from zero to a working Django project—you'll be ready to code in minutes.</p>




<h2>
  
  
  1. <strong>Create a Python Virtual Environment</strong>
</h2>

<p>A virtual environment keeps your dependencies isolated.<br><br>
Run these commands in your project folder:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python <span class="nt">-m</span> venv venv
<span class="nb">source </span>venv/bin/activate  <span class="c"># On Windows: venv\Scripts\activate</span>
</code></pre>

</div>






<h2>
  
  
  2. <strong>Install Django</strong>
</h2>

<p>Install Django inside your virtual environment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>django
</code></pre>

</div>






<h2>
  
  
  3. <strong>Start Your Django Project</strong>
</h2>

<p>In your folder, run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>django-admin startproject config <span class="nb">.</span>
</code></pre>

</div>



<p>Your folder should now look like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>config/
    __init__.py
    settings.py
    urls.py
    asgi.py
    wsgi.py
manage.py
</code></pre>

</div>






<h2>
  
  
  4. <strong>What Are These Files?</strong>
</h2>

<ul>
<li>
<code>manage.py</code>: Django's command-line utility</li>
<li>
<code>config/</code>: Project settings, URLs, and core files</li>
</ul>




<p>That’s it! You’ve got a Django project ready to go.<br><br>
Next up: creating the core tracker app.</p>




