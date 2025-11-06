---
Title: Secure Apache with SSL in Docker
Description: 
Author: Tshenolo Mos
Date: 2025-11-06T21:49:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>This guide will walk you through setting up an Apache server with PHP support within a Docker container, secured with an SSL certificate for local development purposes. We’ll ensure that the server can be accessed securely via <a href="https://localhost" rel="noopener noreferrer">https://localhost</a> using a self-signed SSL certificate. If you’re eager to jump straight into the code, you can locate the complete project on GitHub. Feel free to access it at <a href="https://github.com/tshenolo/docker-Secure-Apache" rel="noopener noreferrer">https://github.com/tshenolo/docker-Secure-Apache</a>.</p>

<h2>
  
  
  Table of Contents
</h2>

<ol>
<li>Introduction</li>
<li>Prerequisites</li>
<li>Step 1: Preparing SSL Certificates</li>
<li>Step 2: Creating the Dockerfile</li>
<li>Step 3: Configuring Apache for SSL</li>
<li>Step 4: Building and Running the Docker Container</li>
<li>Step 5: Accessing Your Secure Apache Server</li>
<li>Conclusion</li>
</ol>

<h2>
  
  
  Introduction
</h2>

<p>Deploying your web applications in a development environment that closely mirrors your production setup is crucial. This guide focuses on configuring an Apache server with SSL in Docker, providing a secure, isolated environment for development.</p>

<p>Check out my <a href="https://www.youtube.com/@tshenolo" rel="noopener noreferrer">Youtube Channel</a> where I post all kinds of content accompanying my posts, including this video showing everything in this post.</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/8A7bO7MDG9Y">
</iframe>
</p>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>Docker installed on your machine.</li>
<li>Basic knowledge of Docker, Apache, and SSL certificates.</li>
<li>OpenSSL for generating a self-signed certificate (if not already installed).</li>
</ul>

<h2>
  
  
  Step 1: Preparing SSL Certificates
</h2>

<p>Before setting up the Docker container, you’ll need to generate a self-signed SSL certificate for localhost. Open a terminal and run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>openssl req <span class="nt">-x509</span> <span class="nt">-nodes</span> <span class="nt">-days</span> 365 <span class="nt">-newkey</span> rsa:2048 <span class="nt">-keyout</span> localhost.key <span class="nt">-out</span> localhost.crt <span class="nt">-subj</span> <span class="s2">"/C=US/ST=YourState/L=YourCity/O=YourOrganization/CN=localhost"</span>
</code></pre>

</div>



<p>This command generates a new SSL certificate (localhost.crt) and a private key (localhost.key), valid for 365 days.</p>

<h2>
  
  
  Step 2: Creating the Dockerfile
</h2>

<p>Create a Dockerfile in your project directory with the following contents:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Use an official PHP image with Apache</span>
<span class="k">FROM</span><span class="s"> php:7.4-apache</span>

<span class="c"># Install Vim (optional)</span>
<span class="k">RUN </span>apt-get update <span class="o">&amp;&amp;</span> <span class="se">\
</span>    apt-get <span class="nb">install</span> <span class="nt">-y</span> vim <span class="o">&amp;&amp;</span> <span class="se">\
</span>    <span class="nb">rm</span> <span class="nt">-rf</span> /var/lib/apt/lists/<span class="k">*</span>

<span class="c"># Copy SSL certificate and key</span>
<span class="k">COPY</span><span class="s"> localhost.crt /etc/ssl/certs/localhost.crt</span>
<span class="k">COPY</span><span class="s"> localhost.key /etc/ssl/private/localhost.key</span>

<span class="c"># Copy the custom Apache virtual host config</span>
<span class="k">COPY</span><span class="s"> ./my-httpd-vhosts.conf /etc/apache2/sites-available/my-ssl.conf</span>

<span class="c"># Enable SSL module, configure Apache for PHP support, and enable our SSL site configuration</span>
<span class="k">RUN </span>a2enmod ssl <span class="o">&amp;&amp;</span> <span class="se">\
</span>    a2enmod rewrite <span class="o">&amp;&amp;</span> <span class="se">\
</span>    a2dissite 000-default default-ssl <span class="o">&amp;&amp;</span> <span class="se">\
</span>    a2ensite my-ssl

<span class="c"># Copy your PHP application into the default Apache document root</span>
<span class="k">COPY</span><span class="s"> ./public-html/ /var/www/html/</span>
</code></pre>

</div>



<h2>
  
  
  Step 3: Configuring Apache for SSL
</h2>

<p>Create a file named my-httpd-vhosts.conf in your project directory with the following Apache virtual host configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>&lt;VirtualHost <span class="k">*</span>:443&gt;
    DocumentRoot <span class="s2">"/var/www/html"</span>
    ServerName localhost

    SSLEngine on
    SSLCertificateFile <span class="s2">"/etc/ssl/certs/localhost.crt"</span>
    SSLCertificateKeyFile <span class="s2">"/etc/ssl/private/localhost.key"</span>

    <span class="c"># Other directives here</span>
&lt;/VirtualHost&gt;
</code></pre>

</div>



<h2>
  
  
  Step 4: Building and Running the Docker Container
</h2>

<p>With the Dockerfile and SSL configuration in place, build your Docker image:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker build <span class="nt">-t</span> apache-php-ssl <span class="nb">.</span>
</code></pre>

</div>



<p>Then, run your container:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker run <span class="nt">-d</span> <span class="nt">-p</span> 443:443 apache-php-ssl
</code></pre>

</div>



<p>This command starts a container from your image, mapping port 443 inside the container to port 443 on your host machine.</p>

<h2>
  
  
  Step 5: Accessing Your Secure Apache Server
</h2>

<p>Open a web browser and navigate to <a href="https://localhost" rel="noopener noreferrer">https://localhost</a>. You should see your PHP application running securely under HTTPS. Since the certificate is self-signed, your browser may warn you about the site’s security certificate. You can proceed by allowing an exception for this certificate.</p>

<h2>
  
  
  Conclusion
</h2>

<p>You’ve successfully set up an Apache server with SSL inside a Docker container, making your local development environment secure and more aligned with production settings. This setup ensures that your development practices include necessary security measures right from the start.</p>

<p>Thank you for reading this blog post. If you found the post helpful or interesting, here are a few ways you can show your support:</p>

<p>🐦 Follow me on <a href="https://x.com/tshenolo" rel="noopener noreferrer">X</a><br>
📺 Subscribe to my <a href="https://www.youtube.com/@tshenolo?sub_confirmation=1" rel="noopener noreferrer">Youtube channel</a><br>
Your support and engagement means a lot to me as an open-source developer.</p>

