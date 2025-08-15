---
Title: Real-Time Notifications in a MERN App — explained like texting your nerd bestie
Description: 
Author: Vaibhav Singh
Date: 2025-08-15T21:07:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>You want that sweet little bell to light up the moment <strong>payments</strong>, <strong>bookings</strong>, or <strong>admin broadcasts</strong> happen — and you want the list to still be there after refresh. This guide takes you <strong>from nothing</strong> to a working <strong>MERN</strong> stack with:</p>

<ul>
<li>
<strong>Socket.IO</strong> for instant in‑app updates</li>
<li>
<strong>MongoDB</strong> for persistence (history + unread)</li>
<li>
<strong>REST</strong> endpoints to fetch + mark read</li>
<li>
<strong>React</strong> hook + UI to display the dropdown</li>
</ul>

<p>Let’s ship. 🚀</p>




<h2>
  
  
  🧭 What we’re building
</h2>

<p><strong>Two lanes</strong> make notifications feel “right”:</p>

<ol>
<li>
<strong>Realtime lane</strong> → server emits Socket.IO events to the right users/roles.</li>
<li>
<strong>Persistence lane</strong> → server stores notifications in Mongo so they survive refresh/offline.</li>
</ol>

<p>The client:</p>

<ul>
<li>fetches last 50 notifications on mount,</li>
<li>listens for socket events and prepends new ones,</li>
<li>marks an item as read when clicked.</li>
</ul>

<blockquote>
<p>Result: <strong>real-time + durable</strong> notifications.</p>
</blockquote>




<h2>
  
  
  📦 Prereqs
</h2>

<ul>
<li>Node 18+</li>
<li>npm or pnpm</li>
<li>A MongoDB connection (local or Atlas).</li>
</ul>

<p>I’ll assume <code>mongodb://localhost:27017/mern_notify</code> locally.</p>




<h2>
  
  
  🗂️ Project layout
</h2>

<p>We’ll create two folders:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mern-realtime-notify/
  server/
  client/
</code></pre>

</div>






<h2>
  
  
  🛠️ Server (Express + Socket.IO + Mongo)
</h2>

<h3>
  
  
  1) Init + install
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> mern-realtime-notify/server <span class="o">&amp;&amp;</span> <span class="nb">cd </span>mern-realtime-notify/server
npm init <span class="nt">-y</span>
npm i express cors dotenv mongoose socket.io jsonwebtoken bcrypt
npm i <span class="nt">-D</span> nodemon
</code></pre>

</div>



<p>Add <strong>scripts</strong> to <code>package.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"server"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"module"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"dev"</span><span class="p">:</span><span class="w"> </span><span class="s2">"nodemon server.js"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Create <strong>.env</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>PORT=5000
MONGO_URI=mongodb://localhost:27017/mern_notify
JWT_SECRET=dev_super_secret_change_me
CORS_ORIGINS=http://localhost:5173
</code></pre>

</div>



<h3>
  
  
  2) Basic server with Socket.IO auth
</h3>

<p><strong>server.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">express</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">cors</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">cors</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">dotenv</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">dotenv</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">mongoose</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">mongoose</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createServer</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">http</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Server</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">socket.io</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">jwt</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">jsonwebtoken</span><span class="dl">'</span>

<span class="nx">dotenv</span><span class="p">.</span><span class="nf">config</span><span class="p">()</span>

<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">()</span>
<span class="kd">const</span> <span class="nx">server</span> <span class="o">=</span> <span class="nf">createServer</span><span class="p">(</span><span class="nx">app</span><span class="p">)</span>

<span class="kd">const</span> <span class="nx">allowed</span> <span class="o">=</span> <span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CORS_ORIGINS</span> <span class="o">||</span> <span class="dl">''</span><span class="p">).</span><span class="nf">split</span><span class="p">(</span><span class="dl">'</span><span class="s1">,</span><span class="dl">'</span><span class="p">).</span><span class="nf">filter</span><span class="p">(</span><span class="nb">Boolean</span><span class="p">)</span>
<span class="kd">const</span> <span class="nx">io</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Server</span><span class="p">(</span><span class="nx">server</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">cors</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">origin</span><span class="p">:</span> <span class="nx">allowed</span><span class="p">.</span><span class="nx">length</span> <span class="p">?</span> <span class="nx">allowed</span> <span class="p">:</span> <span class="dl">'</span><span class="s1">*</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">credentials</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">},</span>
  <span class="na">transports</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">websocket</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">polling</span><span class="dl">'</span><span class="p">],</span>
<span class="p">})</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nf">cors</span><span class="p">({</span> <span class="na">origin</span><span class="p">:</span> <span class="nx">allowed</span><span class="p">.</span><span class="nx">length</span> <span class="p">?</span> <span class="nx">allowed</span> <span class="p">:</span> <span class="dl">'</span><span class="s1">*</span><span class="dl">'</span><span class="p">,</span> <span class="na">credentials</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}))</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">json</span><span class="p">())</span>

