---
Title: Django Tip: Why Your Static Files Disappear When DEBUG = False
Description: 
Author: The Builder
Date: 2025-09-16T21:31:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’ve ever been happily styling your Django project and then suddenly lost all your CSS and JavaScript after setting <code>DEBUG = False,</code> you’re not alone. This “disappearing static files” issue is one of the first surprises many Django developers run into when moving from development to production.</p>

<p>Let’s unpack why this happens, and how to fix it.</p>

<h2>
  
  
  Why it works with <code>DEBUG = True</code>
</h2>

<p>When you’re building locally with <code>DEBUG = True,</code> Django’s runserver is designed to make your life easier. It automatically serves your static files using the django.contrib.staticfiles app. You don’t need to configure anything special — your CSS, JavaScript, and images just show up.</p>

<p>This is fine for development. But Django’s philosophy is that in production you should use a dedicated web server or middleware for serving static files. Why? Because web servers like Nginx or Apache (or CDNs) are optimized for serving static assets efficiently, while Django is not.</p>

<h2>
  
  
  What happens when DEBUG = False
</h2>

<p>Once you flip that setting, Django stops serving static files altogether. If you haven’t set up an alternative way to serve them, you’ll instantly notice that your site is broken — no styles, no images, no scripts.</p>

<p>This isn’t a bug; it’s by design. It’s Django’s way of reminding you:</p>

<blockquote>
<p>“In production, static file handling is your responsibility.</p>
</blockquote>

<p>How to fix it (the right way)</p>

<p>Here’s the proper workflow once you’re ready to run with <code>DEBUG = False:</code></p>

<ul>
<li>Tell Django where to collect your static files
In your settings.py, add:</li>
</ul>

<p><code><br>
STATIC_ROOT = BASE_DIR / "staticfiles"<br>
</code></p>

<ul>
<li>Run the collectstatic command
This command gathers all static files from your apps and places them into the <code>STATIC_ROOT</code>`folder.</li>
</ul>

<p><code></code><br>
python manage.py collectstatic<br>
<code></code></p>

<ul>
<li>Serve those files with a proper solution</li>
</ul>

<p>Web server option: Configure Nginx or Apache to serve the contents of your <code></code>STATIC_ROOT.<code></code></p>

<p>Middleware option: Use Whitenoise, which allows Django to serve static files in production without an external server. This is a common approach for simpler deployments (like on Heroku).</p>

<h3>
  
  
  Key takeaway
</h3>

<p>Django makes static files easy to work with during development, but it hands over the responsibility once you go to production. That’s why your static files vanish when you set <code></code>DEBUG = False<code></code>.</p>

<p>Understanding this behavior early on saves hours of confusion and makes your deployment process much smoother.</p>

<p>So next time you see your CSS disappear, don’t panic, just remember: Django doesn’t serve static files in production; you do.</p>

