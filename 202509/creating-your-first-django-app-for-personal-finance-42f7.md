---
Title: Creating Your First Django App for Personal Finance
Description: 
Author: Joe
Date: 2025-09-11T21:57:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>Now that our Django project is set up, it's time to add an app—the heart of your financial tracker!</p>




<h2>
  
  
  1. <strong>What Is a Django App?</strong>
</h2>

<p>A "Django app" is a module that handles a specific feature (like tracking transactions). Projects can have many apps.</p>




<h2>
  
  
  2. <strong>Create the Tracker App</strong>
</h2>

<p>Run this command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python manage.py startapp tracker
</code></pre>

</div>



<p>Your folder now includes <code>tracker/</code> with:</p>

<ul>
<li>
<code>models.py</code>: Data models</li>
<li>
<code>views.py</code>: Request handlers</li>
<li>
<code>admin.py</code>: Admin settings</li>
</ul>




<h2>
  
  
  3. <strong>Register the App</strong>
</h2>

<p>Open <code>config/settings.py</code> and add <code>'tracker',</code> to <code>INSTALLED_APPS</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">INSTALLED_APPS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c1"># ...default apps
</span>    <span class="sh">'</span><span class="s">tracker</span><span class="sh">'</span><span class="p">,</span>
<span class="p">]</span>
</code></pre>

</div>






<h2>
  
  
  4. <strong>Why Register?</strong>
</h2>

<p>This lets Django know to include your app's models, views, and admin features.</p>




<p>Next up: designing the models for your tracker!</p>