<span class="c1">// connect mongo</span>
<span class="k">await</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">MONGO_URI</span><span class="p">)</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Mongo connected</span><span class="dl">'</span><span class="p">)</span>

<span class="c1">// ===== MODELS =====</span>
<span class="k">import</span> <span class="nx">Notification</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./src/models/Notification.js</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">User</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./src/models/User.js</span><span class="dl">'</span>

<span class="c1">// ===== ROUTES =====</span>
<span class="k">import</span> <span class="nx">authRouter</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./src/routes/auth.js</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">notifRouter</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./src/routes/notifications.js</span><span class="dl">'</span>

<span class="c1">// expose io to routes</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">((</span><span class="nx">req</span><span class="p">,</span> <span class="nx">_res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="nx">req</span><span class="p">.</span><span class="nx">io</span> <span class="o">=</span> <span class="nx">io</span><span class="p">;</span> <span class="nf">next</span><span class="p">()</span> <span class="p">})</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/auth</span><span class="dl">'</span><span class="p">,</span> <span class="nx">authRouter</span><span class="p">)</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/notifications</span><span class="dl">'</span><span class="p">,</span> <span class="nx">notifRouter</span><span class="p">)</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/health</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">_req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">ok</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}))</span>

<span class="c1">// ===== SOCKET AUTH =====</span>
<span class="nx">io</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="k">async </span><span class="p">(</span><span class="nx">socket</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="nx">socket</span><span class="p">.</span><span class="nx">handshake</span><span class="p">.</span><span class="nx">auth</span><span class="p">?.</span><span class="nx">token</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">token</span><span class="p">)</span> <span class="k">return</span> <span class="nf">next</span><span class="p">(</span><span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Missing token</span><span class="dl">'</span><span class="p">))</span>
    <span class="kd">const</span> <span class="nx">decoded</span> <span class="o">=</span> <span class="nx">jwt</span><span class="p">.</span><span class="nf">verify</span><span class="p">(</span><span class="nx">token</span><span class="p">,</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">JWT_SECRET</span><span class="p">)</span>
    <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">User</span><span class="p">.</span><span class="nf">findById</span><span class="p">(</span><span class="nx">decoded</span><span class="p">.</span><span class="nx">userId</span><span class="p">).</span><span class="nf">select</span><span class="p">(</span><span class="dl">'</span><span class="s1">-password</span><span class="dl">'</span><span class="p">)</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">user</span><span class="p">)</span> <span class="k">return</span> <span class="nf">next</span><span class="p">(</span><span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">User not found</span><span class="dl">'</span><span class="p">))</span>
    <span class="nx">socket</span><span class="p">.</span><span class="nx">user</span> <span class="o">=</span> <span class="nx">user</span>
    <span class="nf">next</span><span class="p">()</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">next</span><span class="p">(</span><span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Auth failed: </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">e</span><span class="p">.</span><span class="nx">message</span><span class="p">))</span>
  <span class="p">}</span>
<span class="p">})</span>

<span class="nx">io</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">connection</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">socket</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="nx">socket</span><span class="p">.</span><span class="nx">user</span>
  <span class="kd">const</span> <span class="nx">userRoom</span> <span class="o">=</span> <span class="s2">`user_</span><span class="p">${</span><span class="nx">user</span><span class="p">.</span><span class="nx">_id</span><span class="p">}</span><span class="s2">`</span>
  <span class="kd">const</span> <span class="nx">roleRoom</span> <span class="o">=</span> <span class="s2">`role_</span><span class="p">${</span><span class="nx">user</span><span class="p">.</span><span class="nx">role</span><span class="p">}</span><span class="s2">`</span>
  <span class="nx">socket</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">userRoom</span><span class="p">)</span>
  <span class="nx">socket</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">roleRoom</span><span class="p">)</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Socket connected: </span><span class="p">${</span><span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="p">}</span><span class="s2"> -&gt; rooms: </span><span class="p">${</span><span class="nx">userRoom</span><span class="p">}</span><span class="s2">, </span><span class="p">${</span><span class="nx">roleRoom</span><span class="p">}</span><span class="s2">`</span><span class="p">)</span>

  <span class="nx">socket</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">disconnect</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">reason</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Socket disconnected</span><span class="dl">'</span><span class="p">,</span> <span class="nx">reason</span><span class="p">)</span>
  <span class="p">})</span>
<span class="p">})</span>

<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">||</span> <span class="mi">5000</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Server listening on </span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">))</span>
</code></pre>

</div>



<h3>
  
  
  3) Models
</h3>

<p><strong>src/models/User.js</strong> (super minimal; don’t use in prod without tweaks)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">mongoose</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">mongoose</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">bcrypt</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">bcrypt</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">userSchema</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nc">Schema</span><span class="p">({</span>
  <span class="na">email</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span> <span class="na">unique</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
  <span class="na">password</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span>
  <span class="na">role</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span> <span class="na">default</span><span class="p">:</span> <span class="dl">'</span><span class="s1">patient</span><span class="dl">'</span> <span class="p">}</span> <span class="c1">// 'patient' | 'doctor' | 'admin'</span>
<span class="p">})</span>

