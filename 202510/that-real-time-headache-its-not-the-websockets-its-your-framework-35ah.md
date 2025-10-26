---
Title: That-Real-Time-Headache-Its-Not-The-WebSockets-Its-Your-Framework
Description: 
Author: member_ece4a271
Date: 2025-10-26T22:00:32.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://github.com/hyperlane-dev/hyperlane" rel="noopener noreferrer">GitHub Home</a></p>

<h2>
  
  
  That Real-Time Headache? It's Not The WebSockets, It's Your Framework 🤯
</h2>

<p>I remember a few years ago, I was leading a team to develop a real-time stock ticker dashboard. 📈 Initially, everyone's enthusiasm was incredibly high. We were all excited to build a "live" application with our own hands. But soon, we found ourselves stuck in the mud. The tech stack we chose performed reasonably well for ordinary REST APIs, but as soon as WebSockets came into the picture, everything became unrecognizable.</p>

<p>Our codebase split into two worlds: the "main application" that handled HTTP requests, and a "separate module" that handled WebSocket connections. Sharing state between these two worlds, like a user's login information, became a nightmare. We had to resort to some very clever (or rather, ugly) methods, like using Redis or a message queue to synchronize data. 🐛 The code became increasingly complex, and bugs multiplied. In the end, although we delivered the product, the entire development process felt like a long and painful tooth extraction. 🦷</p>

<p>This experience taught me a profound lesson: for modern web applications that require real-time interaction, <strong>how a framework handles WebSockets directly determines the development experience and the ultimate success or failure of the project.</strong> Many frameworks claim to "support" WebSockets, but most of them simply "weld" a WebSocket module onto the main framework. This "grafted" solution is often the root of all our headaches. Today, I want to talk about how a well-designed framework elevates WebSockets from a "second-class citizen" to a "first-class citizen" on equal footing with HTTP. 😎</p>

<h3>
  
  
  Common Symptoms of "Grafted" WebSockets
</h3>

<p>Let's first look at the problems that these "grafted" solutions typically cause. Whether in the Java world or the Node.js world, you've likely seen similar design patterns.</p>

<h4>
  
  
  Symptom 1: A Divided World
</h4>

<p>In Java, you might use JAX-RS or Spring MVC to build your REST APIs, but to handle WebSockets, you need to use a completely different API, like <code>javax.websocket</code> and the <code>@ServerEndpoint</code> annotation.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// JAX-RS REST Endpoint</span>
<span class="nd">@Path</span><span class="o">(</span><span class="s">"/api/user"</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">UserResource</span> <span class="o">{</span>
    <span class="nd">@GET</span>
    <span class="kd">public</span> <span class="nc">String</span> <span class="nf">getUser</span><span class="o">()</span> <span class="o">{</span> <span class="k">return</span> <span class="s">"Hello User"</span><span class="o">;</span> <span class="o">}</span>
<span class="o">}</span>

<span class="c1">// WebSocket Endpoint</span>
<span class="nd">@ServerEndpoint</span><span class="o">(</span><span class="s">"/ws/chat"</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">ChatEndpoint</span> <span class="o">{</span>
    <span class="nd">@OnOpen</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">onOpen</span><span class="o">(</span><span class="nc">Session</span> <span class="n">session</span><span class="o">)</span> <span class="o">{</span> <span class="cm">/* ... */</span> <span class="o">}</span>

    <span class="nd">@OnMessage</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">onMessage</span><span class="o">(</span><span class="nc">String</span> <span class="n">message</span><span class="o">,</span> <span class="nc">Session</span> <span class="n">session</span><span class="o">)</span> <span class="o">{</span> <span class="cm">/* ... */</span> <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>See that? <code>UserResource</code> and <code>ChatEndpoint</code> seem to live in two parallel universes. They have their own lifecycles, their own annotations, and their own ways of injecting parameters. Want to get the current user's authentication information in <code>ChatEndpoint</code>? In <code>UserResource</code>, this might just require a <code>@Context SecurityContext</code> annotation, but here, you might have to go to great lengths to access the underlying HTTP Session, and often, the framework won't even let you get it easily. 😫</p>

<p>In Node.js, the situation is similar. You set up your web server with Express, and then you need a library like <code>ws</code> to handle WebSockets.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">express</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">express</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">http</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">http</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">WebSocket</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">ws</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/data</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="dl">'</span><span class="s1">Some data</span><span class="dl">'</span><span class="p">);</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">server</span> <span class="o">=</span> <span class="nx">http</span><span class="p">.</span><span class="nf">createServer</span><span class="p">(</span><span class="nx">app</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">wss</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">WebSocket</span><span class="p">.</span><span class="nc">Server</span><span class="p">({</span> <span class="nx">server</span> <span class="p">});</span>

<span class="nx">wss</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">connection</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">ws</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">ws</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">message</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">message</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">received: %s</span><span class="dl">'</span><span class="p">,</span> <span class="nx">message</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>

