---
Title: Building an Organized API in Golang Using Fiber
Description: 
Author: Alexandre Fernandes dos Santos
Date: 2025-10-27T21:33:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>After spending some time exploring Go and struggling a bit with its package system (here I must admit it's actually a great system when used properly), I found myself thinking a lot about the best way to organize the study API I was building.</p>

<p>I gathered many examples, including from the Fiber repository, and arrived at a structure that I find quite readable and easy to extend with new features.</p>

<blockquote>
<p><strong>Disclaimer:</strong> I'm not saying this is the best way to organize a Go API, but it worked for my needs and I believe it can be useful in many cases where a generic API is required.</p>
</blockquote>

<h2>
  
  
  What Does This API Do?
</h2>

<p><a href="https://github.com/XandeCoding/codigos-de-artigos/tree/main/golang/api_simples" rel="noopener noreferrer"><strong>Repository Link</strong></a></p>

<p>This API was created to store information about books and has only three endpoints:</p>

<ul>
<li>
<strong>GET:</strong> Returns information about a book</li>
<li>
<strong>PUT:</strong> Adds or updates information about a book</li>
<li>
<strong>DELETE:</strong> Deletes book information</li>
</ul>

<p>I implemented just the basics, using the Fiber framework which has an approach similar to Express.js that I really liked, combined with Go's advantages like lower memory allocation and incredible speed. The data is stored in Redis, which can be initialized using a docker-compose file.</p>

<h2>
  
  
  Structure
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>| api_simples
├── docker-compose.yml
├── docs
│   └── estrutura.png
├── go.mod
├── go.sum
├── main.go
├── pkg
│   ├── configurations
│   │   └── database.go
│   ├── entities
│   │   └── book.go
│   ├── handlers
│   │   └── book_handler.go
│   ├── repositories
│   │   ├── book_repository.go
│   │   └── commons.go
│   └── routes
│       ├── book_router.go
│       └── routes.go
└── README.md
</code></pre>

</div>



<p>I believe it's more valuable to explain the reasoning behind this organization rather than just listing what each folder contains. I won't follow the exact order above since I think it's clearer to explain in a different sequence:</p>

<h3>
  
  
  go.mod
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="n">module</span> <span class="n">github</span><span class="o">.</span><span class="n">com</span><span class="o">/</span><span class="n">XandeCoding</span><span class="o">/</span><span class="n">codigos</span><span class="o">-</span><span class="n">de</span><span class="o">-</span><span class="n">artigos</span><span class="o">/</span><span class="n">golang</span><span class="o">/</span><span class="n">api_simples</span>

<span class="k">go</span> <span class="m">1.19</span>

<span class="n">require</span> <span class="p">(</span>
    <span class="n">github</span><span class="o">.</span><span class="n">com</span><span class="o">/</span><span class="k">go</span><span class="o">-</span><span class="n">redis</span><span class="o">/</span><span class="n">redis</span><span class="o">/</span><span class="n">v9</span> <span class="n">v9</span><span class="m">.0.0</span><span class="o">-</span><span class="n">beta</span><span class="m">.2</span>
    <span class="n">github</span><span class="o">.</span><span class="n">com</span><span class="o">/</span><span class="n">gofiber</span><span class="o">/</span><span class="n">fiber</span><span class="o">/</span><span class="n">v2</span> <span class="n">v2</span><span class="m">.36.0</span>
</code></pre>

</div>



<p>This file helps resolve various Go workspace issues, allowing me to create a Go repository anywhere without problems accessing external or local packages.</p>

<p>To create it, I ran the go mod init command with the GitHub project path as argument (github.com/XandeCoding/codigos-de-artigos/golang/api_simples). While it's not strictly necessary to use the full GitHub path (the project name api_simples would work), I chose to include it since this is a public project. This makes it easier to reference specific files from the project.</p>

<h3>
  
  
  pkg and main.go
</h3>

<p>The pkg folder contains the main API code where all features are implemented. The main.go file is only used to initialize the application and contains no implementation logic, serving solely to start the API.</p>

<p><strong>main.go</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"github.com/XandeCoding/codigos-de-artigos/golang/api_simples/pkg/routes"</span>
    <span class="s">"github.com/gofiber/fiber/v2"</span>
<span class="p">)</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">app</span> <span class="o">:=</span> <span class="n">fiber</span><span class="o">.</span><span class="n">New</span><span class="p">()</span>
    <span class="n">routes</span><span class="o">.</span><span class="n">AddRoutes</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>

    <span class="n">app</span><span class="o">.</span><span class="n">Listen</span><span class="p">(</span><span class="s">":3000"</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  pkg/configurations
</h3>

<p>This contains configuration files. In this case, <code>database.go</code> configures database access. If we had other application or tool configurations used by one or more parts of the application, they would also go here - such as custom Fiber configurations or environment variables.</p>

<p>Example Redis connection configuration in <strong>database.go</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="k">package</span> <span class="n">configurations</span>

<span class="k">import</span> <span class="s">"github.com/go-redis/redis/v9"</span>

<span class="k">func</span> <span class="n">CreateClient</span><span class="p">()</span> <span class="o">*</span><span class="n">redis</span><span class="o">.</span><span class="n">Client</span> <span class="p">{</span>
    <span class="n">redisDatabase</span> <span class="o">:=</span> <span class="n">redis</span><span class="o">.</span><span class="n">NewClient</span><span class="p">(</span><span class="o">&amp;</span><span class="n">redis</span><span class="o">.</span><span class="n">Options</span><span class="p">{</span>
        <span class="n">Addr</span><span class="o">:</span> <span class="s">"localhost:6379"</span><span class="p">,</span>
        <span class="n">Password</span><span class="o">:</span> <span class="s">""</span><span class="p">,</span>
        <span class="n">DB</span><span class="o">:</span> <span class="m">0</span><span class="p">,</span>
    <span class="p">})</span>

    <span class="k">return</span> <span class="n">redisDatabase</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  pkg/entities
</h3>

<p>Entities can be used in various places, particularly in this case where I use them both for receiving data in endpoints and for database operations. Placing them in a common location is quite useful, though in feature-separated package structures, this approach might not be as ideal.<br>
pkg/repositories</p>

<p>This package contains functions that work directly with the Redis database. They receive the book entity and have functions that insert, update, and delete it from the database. If there were another entity to handle, such as library, it would be in a separate file containing only functions related to that data.</p>

<p>Fragment from <strong>book_repository.go</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="k">type</span> <span class="n">Repository</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">database</span> <span class="o">*</span><span class="n">redis</span><span class="o">.</span><span class="n">Client</span>
<span class="p">}</span>

<span class="o">...</span>

<span class="k">func</span> <span class="p">(</span><span class="n">rdb</span> <span class="n">Repository</span><span class="p">)</span> <span class="n">GetBook</span><span class="p">(</span><span class="n">name</span> <span class="kt">string</span><span class="p">)</span> <span class="kt">string</span> <span class="p">{</span>
    <span class="n">ctx</span><span class="p">,</span> <span class="n">cancel</span> <span class="o">:=</span> <span class="n">getContext</span><span class="p">()</span>
    <span class="k">defer</span> <span class="n">cancel</span><span class="p">()</span>
    <span class="n">book</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">rdb</span><span class="o">.</span><span class="n">database</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">name</span><span class="p">)</span><span class="o">.</span><span class="n">Result</span><span class="p">()</span>

    <span class="k">return</span> <span class="n">book</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  pkg/routes
</h3>

<p>Following the example of other parts of the application, I separated routes into different files. Even though we have a routes.go file that initializes these routes, it's helpful to keep routes for specific resources separate for better readability and understanding for others who might maintain the code.</p>

<p>Route initialization in <strong>routes.go</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="k">func</span> <span class="n">AddRoutes</span><span class="p">(</span><span class="n">app</span> <span class="o">*</span><span class="n">fiber</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="o">*</span><span class="n">fiber</span><span class="o">.</span><span class="n">App</span> <span class="p">{</span>
    <span class="n">bookRouter</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">app</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In book_router.go, I only specify the routes, methods, and handler functions located in another part of the application. Another important aspect is that this structure allows us to create instances that can be reused across all endpoints for this specific resource - in this case, a Redis database connection instance.</p>

<p>Fragment from <strong>book_router.go</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="k">func</span> <span class="n">bookRouter</span><span class="p">(</span><span class="n">app</span> <span class="o">*</span><span class="n">fiber</span><span class="o">.</span><span class="n">App</span><span class="p">)</span> <span class="o">*</span><span class="n">fiber</span><span class="o">.</span><span class="n">App</span> <span class="p">{</span>
    <span class="n">bookRepository</span> <span class="o">:=</span> <span class="n">repositories</span><span class="o">.</span><span class="n">NewRepository</span><span class="p">()</span>

    <span class="n">app</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="s">"/book/:name"</span><span class="p">,</span> <span class="n">handlers</span><span class="o">.</span><span class="n">GetBookHandler</span><span class="p">(</span><span class="n">bookRepository</span><span class="p">))</span>
    <span class="n">app</span><span class="o">.</span><span class="n">Put</span><span class="p">(</span><span class="s">"/book"</span><span class="p">,</span> <span class="n">handlers</span><span class="o">.</span><span class="n">SetBookHandler</span><span class="p">(</span><span class="n">bookRepository</span><span class="p">))</span>
    <span class="n">app</span><span class="o">.</span><span class="n">Delete</span><span class="p">(</span><span class="s">"/book/:name"</span><span class="p">,</span> <span class="n">handlers</span><span class="o">.</span><span class="n">DeleteBookHandler</span><span class="p">(</span><span class="n">bookRepository</span><span class="p">))</span>

    <span class="k">return</span> <span class="n">app</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  pkg/handlers
</h3>

<p>In handlers, I place the functions called by the endpoints. For example, the PUT /book endpoint calls the SetBookHandler function in book_handler.go, which returns the function to be executed when this resource is accessed.</p>

<p>Code for the <strong>SetBookHandler</strong> function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>
<span class="k">func</span> <span class="n">SetBookHandler</span><span class="p">(</span><span class="n">bookRepository</span> <span class="o">*</span><span class="n">repositories</span><span class="o">.</span><span class="n">Repository</span><span class="p">)</span> <span class="n">fiber</span><span class="o">.</span><span class="n">Handler</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">func</span><span class="p">(</span><span class="n">c</span> <span class="o">*</span><span class="n">fiber</span><span class="o">.</span><span class="n">Ctx</span><span class="p">)</span> <span class="kt">error</span> <span class="p">{</span>
        <span class="n">book</span> <span class="o">:=</span> <span class="nb">new</span><span class="p">(</span><span class="n">entities</span><span class="o">.</span><span class="n">Book</span><span class="p">)</span>
        <span class="n">err</span> <span class="o">:=</span> <span class="n">c</span><span class="o">.</span><span class="n">BodyParser</span><span class="p">(</span><span class="n">book</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="n">message</span> <span class="o">:=</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="kt">string</span><span class="p">{</span><span class="s">"message"</span><span class="o">:</span> <span class="s">"Invalid entry data"</span><span class="p">}</span>
            <span class="k">return</span> <span class="n">c</span><span class="o">.</span><span class="n">Status</span><span class="p">(</span><span class="m">400</span><span class="p">)</span><span class="o">.</span><span class="n">JSON</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>
        <span class="p">}</span>

        <span class="n">book_name_normalized</span> <span class="o">:=</span> <span class="n">strings</span><span class="o">.</span><span class="n">ReplaceAll</span><span class="p">(</span>
            <span class="n">strings</span><span class="o">.</span><span class="n">ToLower</span><span class="p">(</span><span class="n">book</span><span class="o">.</span><span class="n">Name</span><span class="p">),</span> <span class="s">" "</span><span class="p">,</span> <span class="s">"_"</span><span class="p">,</span>
        <span class="p">)</span>
        <span class="n">key</span> <span class="o">:=</span> <span class="s">"book:"</span> <span class="o">+</span> <span class="n">book_name_normalized</span>
        <span class="n">bookRepository</span><span class="o">.</span><span class="n">SetBook</span><span class="p">(</span><span class="n">key</span><span class="p">,</span> <span class="n">book</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">c</span><span class="o">.</span><span class="n">Send</span><span class="p">(</span><span class="n">c</span><span class="o">.</span><span class="n">Body</span><span class="p">())</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  That's It!
</h2>

<p>I hope this helps with some of the initial challenges when building an API, especially in a language we're not very familiar with. This initial structure worked well for me, but any comments or feedback are always welcome for continuous improvement. Until next time! 👋</p>