<span class="nx">userSchema</span><span class="p">.</span><span class="nf">pre</span><span class="p">(</span><span class="dl">'</span><span class="s1">save</span><span class="dl">'</span><span class="p">,</span> <span class="k">async</span> <span class="kd">function</span><span class="p">(</span><span class="nx">next</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="k">this</span><span class="p">.</span><span class="nf">isModified</span><span class="p">(</span><span class="dl">'</span><span class="s1">password</span><span class="dl">'</span><span class="p">))</span> <span class="k">return</span> <span class="nf">next</span><span class="p">()</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">password</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">bcrypt</span><span class="p">.</span><span class="nf">hash</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span> <span class="mi">10</span><span class="p">)</span>
  <span class="nf">next</span><span class="p">()</span>
<span class="p">})</span>

<span class="nx">userSchema</span><span class="p">.</span><span class="nx">methods</span><span class="p">.</span><span class="nx">compare</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">pw</span><span class="p">)</span> <span class="p">{</span> <span class="k">return</span> <span class="nx">bcrypt</span><span class="p">.</span><span class="nf">compare</span><span class="p">(</span><span class="nx">pw</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">password</span><span class="p">)</span> <span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nf">model</span><span class="p">(</span><span class="dl">'</span><span class="s1">User</span><span class="dl">'</span><span class="p">,</span> <span class="nx">userSchema</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>src/models/Notification.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">mongoose</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">mongoose</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">notificationSchema</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nc">Schema</span><span class="p">(</span>
  <span class="p">{</span>
    <span class="na">userId</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nx">Schema</span><span class="p">.</span><span class="nx">Types</span><span class="p">.</span><span class="nx">ObjectId</span><span class="p">,</span> <span class="na">index</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
    <span class="na">type</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span> <span class="c1">// e.g., 'payment_success'</span>
    <span class="na">title</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span>
    <span class="na">message</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span>
    <span class="na">meta</span><span class="p">:</span> <span class="nb">Object</span><span class="p">,</span>
    <span class="na">readAt</span><span class="p">:</span> <span class="nb">Date</span><span class="p">,</span>
  <span class="p">},</span>
  <span class="p">{</span> <span class="na">timestamps</span><span class="p">:</span> <span class="p">{</span> <span class="na">createdAt</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="na">updatedAt</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}</span> <span class="p">}</span>
<span class="p">)</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nf">model</span><span class="p">(</span><span class="dl">'</span><span class="s1">Notification</span><span class="dl">'</span><span class="p">,</span> <span class="nx">notificationSchema</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  4) Auth routes (login/register + JWT)
</h3>

<p><strong>src/routes/auth.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">express</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">jwt</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">jsonwebtoken</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">User</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/User.js</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nx">express</span><span class="p">.</span><span class="nc">Router</span><span class="p">()</span>

<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/register</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">email</span><span class="p">,</span> <span class="nx">password</span><span class="p">,</span> <span class="nx">role</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">patient</span><span class="dl">'</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span>
  <span class="kd">const</span> <span class="nx">exists</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">User</span><span class="p">.</span><span class="nf">findOne</span><span class="p">({</span> <span class="nx">email</span> <span class="p">})</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">exists</span><span class="p">)</span> <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span> <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Email in use</span><span class="dl">'</span> <span class="p">})</span>
  <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">User</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span> <span class="nx">email</span><span class="p">,</span> <span class="nx">password</span><span class="p">,</span> <span class="nx">role</span> <span class="p">})</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="na">user</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">_id</span><span class="p">,</span> <span class="na">email</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span> <span class="na">role</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">role</span> <span class="p">}</span> <span class="p">})</span>
<span class="p">})</span>

<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">email</span><span class="p">,</span> <span class="nx">password</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span>
  <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">User</span><span class="p">.</span><span class="nf">findOne</span><span class="p">({</span> <span class="nx">email</span> <span class="p">})</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">user</span> <span class="o">||</span> <span class="o">!</span><span class="p">(</span><span class="k">await</span> <span class="nx">user</span><span class="p">.</span><span class="nf">compare</span><span class="p">(</span><span class="nx">password</span><span class="p">)))</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">401</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span> <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Invalid credentials</span><span class="dl">'</span> <span class="p">})</span>
  <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="nx">jwt</span><span class="p">.</span><span class="nf">sign</span><span class="p">({</span> <span class="na">userId</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">_id</span> <span class="p">},</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">JWT_SECRET</span><span class="p">,</span> <span class="p">{</span> <span class="na">expiresIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">7d</span><span class="dl">'</span> <span class="p">})</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="nx">token</span><span class="p">,</span> <span class="na">user</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">_id</span><span class="p">,</span> <span class="na">email</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span> <span class="na">role</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">role</span> <span class="p">}</span> <span class="p">})</span>
<span class="p">})</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">router</span>
</code></pre>

</div>



<h3>
  
  
  5) Notifications routes (fetch + markRead + test)
</h3>

