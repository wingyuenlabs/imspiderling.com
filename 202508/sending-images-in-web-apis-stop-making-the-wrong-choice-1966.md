---
Title: Sending Images in Web APIs: Stop Making the Wrong Choice
Description: 
Author: Michal
Date: 2025-08-29T21:58:46.000Z
Robots: noindex,nofollow
Template: index
---
<p>You're building an API endpoint that accepts images. You have three main approaches, and most devs pick the wrong one. Let's fix that.</p>

<h2>
  
  
  1. Base64 Encoding (The Lazy Way)
</h2>

<p>Base64 converts binary data to text. Yes, it was useful for email attachments in 1995 when SMTP servers choked on binary data. Today? It's mostly a bad idea.</p>

<p><strong>The reality:</strong> Base64 inflates your payload by ~33%. That 3MB image becomes 4MB. Your API parsing that JSON? It's now handling massive strings. Your database storing these strings? Enjoy the bloat.</p>

<p><strong>When it's actually okay:</strong> Tiny icons or SVGs under 10KB embedded in CSS/HTML. That's it. If you're sending actual images this way, you're doing it wrong.</p>

<h2>
  
  
  2. Multipart/Form-Data (The Standard Way)
</h2>

<p>This is how HTML forms have sent files since forever. The catch? Everything that isn't a file becomes a string.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// What you send</span>
<span class="nx">formData</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="dl">'</span><span class="s1">file</span><span class="dl">'</span><span class="p">,</span> <span class="nx">imageFile</span><span class="p">);</span>
<span class="nx">formData</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="dl">'</span><span class="s1">metadata</span><span class="dl">'</span><span class="p">,</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">title</span><span class="p">:</span> <span class="dl">'</span><span class="s1">My Image</span><span class="dl">'</span><span class="p">,</span> <span class="na">tags</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">nature</span><span class="dl">'</span><span class="p">]</span> <span class="p">}));</span>
</code></pre>

</div>



<p><strong>The problem:</strong> Your nice DTOs and validation decorators in NestJS? Useless. You're parsing strings and validating manually. That <code>metadata</code> field? You're <code>JSON.parse()</code>-ing it and praying it's valid.</p>

<p><strong>When to use it:</strong> Simple uploads with minimal metadata. Profile picture update? Sure. Complex nested objects with files? Pain.</p>

<h2>
  
  
  3. Separate Endpoints (The Clean Way)
</h2>

<p>Upload files to one endpoint, get back IDs, then send those IDs with your JSON payload to another endpoint.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Step 1: Upload image</span>
<span class="nx">POST</span> <span class="o">/</span><span class="nx">api</span><span class="o">/</span><span class="nx">uploads</span>
<span class="nx">Response</span><span class="p">:</span> <span class="p">{</span> <span class="nl">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">abc123</span><span class="dl">"</span><span class="p">,</span> <span class="nx">url</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://...</span><span class="dl">"</span> <span class="p">}</span>

<span class="c1">// Step 2: Create resource with image reference</span>
<span class="nx">POST</span> <span class="o">/</span><span class="nx">api</span><span class="o">/</span><span class="nx">posts</span>
<span class="nx">Body</span><span class="p">:</span> <span class="p">{</span> <span class="nl">title</span><span class="p">:</span> <span class="dl">"</span><span class="s2">My Post</span><span class="dl">"</span><span class="p">,</span> <span class="nx">imageId</span><span class="p">:</span> <span class="dl">"</span><span class="s2">abc123</span><span class="dl">"</span><span class="p">,</span> <span class="p">...</span> <span class="p">}</span>
</code></pre>

</div>



<p><strong>The tradeoff:</strong> Yes, orphaned files. Write a cleanup job that runs every hour and deletes uploads older than 24 hours without associations. It's 20 lines of code, not rocket science.</p>

<p><strong>Why this wins:</strong> </p>

<ul>
<li>Clean separation of concerns</li>
<li>DTOs work normally</li>
<li>Better caching strategies</li>
<li>Resumable uploads become possible</li>
<li>Can optimize file handling separately (CDN, processing queues)</li>
</ul>

<h2>
  
  
  The Production Reality
</h2>

<p>Stop overthinking this. For 90% of apps, use separate endpoints. For simple forms with a single file, use multipart. Never use base64 for actual files unless you enjoy angry users and slow APIs.</p>

<p>And please, implement proper file validation. Check MIME types, file sizes, and actually validate the file content. Users will upload anything.</p>

<h2>
  
  
  Questions?
</h2>

<p>Got a different approach? Think I'm wrong about base64? Running into edge cases I didn't cover? Drop a comment. And if you're still sending 10MB PNGs as base64 strings in 2025, we need to talk.</p>

