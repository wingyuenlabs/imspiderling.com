---
Title: Express + MySQL Boilerplate Project Structure (Real App Ready)
Description: 
Author: Sospeter Mong'are
Date: 2025-11-07T21:38:43.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>project-root/
│
├── src/
│   ├── config/
│   │   ├── db.js
│   │   └── index.js
│   │
│   ├── controllers/
│   │   ├── AuthController.js
│   │   ├── UserController.js
│   │   └── ExampleController.js
│   │
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── user.routes.js
│   │   └── index.js
│   │
│   ├── services/
│   │   ├── AuthService.js
│   │   ├── UserService.js
│   │   └── ExampleService.js
│   │
│   ├── models/
│   │   ├── User.js
│   │   ├── index.js
│   │   └── migrations/
│   │       └── 001_create_users_table.sql
│   │
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   ├── errorHandler.js
│   │   └── validateRequest.js
│   │
│   ├── utils/
│   │   ├── logger.js
│   │   └── helpers.js
│   │
│   ├── app.js
│   └── server.js
│
├── .env
├── .gitignore
├── package.json
└── README.md
</code></pre>

</div>






<h1>
  
  
  ✅ <strong>Folder Breakdown: What Each Part Does</strong>
</h1>

<h2>
  
  
  <strong>1. config/</strong>
</h2>

<p>This is the control tower.<br>
Houses database and environment configurations.</p>

<p><strong>db.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">mysql</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">mysql2/promise</span><span class="dl">"</span><span class="p">);</span>
<span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">dotenv</span><span class="dl">"</span><span class="p">).</span><span class="nf">config</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">pool</span> <span class="o">=</span> <span class="nx">mysql</span><span class="p">.</span><span class="nf">createPool</span><span class="p">({</span>
  <span class="na">host</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_HOST</span><span class="p">,</span>
  <span class="na">user</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_USER</span><span class="p">,</span>
  <span class="na">password</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_PASS</span><span class="p">,</span>
  <span class="na">database</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_NAME</span><span class="p">,</span>
  <span class="na">waitForConnections</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">connectionLimit</span><span class="p">:</span> <span class="mi">10</span>
<span class="p">});</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="nx">pool</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  <strong>2. controllers/</strong>
</h2>

<p>Controllers are the “traffic cops.”<br>
They receive requests, call the service layer, and return responses.</p>

<p><strong>UserController.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">UserService</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">../services/UserService</span><span class="dl">"</span><span class="p">);</span>

<span class="nx">exports</span><span class="p">.</span><span class="nx">createUser</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">UserService</span><span class="p">.</span><span class="nf">createUser</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">);</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">201</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="nx">data</span> <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">next</span><span class="p">(</span><span class="nx">err</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>






<h2>
  
  
  <strong>3. routes/</strong>
</h2>

<p>Routes connect URLs to controllers.<br>
Keeps app.js clean and modular.</p>

<p><strong>user.routes.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">express</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nx">express</span><span class="p">.</span><span class="nc">Router</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">UserController</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">../controllers/UserController</span><span class="dl">"</span><span class="p">);</span>

<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">"</span><span class="s2">/</span><span class="dl">"</span><span class="p">,</span> <span class="nx">UserController</span><span class="p">.</span><span class="nx">createUser</span><span class="p">);</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">/</span><span class="dl">"</span><span class="p">,</span> <span class="nx">UserController</span><span class="p">.</span><span class="nx">getAllUsers</span><span class="p">);</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="nx">router</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>index.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">).</span><span class="nc">Router</span><span class="p">();</span>

<span class="nx">router</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="dl">"</span><span class="s2">/users</span><span class="dl">"</span><span class="p">,</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./user.routes</span><span class="dl">"</span><span class="p">));</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="dl">"</span><span class="s2">/auth</span><span class="dl">"</span><span class="p">,</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./auth.routes</span><span class="dl">"</span><span class="p">));</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="nx">router</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  <strong>4. services/</strong>
</h2>

<p>Business logic lives here.<br>
Services talk to models and keep controllers lightweight.</p>

<p><strong>UserService.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">../config/db</span><span class="dl">"</span><span class="p">);</span>

<span class="nx">exports</span><span class="p">.</span><span class="nx">createUser</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">result</span><span class="p">]</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span>
    <span class="dl">"</span><span class="s2">INSERT INTO users (name, email) VALUES (?, ?)</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">[</span><span class="nx">data</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span> <span class="nx">data</span><span class="p">.</span><span class="nx">email</span><span class="p">]</span>
  <span class="p">);</span>

  <span class="k">return</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="nx">result</span><span class="p">.</span><span class="nx">insertId</span><span class="p">,</span> <span class="p">...</span><span class="nx">data</span> <span class="p">};</span>
<span class="p">};</span>
</code></pre>

</div>






<h2>
  
  
  <strong>5. models/</strong>
</h2>

<p>Models represent database access logic.</p>

<p><strong>User.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">../config/db</span><span class="dl">"</span><span class="p">);</span>

<span class="nx">exports</span><span class="p">.</span><span class="nx">findAll</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">db</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="dl">"</span><span class="s2">SELECT * FROM users</span><span class="dl">"</span><span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>






<h2>
  
  
  <strong>6. middleware/</strong>
</h2>

<p>Reusable logic that sits between request and controller.</p>

<p>Examples:</p>

<ul>
<li>Authentication</li>
<li>Request validation</li>
<li>Error handling</li>
</ul>

<p><strong>authMiddleware.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">req</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nx">authorization</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">401</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Unauthorized</span><span class="dl">"</span> <span class="p">});</span>
  <span class="p">}</span>
  <span class="nf">next</span><span class="p">();</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>errorHandler.js</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">(</span><span class="nx">err</span><span class="p">,</span> <span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
    <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
    <span class="na">message</span><span class="p">:</span> <span class="nx">err</span><span class="p">.</span><span class="nx">message</span> <span class="o">||</span> <span class="dl">"</span><span class="s2">Internal Server Error</span><span class="dl">"</span>
  <span class="p">});</span>
<span class="p">};</span>
</code></pre>

</div>






<h2>
  
  
  <strong>7. utils/</strong>
</h2>

<p>Helper functions, loggers, etc.</p>




<h2>
  
  
  <strong>8. app.js &amp; server.js</strong>
</h2>

<h3>
  
  
  <strong>app.js</strong>
</h3>

<p>Assembles routes and middleware.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">express</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">routes</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./routes</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">errorHandler</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./middleware/errorHandler</span><span class="dl">"</span><span class="p">);</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">json</span><span class="p">());</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="dl">"</span><span class="s2">/api</span><span class="dl">"</span><span class="p">,</span> <span class="nx">routes</span><span class="p">);</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">errorHandler</span><span class="p">);</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="nx">app</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  <strong>server.js</strong>
</h3>

<p>Runs the app.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">dotenv</span><span class="dl">"</span><span class="p">).</span><span class="nf">config</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./app</span><span class="dl">"</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">||</span> <span class="mi">3000</span><span class="p">;</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Server running on port </span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>






<h1>
  
  
  ✅ Bonus: <code>.env</code> Example
</h1>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>PORT=3000
DB_HOST=localhost
DB_USER=root
DB_PASS=secret123
DB_NAME=express_app
</code></pre>

</div>






<h1>
  
  
  ✅ Why This Structure Works in Real Apps
</h1>

<p>✅ Separates logic by responsibility<br>
✅ Helps you scale: add more features without chaos<br>
✅ Easier debugging<br>
✅ Clean, readable, team-friendly<br>
✅ Mirrors modern frameworks (Laravel, Django style)<br>
✅ Production-grade layout</p>