<p><strong>src/routes/notifications.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">express</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">jwt</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">jsonwebtoken</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">Notification</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../models/Notification.js</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nx">express</span><span class="p">.</span><span class="nc">Router</span><span class="p">()</span>

<span class="c1">// simple auth middleware reading Bearer token</span>
<span class="kd">function</span> <span class="nf">authenticate</span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">h</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nx">authorization</span> <span class="o">||</span> <span class="dl">''</span>
    <span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="nx">h</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">Bearer </span><span class="dl">'</span><span class="p">)</span> <span class="p">?</span> <span class="nx">h</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">7</span><span class="p">)</span> <span class="p">:</span> <span class="kc">null</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">token</span><span class="p">)</span> <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">401</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span> <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">No token</span><span class="dl">'</span> <span class="p">})</span>
    <span class="kd">const</span> <span class="nx">decoded</span> <span class="o">=</span> <span class="nx">jwt</span><span class="p">.</span><span class="nf">verify</span><span class="p">(</span><span class="nx">token</span><span class="p">,</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">JWT_SECRET</span><span class="p">)</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">user</span> <span class="o">=</span> <span class="p">{</span> <span class="na">_id</span><span class="p">:</span> <span class="nx">decoded</span><span class="p">.</span><span class="nx">userId</span> <span class="p">}</span>
    <span class="nf">next</span><span class="p">()</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">401</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span> <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Invalid token</span><span class="dl">'</span> <span class="p">})</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// GET: latest 50</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">,</span> <span class="nx">authenticate</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">list</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Notification</span><span class="p">.</span><span class="nf">find</span><span class="p">({</span> <span class="na">userId</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nx">_id</span> <span class="p">}).</span><span class="nf">sort</span><span class="p">({</span> <span class="na">createdAt</span><span class="p">:</span> <span class="o">-</span><span class="mi">1</span> <span class="p">}).</span><span class="nf">limit</span><span class="p">(</span><span class="mi">50</span><span class="p">).</span><span class="nf">lean</span><span class="p">()</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="na">notifications</span><span class="p">:</span> <span class="nx">list</span> <span class="p">})</span>
<span class="p">})</span>

<span class="c1">// POST: mark read</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/read</span><span class="dl">'</span><span class="p">,</span> <span class="nx">authenticate</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">id</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span>
  <span class="k">await</span> <span class="nx">Notification</span><span class="p">.</span><span class="nf">updateOne</span><span class="p">({</span> <span class="na">_id</span><span class="p">:</span> <span class="nx">id</span><span class="p">,</span> <span class="na">userId</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nx">_id</span> <span class="p">},</span> <span class="p">{</span> <span class="na">$set</span><span class="p">:</span> <span class="p">{</span> <span class="na">readAt</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">()</span> <span class="p">}</span> <span class="p">})</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
<span class="p">})</span>

<span class="c1">// POST: test -&gt; store + emit</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/test</span><span class="dl">'</span><span class="p">,</span> <span class="nx">authenticate</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">message</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">This is a test</span><span class="dl">'</span><span class="p">,</span> <span class="nx">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">info</span><span class="dl">'</span><span class="p">,</span> <span class="nx">title</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">Test notification</span><span class="dl">'</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span>
  <span class="kd">const</span> <span class="nx">doc</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Notification</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span> <span class="na">userId</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nx">_id</span><span class="p">,</span> <span class="nx">type</span><span class="p">,</span> <span class="nx">title</span><span class="p">,</span> <span class="nx">message</span> <span class="p">})</span>
  <span class="nx">req</span><span class="p">.</span><span class="nx">io</span><span class="p">.</span><span class="nf">to</span><span class="p">(</span><span class="s2">`user_</span><span class="p">${</span><span class="nx">req</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nx">_id</span><span class="p">}</span><span class="s2">`</span><span class="p">).</span><span class="nf">emit</span><span class="p">(</span><span class="dl">'</span><span class="s1">notification</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">_id</span><span class="p">:</span> <span class="nx">doc</span><span class="p">.</span><span class="nx">_id</span><span class="p">,</span> <span class="nx">type</span><span class="p">,</span> <span class="nx">title</span><span class="p">,</span> <span class="nx">message</span><span class="p">,</span> <span class="na">createdAt</span><span class="p">:</span> <span class="nx">doc</span><span class="p">.</span><span class="nx">createdAt</span><span class="p">,</span> <span class="na">readAt</span><span class="p">:</span> <span class="kc">null</span>
  <span class="p">})</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span> <span class="p">})</span>
<span class="p">})</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">router</span>
</code></pre>

</div>



<p>Start the server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm run dev
</code></pre>

</div>






<h2>
  
  
  🎨 Client (React + Vite)
</h2>

<h3>
  
  
  1) Init + install
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd</span> ../
npm create vite@latest client <span class="nt">--</span> <span class="nt">--template</span> react
<span class="nb">cd </span>client
npm i socket.io-client axios react-hot-toast
</code></pre>

</div>



<p>Add <strong>.env</strong> (Vite expects <code>VITE_</code> prefix):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>VITE_API_URL=http://localhost:5000/api
VITE_SOCKET_URL=http://localhost:5000
</code></pre>