<span class="nx">server</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="mi">8080</span><span class="p">);</span>
</code></pre>

</div>



<p>Same problem: <code>app.get</code> and <code>wss.on('connection')</code> are two completely different sets of logic. How do they share middleware? For example, if you want to use an Express authentication middleware to protect your WebSocket connection, can you do it directly? The answer is no. You need to find workarounds, manually invoking the Express middleware when the WebSocket's <code>upgrade</code> request is being processed, which is a very cumbersome process.</p>

<h4>
  
  
  Symptom 2: The Challenge of State Sharing
</h4>

<p>The core of a real-time application is state. You need to know which user corresponds to which WebSocket connection, which channels the user is subscribed to, and so on. In a divided world, sharing this state becomes extremely difficult. Your REST API handles user login and saves session information in the HTTP session storage. Can your WebSocket module access it directly? Usually not. So, you are forced to introduce external dependencies, like Redis, to act as a "state intermediary" between the two worlds. This not only increases the complexity and operational cost of the system but also introduces new potential points of failure. 💔</p>

<h3>
  
  
  The Hyperlane Way: A Natural Unity 🤝
</h3>

<p>Now, let's see how a natively integrated WebSocket framework solves these problems from the ground up. In Hyperlane, a WebSocket handler, just like any other HTTP route handler, is simply a regular <code>async</code> function that receives a <code>Context</code> object. They are natural "siblings," not distant relatives.</p>

<p>The beauty of this design lies in its <strong>consistency</strong>. Once you learn how to write middleware, handle requests, and manipulate the <code>Context</code> for an HTTP route, you automatically know how to do the same for a WebSocket route. The learning curve is almost zero!</p>

<h4>
  
  
  Shared Middleware? A Piece of Cake!
</h4>

<p>Remember the <code>auth_middleware</code> we wrote in the previous article? It passes user information through the <code>Context</code>'s <code>attributes</code>. Now, we can apply it directly to our WebSocket route <strong>without any modifications</strong>!<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="c1">// In the main function</span>
<span class="c1">// ...</span>
<span class="n">server</span><span class="nf">.request_middleware</span><span class="p">(</span><span class="n">auth_middleware</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span> <span class="c1">// Global authentication middleware</span>

<span class="n">server</span><span class="nf">.route</span><span class="p">(</span><span class="s">"/api/secure-data"</span><span class="p">,</span> <span class="n">secure_http_route</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>
<span class="n">server</span><span class="nf">.route</span><span class="p">(</span><span class="s">"/ws/secure-chat"</span><span class="p">,</span> <span class="n">secure_websocket_route</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span> <span class="c1">// ✨ Protected by the same middleware</span>
</code></pre>

</div>



<p>When a WebSocket connection request comes in, it is first an HTTP <code>Upgrade</code> request. Our <code>auth_middleware</code> will run normally, check its token, and if validated, put the <code>User</code> information into the <code>Context</code>. Then, inside <code>secure_websocket_route</code>, we can safely retrieve the user information from the <code>Context</code> and bind this WebSocket connection to that user. The whole process is seamless, without any "glue code." This is just so cool! 😎</p>

<h4>
  
  
  A Unified API: The Magic of <code>send_body</code>
</h4>

<p>Hyperlane also pursues this unity in its API design. Whether you are sending a regular HTTP response body, an SSE event, or a WebSocket message, you use the same method: <code>ctx.send_body().await</code>.</p>

<p>The framework handles all the complexities of the WebSocket protocol for you under the hood (like message framing, masking, etc.). You only need to care about the business data (<code>Vec&lt;u8&gt;</code>) you want to send. This abstraction allows developers to focus on business logic rather than protocol details.</p>

<h4>
  
  
  Broadcasting? Of Course!
</h4>

<p>The documentation even shows us the way to implement a chat room broadcast feature. By using a helper crate like , we can easily distribute messages to all connected clients. The documentation also kindly provides an important technical tip:</p>

<p>This kind of "veteran" advice helps developers avoid common pitfalls. This is exactly what a mature framework should be: it not only gives you powerful tools but also tells you the best practices for using them. 👍</p>

<h3>
  
  
  Stop Letting Your Framework Hold You Back
</h3>

<p>Real-time functionality should no longer be a "special problem" in web development. It is a core component of modern applications. If your framework is still making you handle WebSockets in a completely different, fragmented way, then it may no longer be suitable for this era.</p>

<p>A truly modern framework should seamlessly integrate real-time communication into its core model. It should provide a <strong>consistent API</strong>, a <strong>shareable middleware ecosystem</strong>, and a <strong>unified state management</strong> mechanism. Hyperlane shows us this possibility.</p>

<p>So, the next time you have a headache developing real-time features, please consider that the problem may not be with WebSockets themselves, but with the outdated framework you've chosen that still operates with a "grafted" mindset. It's time for a change! 🚀</p>

<p><a href="https://github.com/hyperlane-dev/hyperlane" rel="noopener noreferrer">GitHub Home</a></p>

