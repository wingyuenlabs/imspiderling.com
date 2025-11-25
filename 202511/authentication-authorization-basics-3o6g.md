---
Title: ✅ *Authentication & Authorization Basics* 🔐🌐
Description: 
Author: ssekabira robert sims
Date: 2025-11-25T21:04:20.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>🔹 What is Authentication?</em><br><br>
It’s the process of verifying <em>who</em> a user is.</p>

<p><em>🔹 What is Authorization?</em><br><br>
It’s the process of verifying <em>what</em> a user is allowed to do after logging in.</p>

<p><em>✅ Step 1: Authentication – Common Methods</em><br><br>
• <em>Username &amp; Password</em> – Basic login<br><br>
• <em>OAuth</em> – Login via Google, GitHub, etc.<br><br>
• <em>JWT (JSON Web Token)</em> – Popular for token-based auth<br><br>
• <em>Session-Based</em> – Stores session on server with session ID</p>

<p><em>✅ Step 2: How Login Works (JWT Example)</em>  </p>

<ol>
<li>User sends email &amp; password to server
</li>
<li>Server verifies and sends back a JWT
</li>
<li>JWT is stored in browser (usually localStorage)
</li>
<li>On each request, client sends JWT in headers
</li>
<li>Server checks token before giving access</li>
</ol>

<p><em>✅ Step 3: Authorization Types</em><br><br>
• <em>Role-Based Access</em> – Admin, Editor, User<br><br>
• <em>Resource-Based</em> – Only owners can edit their content<br><br>
• <em>Route Protection</em> – Block some pages unless logged in</p>

<p><em>✅ Step 4: Protecting Routes (Frontend Example)</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">localStorage</span><span class="p">.</span><span class="nf">getItem</span><span class="p">(</span><span class="dl">'</span><span class="s1">token</span><span class="dl">'</span><span class="p">))</span> <span class="p">{</span>
  <span class="nb">window</span><span class="p">.</span><span class="nx">location</span><span class="p">.</span><span class="nx">href</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p><em>✅ Step 5: Backend Route Protection (Express.js)</em><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">authMiddleware</span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nx">authorization</span><span class="p">;</span>
<span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">token</span><span class="p">)</span> <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">401</span><span class="p">).</span><span class="nf">send</span><span class="p">(</span><span class="dl">'</span><span class="s1">Access Denied</span><span class="dl">'</span><span class="p">);</span>
  <span class="c1">// Verify token and decode user info</span>
  <span class="nf">next</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p><em>✅ Step 6: Common Tools &amp; Libraries</em><br><br>
• <em>bcrypt</em> – Hash passwords<br><br>
• <em>jsonwebtoken (JWT)</em> – Create &amp; verify tokens<br><br>
• <em>passport.js</em> – Auth middleware<br><br>
• <em>OAuth Providers</em> – Google, Facebook, GitHub</p>

<p><em>✅ Step 7: Best Practices</em><br><br>
• Always hash passwords (never store plain text)<br><br>
• Use HTTPS<br><br>
• Set token expiry (e.g. 15 mins)<br><br>
• Refresh tokens securely<br><br>
• Don't expose sensitive data in JWT</p>

<p>💬 <em>and like for more</em></p>