</div>



<h3>
  
  
  2) Minimal API helper
</h3>

<p><strong>src/lib/api.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">axios</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">axios</span><span class="dl">'</span>
<span class="kd">const</span> <span class="nx">api</span> <span class="o">=</span> <span class="nx">axios</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span> <span class="na">baseURL</span><span class="p">:</span> <span class="k">import</span><span class="p">.</span><span class="nx">meta</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">VITE_API_URL</span> <span class="p">})</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">setAuth</span><span class="p">(</span><span class="nx">token</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">api</span><span class="p">.</span><span class="nx">defaults</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nx">common</span><span class="p">[</span><span class="dl">'</span><span class="s1">Authorization</span><span class="dl">'</span><span class="p">]</span> <span class="o">=</span> <span class="nx">token</span> <span class="p">?</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">token</span><span class="p">}</span><span class="s2">`</span> <span class="p">:</span> <span class="kc">undefined</span>
<span class="p">}</span>
<span class="k">export</span> <span class="k">default</span> <span class="nx">api</span>
</code></pre>

</div>



<h3>
  
  
  3) Auth context (store token + user)
</h3>

<p><strong>src/contexts/AuthContext.jsx</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createContext</span><span class="p">,</span> <span class="nx">useContext</span><span class="p">,</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">api</span><span class="p">,</span> <span class="p">{</span> <span class="nx">setAuth</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../lib/api</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">AuthCtx</span> <span class="o">=</span> <span class="nf">createContext</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">useAuth</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">useContext</span><span class="p">(</span><span class="nx">AuthCtx</span><span class="p">)</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">AuthProvider</span><span class="p">({</span> <span class="nx">children</span> <span class="p">})</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">user</span><span class="p">,</span> <span class="nx">setUser</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">token</span><span class="p">,</span> <span class="nx">setToken</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>

  <span class="kd">const</span> <span class="nx">login</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">email</span><span class="p">,</span> <span class="nx">password</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">api</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/auth/login</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="nx">email</span><span class="p">,</span> <span class="nx">password</span> <span class="p">})</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">success</span><span class="p">)</span> <span class="p">{</span>
      <span class="nf">setToken</span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">token</span><span class="p">)</span>
      <span class="nf">setAuth</span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">token</span><span class="p">)</span>
      <span class="nf">setUser</span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">user</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nx">data</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">register</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">email</span><span class="p">,</span> <span class="nx">password</span><span class="p">,</span> <span class="nx">role</span><span class="o">=</span><span class="dl">'</span><span class="s1">patient</span><span class="dl">'</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">api</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/auth/register</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="nx">email</span><span class="p">,</span> <span class="nx">password</span><span class="p">,</span> <span class="nx">role</span> <span class="p">})</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nx">data</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">logout</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">setToken</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span> <span class="nf">setUser</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span> <span class="nf">setAuth</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>
  <span class="p">}</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">AuthCtx</span><span class="p">.</span><span class="nc">Provider</span> <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="nx">user</span><span class="p">,</span> <span class="nx">token</span><span class="p">,</span> <span class="nx">login</span><span class="p">,</span> <span class="nx">register</span><span class="p">,</span> <span class="nx">logout</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
      <span class="si">{</span><span class="nx">children</span><span class="si">}</span>
    <span class="p">&lt;/</span><span class="nc">AuthCtx</span><span class="p">.</span><span class="nc">Provider</span><span class="p">&gt;</span>
  <span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  4) Socket context (auth handshake)
</h3>

<p><strong>src/contexts/SocketContext.jsx</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createContext</span><span class="p">,</span> <span class="nx">useContext</span><span class="p">,</span> <span class="nx">useEffect</span><span class="p">,</span> <span class="nx">useRef</span><span class="p">,</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">io</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">socket.io-client</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useAuth</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./AuthContext</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">SocketCtx</span> <span class="o">=</span> <span class="nf">createContext</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">useSocket</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">useContext</span><span class="p">(</span><span class="nx">SocketCtx</span><span class="p">)</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">SocketProvider</span><span class="p">({</span> <span class="nx">children</span> <span class="p">})</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">token</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useAuth</span><span class="p">()</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">isConnected</span><span class="p">,</span> <span class="nx">setIsConnected</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="kc">false</span><span class="p">)</span>
  <span class="kd">const</span> <span class="nx">socketRef</span> <span class="o">=</span> <span class="nf">useRef</span><span class="p">(</span><span class="kc">null</span><span class="p">)</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">token</span><span class="p">)</span> <span class="k">return</span>
    <span class="kd">const</span> <span class="nx">socket</span> <span class="o">=</span> <span class="nf">io</span><span class="p">(</span><span class="k">import</span><span class="p">.</span><span class="nx">meta</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">VITE_SOCKET_URL</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">auth</span><span class="p">:</span> <span class="p">{</span> <span class="nx">token</span> <span class="p">},</span>
      <span class="na">transports</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">websocket</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">polling</span><span class="dl">'</span><span class="p">],</span>
    <span class="p">})</span>
    <span class="nx">socketRef</span><span class="p">.</span><span class="nx">current</span> <span class="o">=</span> <span class="nx">socket</span>
    <span class="nx">socket</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">connect</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">setIsConnected</span><span class="p">(</span><span class="kc">true</span><span class="p">))</span>
    <span class="nx">socket</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">disconnect</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">setIsConnected</span><span class="p">(</span><span class="kc">false</span><span class="p">))</span>
    <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">socket</span><span class="p">.</span><span class="nf">disconnect</span><span class="p">()</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">token</span><span class="p">])</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">SocketCtx</span><span class="p">.</span><span class="nc">Provider</span> <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">socket</span><span class="p">:</span> <span class="nx">socketRef</span><span class="p">.</span><span class="nx">current</span><span class="p">,</span> <span class="nx">isConnected</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
      <span class="si">{</span><span class="nx">children</span><span class="si">}</span>
    <span class="p">&lt;/</span><span class="nc">SocketCtx</span><span class="p">.</span><span class="nc">Provider</span><span class="p">&gt;</span>
  <span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  5) Notifications hook (fetch + live + markRead)
