---
Title: Clean MVC Architecture in Node.js Without Express - A Laravel Inspired Approach
Description: 
Author: Al-Amin Sarker
Date: 2025-08-30T21:35:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>A lightweight, framework-free Node.js project demonstrating a clean MVC structure inspired by Laravel. Includes <strong>routing</strong>, <strong>controllers</strong>, <strong>middleware</strong>, <strong>MySQL integration</strong>, <strong>form handling</strong>, <strong>security</strong> and a simple <strong>view Engine</strong>. All built from scratch.</p>

<p>When most developers think of Node.js, they immediately think of Express.js. But what if you could build a clean MVC architecture routing, controllers, models, views, middleware, security without any framework ? That’s exactly what I built in my project. It’s a lightweight MVC boilerplate that teaches you how frameworks work under the hood.</p>

<p><strong>Why This Project ?</strong></p>

<ol>
<li>
<strong>No Frameworks:</strong> Everything is plain Node.js. Learn how MVC logic works internally.</li>
<li>
<strong>MVC Structure:</strong> Inspired by Laravel’s clarity and organization.</li>
<li>
<strong>Secure by Default:</strong> Includes <strong>CSRF</strong>, <strong>XSS protection</strong>, <strong>sessions</strong>, <strong>CORS</strong> and <strong>Rate limiting</strong>.</li>
<li>
<strong>Full Feature Set:</strong> <strong>Routing</strong>, <strong>controllers</strong>, <strong>models</strong>, <strong>validation</strong>, <strong>API support</strong>, <strong>view Engine</strong> and <strong>file uploads</strong>.</li>
</ol>

<p>If you’ve ever wanted to understand what happens behind Express, Laravel or Django, this project is the perfect deep dive.</p>

<p><strong>Routing &amp; Route Service Provider</strong><br>
This project has a custom routing system for <strong>web</strong> and <strong>API</strong> endpoints, complete with global middleware and <strong>/api</strong> prefixes. Routes are defined like Laravel:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// routes/web.js
const Route = require('../system/WebRoute');
const UserController = require('../app/controllers/UserController');

Route.get('/users', UserController.index);
Route.post('/users', UserController.store);

module.exports = Route;
</code></pre>

</div>



<p><strong>Features:</strong></p>

<ul>
<li>Supports <strong>GET</strong>, <strong>POST</strong>, <strong>PUT</strong>, <strong>DELETE</strong>.</li>
<li>Middleware can be attached per route.</li>
</ul>

<p><strong>Middleware</strong><br>
Middleware enhances modularity, running before controllers for tasks like logging, authentication, validation, and CSRF protection. Just like Laravel’s pipeline.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Example usage
Route.post('/users', UserController.store, [AuthMiddleware]);
</code></pre>

</div>



<p><strong>Controllers</strong><br>
Controllers manage the business logic and prepare responses for <strong>web</strong> or <strong>API</strong> routes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// app/controllers/api/UserController.js
const response = require('../../../helpers/response');
const User = require('../../models/User');

class UserController {
  async index(req, res) {
    const users = await User.all();

    return response.json(res, {
        success: true,
        message: 'Success',
        data: users,
     });
  }
}

module.exports = new UserController();
</code></pre>

</div>



<p><strong>Models with Fillable Support</strong><br>
Models extend a base Model class and define <strong>fillable</strong> fields to prevent mass assignment vulnerabilities. Just like Laravel’s fillable fields, models prevent mass assignment vulnerabilities.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// app/models/User.js
const Model = require('../../system/Model');

class User extends Model {
   constructor() {
     super('users', ['name', 'email', 'password']);
   }
}

module.exports = new User();
</code></pre>

</div>



<p>Only <strong>name</strong>, <strong>email</strong> and <strong>password</strong> can be mass-assigned.</p>

<p><strong>Usage in a controller:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>await User.create({
   name: 'Alamin',
   email: 'alamin@gmail.com',
   password: hash('1234')
});
</code></pre>

</div>



<p><strong>View Engine</strong><br>
A minimalist templating engine allows server-side rendering with simple HTML interpolation, similar to <strong>Blade</strong> or <strong>EJS</strong>.<br>
View Template (<em>views/home.html</em>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;html&gt;
  &lt;head&gt;&lt;title&gt;{{ title }}&lt;/title&gt;&lt;/head&gt;
  &lt;body&gt;
    &lt;h1&gt;{{ content }}&lt;/h1&gt;
  &lt;/body&gt;
&lt;/html&gt;
</code></pre>

</div>



<p><strong>Render from Controller:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const html = await view('home', { 
  title: 'Hello Node.js', 
  content: 'Hello World!' 
});
res.writeHead(200, { 'Content-Type': 'text/html' });
res.end(html);
</code></pre>

</div>



<p><strong>Security First</strong><br>
The project includes essential security protections by default:</p>

<ol>
<li>
<strong>CSRF Tokens</strong> for safe form submissions in web routes.</li>
<li>
<strong>XSS Sanitization</strong> using sanitize-html</li>
<li>
<strong>CORS Middleware</strong> for safe cross-origin requests.</li>
<li>
<strong>Rate Limiting</strong> to prevent abuse.</li>
<li>
<strong>Request Loggin</strong> for debugging and monitoring.</li>
</ol>

<p><strong>Form Handling &amp; File Uploads</strong><br>
Supports <em>application/json</em>, <em>x-www-form-urlencoded</em> and <em>multipart/form-data</em> (via formidable)</p>

<p><strong>Quick Start</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git clone git@github.com:alamincse/mvc-app-nodejs.git
cd mvc-app-nodejs
npm install
</code></pre>

</div>



<p><strong>Run the server:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>nodemon server
node server # or
npm run pm2:start # or use pm2
</code></pre>

</div>



<p>And create database tables with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>nodemon database  # Or `node database`
</code></pre>

</div>



<p>Configure your <strong>.env</strong> file for database credentials and ports.</p>

<p><strong>Access the Application</strong><br>
Open your browser and navigate to <strong><a href="http://localhost:3000" rel="noopener noreferrer">http://localhost:3000</a></strong></p>

<p><strong>Why This Architecture Stands Out ?</strong></p>

<ol>
<li>
<strong>Pure JavaScript:</strong> No reliance on Express or other heavy frameworks.</li>
<li>
<strong>Understand MVC deeply:</strong> Learn core patterns and workflows.</li>
<li>
<strong>Security-first:</strong> Shields common attacks with default middleware.</li>
<li>
<strong>Feature-rich:</strong> Includes templating, validation, models, routing, APIs.</li>
<li>
<strong>Ideal for learning:</strong> Perfect for developers keen on diving into MVC fundamentals.</li>
</ol>

<p><strong>Why You Should Try It ?</strong></p>

<ol>
<li>
<strong>Understand frameworks deeply:</strong> You’ll never see Express or Laravel the same way again.</li>
<li>
<strong>Educational:</strong> Great for students and self-learners.</li>
<li>
<strong>Extensible:</strong> Your own middleware, validation or auth system.</li>
<li>
<strong>Lightweight:</strong> Perfect for small projects and experiments.</li>
</ol>

<p><strong>Final Thoughts</strong><br>
Building mvc-app-nodejs demonstrates that frameworks are essentially layers of abstraction. By building an MVC system from scratch, you gain insight into the inner workings of routing, controllers, models, views and middleware.</p>

<p><strong>Read more on the GitHub repository:</strong> <a href="https://github.com/alamincse/mvc-app-nodejs" rel="noopener noreferrer">MVC-APP-Nodejs</a></p>

