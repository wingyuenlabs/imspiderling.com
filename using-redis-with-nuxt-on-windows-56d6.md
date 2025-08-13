---
Title: Using Redis with Nuxt on Windows
Description: This is more a self documentation post as I ran into some issues trying to get Redis to work on Windows using Docker
Recently I've been messing around with Nuxt. As a fan of Vue.js and its ecosystem, ...
Author: Ibz786
Date: 2025-08-13T00:23:46.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>This is more a self documentation post as I ran into some issues trying to get Redis to work on Windows using Docker</p>
</blockquote>

<p>Recently I've been messing around with <code>Nuxt</code>. As a fan of <code>Vue.js</code> and its ecosystem, I've found <code>Nuxt</code> to be an incredibly intuitive framework. There have been some "pain points" but thankfully overcome quite easily</p>

<p>Currently, when messing around with a few concepts, I came to the conclusion that I need to store some data on the server, that could essentially be cached and accessed quickly, without having to constantly hit the DB and do various joins etc.</p>

<p>Nuxt "out of the box" offers standard memory storage on the server side, but with the option to use other means or storage drivers. One of which, being <code>redis</code></p>

<p>Since I'm using Windows, there's no official driver or way to install and use Redis. However, I came across one solution, <code>Docker</code>. Admittingly, I've heard of Docker, but have never in fact used it... up until this point. So I went ahead on the Docker website, got the Windows installer and installed it</p>

<p>You can install Docker for Windows here: <a href="https://docs.docker.com/desktop/setup/install/windows-install/" rel="noopener noreferrer">https://docs.docker.com/desktop/setup/install/windows-install/</a></p>

<p>It's also important to note, that you should also have WSL (Windows Subsystem for Linux) installed. If you don't have it installed you can run the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight postscript"><code><span class="nf">wsl</span> <span class="nf">--install</span>
</code></pre>

</div>



<p>Otherwise if you do, it would be best to update it just in case, using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight postscript"><code><span class="nf">wsl</span> <span class="nf">--update</span>
</code></pre>

</div>



<p>Once docker is installed, install redis by running the following command. This should get the latest version:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight postscript"><code><span class="nf">docker</span> <span class="nf">pull</span> <span class="nf">redis</span>
</code></pre>

</div>



<p>Afterwards your ready to start up redis in your docker container. Run the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight postscript"><code><span class="nf">docker</span> <span class="nb">run</span> <span class="nf">-p</span> <span class="nf">6379:6379</span> <span class="nf">--name</span> <span class="nf">redis1</span> <span class="nf">-d</span> <span class="nf">redis</span>
</code></pre>

</div>



<p>Its important to ensure that <code>6379:6379</code> is specificed as this part of the command exposes the docker container port to your local machine (something that I had missed earlier and was going in circles trying to figure out what went wrong)</p>

<p>Also <code>redis1</code> in this case is the container or image name, and you can name yours anything you wish</p>

<p>This should kick start your Docker Redis container. You can check your containers using the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight postscript"><code><span class="nf">docker</span> <span class="nf">ps</span>
</code></pre>

</div>



<p>After which you can access your redis container via<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight postscript"><code><span class="nf">docker</span> <span class="nb">exec</span> <span class="nf">-it</span> <span class="nf">redis1</span> <span class="nf">redis-cli</span>
</code></pre>

</div>



<p>It should show up as 127.0.0.1:6379. You can test if its working by running<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight postscript"><code><span class="nf">ping</span>
</code></pre>

</div>



<p>And you'll get <code>PONG</code> back</p>

<p>Now to integrate it with Nuxt, simply add redis to your Nuxt app, via<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">npm</span> <span class="nx">i</span> <span class="nx">ioredis</span>
</code></pre>

</div>



<p>And following the Nuxt documentation, I added the config to my Nuxt config<br>
<a href="https://nuxt.com/docs/4.x/guide/directory-structure/server#server-storage" rel="noopener noreferrer">https://nuxt.com/docs/4.x/guide/directory-structure/server#server-storage</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineNuxtConfig</span><span class="p">({</span>
  <span class="na">nitro</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">storage</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">redis</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">driver</span><span class="p">:</span> <span class="dl">'</span><span class="s1">redis</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">port</span><span class="p">:</span> <span class="mi">6379</span><span class="p">,</span>
        <span class="na">host</span><span class="p">:</span> <span class="dl">"</span><span class="s2">127.0.0.1</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">})</span>
</code></pre>

</div>



<p>In your API handler or other logic file, you can run the following<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">export</span> <span class="k">default</span> <span class="nf">defineEventHandler</span><span class="p">(</span><span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// List all keys with</span>
  <span class="kd">const</span> <span class="nx">keys</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">useStorage</span><span class="p">(</span><span class="dl">'</span><span class="s1">redis</span><span class="dl">'</span><span class="p">).</span><span class="nf">getKeys</span><span class="p">();</span>

  <span class="c1">// Set a key with</span>
  <span class="k">await</span> <span class="nf">useStorage</span><span class="p">(</span><span class="dl">'</span><span class="s1">redis</span><span class="dl">'</span><span class="p">).</span><span class="nf">setItem</span><span class="p">(</span><span class="dl">'</span><span class="s1">foo</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">bar</span><span class="dl">'</span><span class="p">);</span>

  <span class="c1">// Get a key with</span>
  <span class="k">await</span> <span class="nf">useStorage</span><span class="p">(</span><span class="dl">'</span><span class="s1">redis</span><span class="dl">'</span><span class="p">).</span><span class="nf">getItem</span><span class="p">(</span><span class="dl">'</span><span class="s1">foo</span><span class="dl">'</span><span class="p">);</span>

  <span class="c1">// Remove a key with</span>
  <span class="k">await</span> <span class="nf">useStorage</span><span class="p">(</span><span class="dl">'</span><span class="s1">redis</span><span class="dl">'</span><span class="p">).</span><span class="nf">removeItem</span><span class="p">(</span><span class="dl">'</span><span class="s1">foo</span><span class="dl">'</span><span class="p">);</span>

  <span class="k">return</span> <span class="p">{}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>And viola! Hope that helps anyone else facing similar issues</p>