</h3>

<p><strong>src/hooks/useNotifications.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useEffect</span><span class="p">,</span> <span class="nx">useMemo</span><span class="p">,</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">api</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../lib/api</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useSocket</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../contexts/SocketContext</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">EVENTS</span> <span class="o">=</span> <span class="p">[</span>
  <span class="dl">'</span><span class="s1">notification</span><span class="dl">'</span><span class="p">,</span> <span class="c1">// generic catch-all we emit in /notifications/test</span>
  <span class="c1">// add more if your server emits: 'payment_success', 'new_appointment', etc.</span>
<span class="p">]</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">useNotifications</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">socket</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useSocket</span><span class="p">()</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">items</span><span class="p">,</span> <span class="nx">setItems</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">([])</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="nx">dead</span> <span class="o">=</span> <span class="kc">false</span>
    <span class="p">;(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">api</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/notifications</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">dead</span> <span class="o">&amp;&amp;</span> <span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">?.</span><span class="nx">success</span><span class="p">)</span> <span class="nf">setItems</span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">notifications</span><span class="p">)</span>
      <span class="p">}</span> <span class="k">catch</span> <span class="p">{}</span>
    <span class="p">})()</span>
    <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="nx">dead</span> <span class="o">=</span> <span class="kc">true</span> <span class="p">}</span>
  <span class="p">},</span> <span class="p">[])</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">socket</span><span class="p">)</span> <span class="k">return</span>
    <span class="kd">const</span> <span class="nx">on</span> <span class="o">=</span> <span class="p">(</span><span class="nx">type</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="nx">p</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">setItems</span><span class="p">(</span><span class="nx">prev</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="nf">normalize</span><span class="p">(</span><span class="nx">type</span><span class="p">,</span> <span class="nx">p</span><span class="p">),</span> <span class="p">...</span><span class="nx">prev</span><span class="p">])</span>
    <span class="nx">EVENTS</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">evt</span> <span class="o">=&gt;</span> <span class="nx">socket</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="nx">evt</span><span class="p">,</span> <span class="nf">on</span><span class="p">(</span><span class="nx">evt</span><span class="p">)))</span>
    <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="nx">EVENTS</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">evt</span> <span class="o">=&gt;</span> <span class="nx">socket</span><span class="p">.</span><span class="nf">off</span><span class="p">(</span><span class="nx">evt</span><span class="p">))</span> <span class="p">}</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">socket</span><span class="p">])</span>

  <span class="kd">const</span> <span class="nx">unreadCount</span> <span class="o">=</span> <span class="nf">useMemo</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">items</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">n</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">n</span><span class="p">.</span><span class="nx">readAt</span><span class="p">).</span><span class="nx">length</span><span class="p">,</span> <span class="p">[</span><span class="nx">items</span><span class="p">])</span>

  <span class="kd">const</span> <span class="nx">markRead</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">id</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">setItems</span><span class="p">(</span><span class="nx">prev</span> <span class="o">=&gt;</span> <span class="nx">prev</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">n</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="nx">n</span><span class="p">.</span><span class="nx">_id</span> <span class="o">===</span> <span class="nx">id</span> <span class="p">?</span> <span class="p">{</span> <span class="p">...</span><span class="nx">n</span><span class="p">,</span> <span class="na">readAt</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">()</span> <span class="p">}</span> <span class="p">:</span> <span class="nx">n</span><span class="p">)))</span>
    <span class="k">try</span> <span class="p">{</span> <span class="k">await</span> <span class="nx">api</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/notifications/read</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="nx">id</span> <span class="p">})</span> <span class="p">}</span> <span class="k">catch</span> <span class="p">{}</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">items</span><span class="p">,</span> <span class="nx">unreadCount</span><span class="p">,</span> <span class="nx">markRead</span> <span class="p">}</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">normalize</span><span class="p">(</span><span class="nx">type</span><span class="p">,</span> <span class="nx">p</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="na">_id</span><span class="p">:</span> <span class="nx">p</span><span class="p">.</span><span class="nx">_id</span> <span class="o">||</span> <span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">().</span><span class="nf">toString</span><span class="p">(</span><span class="mi">36</span><span class="p">).</span><span class="nf">slice</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span> <span class="o">+</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">().</span><span class="nf">toString</span><span class="p">(</span><span class="mi">36</span><span class="p">)),</span>
    <span class="nx">type</span><span class="p">,</span>
    <span class="na">title</span><span class="p">:</span> <span class="nx">p</span><span class="p">.</span><span class="nx">title</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">Notification</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">message</span><span class="p">:</span> <span class="nx">p</span><span class="p">.</span><span class="nx">message</span> <span class="o">||</span> <span class="dl">''</span><span class="p">,</span>
    <span class="na">createdAt</span><span class="p">:</span> <span class="nx">p</span><span class="p">.</span><span class="nx">createdAt</span> <span class="o">||</span> <span class="nx">p</span><span class="p">.</span><span class="nx">timestamp</span> <span class="o">||</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
    <span class="na">readAt</span><span class="p">:</span> <span class="nx">p</span><span class="p">.</span><span class="nx">readAt</span> <span class="o">||</span> <span class="kc">null</span><span class="p">,</span>
    <span class="p">...</span><span class="nx">p</span><span class="p">,</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  6) Tiny UI (login + bell)
