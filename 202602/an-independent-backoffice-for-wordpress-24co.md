---
Title: An Independent backoffice for WordPress?
Description: 
Author: giuliopanda
Date: 2026-02-15T21:51:15.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  It All Started with Metadata
</h2>

<p>Milk Admin natively relies on two separate databases — one for internal configuration, the other for managing external data. This architecture made it a natural candidate for integration with external systems, such as a WordPress database.</p>

<p>Once connected, I was able to start managing a WordPress site's tables directly from my panel. From there, an immediate need emerged: handling metadata relationships natively. I looked at how the most well-known and mature systems similar to mine handled this, and I found that there usually isn't a native system for it. So I wrote hasMeta, a method that handles native metadata relationships.</p>

<h2>
  
  
  How it looks in a model definition
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$rule</span><span class="o">-&gt;</span><span class="nf">table</span><span class="p">(</span><span class="s1">'#__users'</span><span class="p">)</span>
    <span class="o">-&gt;</span><span class="nf">id</span><span class="p">()</span>
        <span class="o">-&gt;</span><span class="nf">hasMeta</span><span class="p">(</span><span class="s1">'nickname'</span><span class="p">,</span> <span class="nc">UserMetaModel</span><span class="o">::</span><span class="n">class</span><span class="p">,</span> <span class="s1">'user_id'</span><span class="p">)</span>
        <span class="o">-&gt;</span><span class="nf">hasMeta</span><span class="p">(</span><span class="s1">'city'</span><span class="p">,</span> <span class="nc">UserMetaModel</span><span class="o">::</span><span class="n">class</span><span class="p">,</span> <span class="s1">'user_id'</span><span class="p">)</span>
    <span class="o">-&gt;</span><span class="nf">string</span><span class="p">(</span><span class="s1">'name'</span><span class="p">,</span> <span class="mi">100</span><span class="p">)</span><span class="o">-&gt;</span><span class="nf">required</span><span class="p">();</span>
</code></pre>

</div>



<p>Then there are a number of other small conventions I inherited from WordPress: the dynamic table prefix (#__) and a functions.php file for more advanced customizations.</p>

<h2>
  
  
  A Small Personal Win
</h2>

<p>Every now and then I try to promote my work — I'm terrible at it, but I try. When I showed the metadata functionality to a client of mine, his reaction was enthusiastic. He asked me for a demo… that had nothing to do with metadata. He wanted to verify whether my admin panel could handle WordPress authentication.</p>

<p>A different request than expected, but an exciting one, so I got to work.</p>

<h2>
  
  
  The WordPress Plugin
</h2>

<p>I developed a plugin that hooks into WordPress's authenticate filter, with priority 30, to intervene after the standard checks:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nf">add_filter</span><span class="p">(</span><span class="s1">'authenticate'</span><span class="p">,</span> <span class="p">[</span><span class="nv">$this</span><span class="p">,</span> <span class="s1">'authenticate'</span><span class="p">],</span> <span class="mi">30</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>
</code></pre>

</div>



<p>The method receives three parameters: $user (the result of previous checks), $username, and $password. If conditions require it, the plugin makes a POST call to my admin panel's endpoint via wp_remote_post:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$args</span> <span class="o">=</span> <span class="p">[</span>
    <span class="s1">'headers'</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="s1">'Content-Type'</span> <span class="o">=&gt;</span> <span class="s1">'application/json'</span><span class="p">],</span>
    <span class="s1">'timeout'</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="n">int</span><span class="p">)</span><span class="nv">$options</span><span class="p">[</span><span class="s1">'request_timeout'</span><span class="p">],</span>
    <span class="s1">'body'</span>    <span class="o">=&gt;</span> <span class="nf">wp_json_encode</span><span class="p">([</span>
        <span class="s1">'username'</span> <span class="o">=&gt;</span> <span class="nv">$username</span><span class="p">,</span>
        <span class="s1">'password'</span> <span class="o">=&gt;</span> <span class="nv">$password</span><span class="p">,</span>
    <span class="p">]),</span>
<span class="p">];</span>

<span class="nv">$response</span> <span class="o">=</span> <span class="nf">wp_remote_post</span><span class="p">(</span><span class="nv">$endpoint</span><span class="p">,</span> <span class="nv">$args</span><span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  The Authentication Endpoint
</h2>

<p>On the Milk Admin side, I created a dedicated module that exposes a public endpoint for credential verification:</p>

<p><code>#[ApiEndpoint('public-auth/check-credentials', 'POST')]</code></p>

<p>Reachable with a simple call:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="no">POST</span> <span class="o">/</span><span class="n">public_html</span><span class="o">/</span><span class="n">api</span><span class="mf">.</span><span class="n">php</span><span class="o">?</span><span class="n">page</span><span class="o">=</span><span class="k">public</span><span class="o">-</span><span class="n">auth</span><span class="o">/</span><span class="n">check</span><span class="o">-</span><span class="n">credentials</span>
<span class="nc">Content</span><span class="o">-</span><span class="nc">Type</span><span class="o">:</span> <span class="n">application</span><span class="o">/</span><span class="n">json</span>

<span class="p">{</span>
  <span class="s2">"username"</span><span class="o">:</span> <span class="s2">"user@example.com"</span><span class="p">,</span>
  <span class="s2">"password"</span><span class="o">:</span> <span class="s2">"password"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>For this demo the table only contains username and password, but nothing prevents extending the response with additional data such as roles, permissions, or profile information.</p>

<h2>
  
  
  Key design choices
</h2>

<p>The plugin is designed not to interfere with administrators and not to block the native flow when username or password are empty: in those cases, WordPress handles everything as usual.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>What started as an experiment with metadata turned into a concrete demonstration of interoperability. My admin panel doesn't replace WordPress — it works alongside it: it makes it easy to create a management panel that extends WordPress data, but I discovered it can also integrate well via API.</p>

<p><a href="https://github.com/giuliopanda/milk-admin" rel="noopener noreferrer">Github project</a></p>