</h3>

<p><strong>src/App.jsx</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">AuthProvider</span><span class="p">,</span> <span class="p">{</span> <span class="nx">useAuth</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./contexts/AuthContext</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">SocketProvider</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./contexts/SocketContext</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useNotifications</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./hooks/useNotifications</span><span class="dl">'</span>

<span class="kd">function</span> <span class="nf">Login</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">login</span><span class="p">,</span> <span class="nx">register</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useAuth</span><span class="p">()</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">email</span><span class="p">,</span> <span class="nx">setEmail</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">'</span><span class="s1">demo@example.com</span><span class="dl">'</span><span class="p">)</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">password</span><span class="p">,</span> <span class="nx">setPassword</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">'</span><span class="s1">secret</span><span class="dl">'</span><span class="p">)</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">display</span><span class="p">:</span><span class="dl">'</span><span class="s1">flex</span><span class="dl">'</span><span class="p">,</span> <span class="na">gap</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span> <span class="na">marginBottom</span><span class="p">:</span> <span class="mi">16</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">input</span> <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">email</span><span class="si">}</span> <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="nx">e</span><span class="o">=&gt;</span><span class="nf">setEmail</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">value</span><span class="p">)</span><span class="si">}</span> <span class="na">placeholder</span><span class="p">=</span><span class="s">"email"</span> <span class="p">/&gt;</span>
      <span class="p">&lt;</span><span class="nt">input</span> <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">password</span><span class="si">}</span> <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="nx">e</span><span class="o">=&gt;</span><span class="nf">setPassword</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">value</span><span class="p">)</span><span class="si">}</span> <span class="na">placeholder</span><span class="p">=</span><span class="s">"password"</span> <span class="na">type</span><span class="p">=</span><span class="s">"password"</span> <span class="p">/&gt;</span>
      <span class="p">&lt;</span><span class="nt">button</span> <span class="na">onClick</span><span class="p">=</span><span class="si">{</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">register</span><span class="p">(</span><span class="nx">email</span><span class="p">,</span> <span class="nx">password</span><span class="p">)</span><span class="si">}</span><span class="p">&gt;</span>Register<span class="p">&lt;/</span><span class="nt">button</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">button</span> <span class="na">onClick</span><span class="p">=</span><span class="si">{</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">login</span><span class="p">(</span><span class="nx">email</span><span class="p">,</span> <span class="nx">password</span><span class="p">)</span><span class="si">}</span><span class="p">&gt;</span>Login<span class="p">&lt;/</span><span class="nt">button</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
  <span class="p">)</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">Bell</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">items</span><span class="p">,</span> <span class="nx">unreadCount</span><span class="p">,</span> <span class="nx">markRead</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useNotifications</span><span class="p">()</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">button</span><span class="p">&gt;</span>🔔 <span class="si">{</span><span class="nx">unreadCount</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="p">?</span> <span class="s2">`(</span><span class="p">${</span><span class="nx">unreadCount</span><span class="p">}</span><span class="s2">)`</span> <span class="p">:</span> <span class="dl">''</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">button</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">border</span><span class="p">:</span><span class="dl">'</span><span class="s1">1px solid #ddd</span><span class="dl">'</span><span class="p">,</span> <span class="na">padding</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span> <span class="na">width</span><span class="p">:</span> <span class="mi">360</span><span class="p">,</span> <span class="na">marginTop</span><span class="p">:</span> <span class="mi">8</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">items</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">n</span> <span class="o">=&gt;</span> <span class="p">(</span>
          <span class="p">&lt;</span><span class="nt">div</span> <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">n</span><span class="p">.</span><span class="nx">_id</span><span class="si">}</span> <span class="na">onClick</span><span class="p">=</span><span class="si">{</span><span class="p">()</span><span class="o">=&gt;</span><span class="nf">markRead</span><span class="p">(</span><span class="nx">n</span><span class="p">.</span><span class="nx">_id</span><span class="p">)</span><span class="si">}</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">padding</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span> <span class="na">borderBottom</span><span class="p">:</span><span class="dl">'</span><span class="s1">1px solid #eee</span><span class="dl">'</span><span class="p">,</span> <span class="na">cursor</span><span class="p">:</span><span class="dl">'</span><span class="s1">pointer</span><span class="dl">'</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
            <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">fontWeight</span><span class="p">:</span><span class="dl">'</span><span class="s1">bold</span><span class="dl">'</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">n</span><span class="p">.</span><span class="nx">title</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
            <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">fontSize</span><span class="p">:</span> <span class="mi">12</span><span class="p">,</span> <span class="na">opacity</span><span class="p">:.</span><span class="mi">7</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span><span class="si">{</span><span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="nx">n</span><span class="p">.</span><span class="nx">createdAt</span><span class="p">).</span><span class="nf">toLocaleString</span><span class="p">()</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
            <span class="si">{</span><span class="nx">n</span><span class="p">.</span><span class="nx">message</span> <span class="o">&amp;&amp;</span> <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">fontSize</span><span class="p">:</span> <span class="mi">13</span><span class="p">,</span> <span class="na">marginTop</span><span class="p">:</span> <span class="mi">4</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">n</span><span class="p">.</span><span class="nx">message</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span><span class="si">}</span>
          <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
        <span class="p">))</span><span class="si">}</span>
      <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
  <span class="p">)</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">App</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">AuthProvider</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">SocketProvider</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">padding</span><span class="p">:</span> <span class="mi">24</span><span class="p">,</span> <span class="na">fontFamily</span><span class="p">:</span> <span class="dl">'</span><span class="s1">sans-serif</span><span class="dl">'</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
          <span class="p">&lt;</span><span class="nt">h1</span><span class="p">&gt;</span>MERN Real-time Notifications<span class="p">&lt;/</span><span class="nt">h1</span><span class="p">&gt;</span>
          <span class="p">&lt;</span><span class="nc">Login</span> <span class="p">/&gt;</span>
          <span class="p">&lt;</span><span class="nc">Bell</span> <span class="p">/&gt;</span>
        <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
      <span class="p">&lt;/</span><span class="nc">SocketProvider</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nc">AuthProvider</span><span class="p">&gt;</span>
  <span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Run the client:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm run dev
</code></pre>

</div>



<p>Open <a href="http://localhost:5173/" rel="noopener noreferrer">http://localhost:5173/</a>. Register, then login.</p>




<h2>
  
  
  🧪 Test it (Does it ding?)
</h2>

<p>With the client logged in, hit the test endpoint from a terminal:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:5000/api/notifications/test <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer YOUR_JWT_HERE"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"message":"Hello from test!","type":"info"}'</span>
</code></pre>

</div>



<p>You should see a new row appear under the bell <strong>instantly</strong>. Click it → it’s marked read. Refresh → it’s still there. ✅</p>




<h2>
  
  
  🧯 Common gotchas
</h2>

<ul>
<li>
<strong>CORS</strong>: Allow your client origin on <strong>both</strong> Express and Socket.IO CORS.</li>
<li>
<strong>Auth handshake</strong>: Send <code>auth: { token }</code> when connecting the socket (we did).</li>
<li>
<strong>Cleanup</strong>: Always <code>off()</code> socket listeners on unmount (the hook does).</li>
<li>
<strong>Reverse proxy</strong>: Enable WebSocket upgrades on Nginx/Cloudflare/etc.</li>
<li>
<strong>Security</strong>: On mark-read, always filter by <code>{ _id, userId }</code> (we do).</li>
</ul>




<h2>
  
  
  🧰 Extras (when you want more)
</h2>

<ul>
<li>
<strong>Role broadcasts</strong>: Server joins users to <code>role_admin</code>, <code>role_doctor</code>, <code>role_patient</code>. Emit to a role room.</li>
<li>
<strong>Scheduled reminders</strong>: Add Redis + BullMQ → schedule “appointment in 1h” jobs, emit at the right time.</li>
<li>
<strong>Web Push/FCM</strong>: For OS-level notifications when the tab is closed.</li>
<li>
<strong>Hosted websockets</strong>: Pusher/Ably/PubNub if you don’t want to run Socket.IO infra.</li>
<li>
<strong>Emails/SMS</strong>: SendGrid/SES and Twilio for critical backups.</li>
</ul>




<h2>
  
  
  🏁 Recap
</h2>

<ul>
<li>Mongo stores notifications.
</li>
<li>Socket.IO delivers them in real-time.
</li>
<li>React hook keeps UI in sync and handles “mark read”.
</li>
</ul>

<p>This pattern is tiny, fast, and rock-solid. You can paste it into any MERN app and be “the person who made the bell work” by lunch. 🔔✨</p>

