---
Title: "Rust and TypeScript"
Description: 
Author: GAUTAM MANAK
Date: 2025-08-30T20:37:22.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Rust and TypeScript: A Powerful Duo for Modern Development
</h2>

<p>Rust and TypeScript, two seemingly disparate languages, are increasingly finding common ground in modern software development. Rust, known for its unparalleled performance and memory safety, excels in system-level programming and performance-critical applications. TypeScript, a superset of JavaScript, brings strong typing and improved tooling to web development, making large-scale JavaScript projects more manageable. This post explores how these two languages can complement each other, creating robust and efficient applications from backend to frontend. We'll dive into practical examples, demonstrating how Rust can power your backend while TypeScript handles the complexities of your frontend. By understanding their strengths and weaknesses, you can leverage the best of both worlds for a superior development experience.</p>

<h3>
  
  
  Why Combine Rust and TypeScript?
</h3>

<p>The combination of Rust and TypeScript offers several compelling advantages. Rust provides a solid foundation for backend services, offering:</p>

<ul>
<li>  <strong>Performance:</strong> Rust's zero-cost abstractions and low-level control allow for highly optimized code, crucial for demanding applications.</li>
<li>  <strong>Memory Safety:</strong> Rust's borrow checker eliminates common memory errors like dangling pointers and data races, leading to more reliable and secure applications.</li>
<li>  <strong>Concurrency:</strong> Rust's ownership system makes concurrent programming safer and easier to reason about.</li>
</ul>

<p>TypeScript, on the other hand, shines in frontend development with features like:</p>

<ul>
<li>  <strong>Strong Typing:</strong> TypeScript's static typing helps catch errors early in the development process, improving code quality and maintainability.</li>
<li>  <strong>Improved Tooling:</strong> TypeScript benefits from excellent IDE support, code completion, and refactoring capabilities.</li>
<li>  <strong>Large Ecosystem:</strong> TypeScript leverages the vast JavaScript ecosystem, providing access to a wide range of libraries and frameworks.</li>
</ul>

<p>By using Rust for performance-sensitive tasks and TypeScript for the user interface, you can build applications that are both fast and maintainable.</p>

<h3>
  
  
  Building a Simple Rust Backend with <code>actix-web</code>
</h3>

<p>Let's create a basic Rust backend using the <code>actix-web</code> framework, a popular choice for building web applications. We'll create a simple API endpoint that returns a JSON response.</p>

<p>First, you'll need to have Rust installed. Then, create a new Rust project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cargo new rust-typescript-example
<span class="nb">cd </span>rust-typescript-example
</code></pre>

</div>



<p>Next, add the <code>actix-web</code> and <code>serde</code> dependencies to your <code>Cargo.toml</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="nn">[dependencies]</span>
<span class="py">actix-web</span> <span class="p">=</span> <span class="s">"4"</span>
<span class="py">serde</span> <span class="o">=</span> <span class="p">{</span> <span class="py">version</span> <span class="p">=</span> <span class="s">"1.0"</span><span class="p">,</span> <span class="py">features</span> <span class="p">=</span> <span class="p">[</span><span class="s">"derive"</span><span class="p">]</span> <span class="p">}</span>
<span class="py">serde_json</span> <span class="p">=</span> <span class="s">"1.0"</span>
</code></pre>

</div>



<p>Now, let's create the main Rust file (<code>src/main.rs</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="nn">actix_web</span><span class="p">::{</span><span class="n">get</span><span class="p">,</span> <span class="n">App</span><span class="p">,</span> <span class="n">HttpResponse</span><span class="p">,</span> <span class="n">HttpServer</span><span class="p">,</span> <span class="n">Responder</span><span class="p">};</span>
<span class="k">use</span> <span class="nn">serde</span><span class="p">::{</span><span class="n">Deserialize</span><span class="p">,</span> <span class="n">Serialize</span><span class="p">};</span>

<span class="nd">#[derive(Serialize,</span> <span class="nd">Deserialize)]</span>
<span class="k">struct</span> <span class="n">Message</span> <span class="p">{</span>
    <span class="n">text</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span>
<span class="p">}</span>

<span class="nd">#[get(</span><span class="s">"/hello"</span><span class="nd">)]</span>
<span class="k">async</span> <span class="k">fn</span> <span class="nf">hello</span><span class="p">()</span> <span class="k">-&gt;</span> <span class="k">impl</span> <span class="n">Responder</span> <span class="p">{</span>
    <span class="k">let</span> <span class="n">message</span> <span class="o">=</span> <span class="n">Message</span> <span class="p">{</span>
        <span class="n">text</span><span class="p">:</span> <span class="s">"Hello from Rust!"</span><span class="nf">.to_string</span><span class="p">(),</span>
    <span class="p">};</span>
    <span class="nn">HttpResponse</span><span class="p">::</span><span class="nf">Ok</span><span class="p">()</span><span class="nf">.json</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>
<span class="p">}</span>

<span class="nd">#[actix_web::main]</span>
<span class="k">async</span> <span class="k">fn</span> <span class="nf">main</span><span class="p">()</span> <span class="k">-&gt;</span> <span class="nn">std</span><span class="p">::</span><span class="nn">io</span><span class="p">::</span><span class="nb">Result</span><span class="o">&lt;</span><span class="p">()</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="nn">HttpServer</span><span class="p">::</span><span class="nf">new</span><span class="p">(||</span> <span class="p">{</span>
        <span class="nn">App</span><span class="p">::</span><span class="nf">new</span><span class="p">()</span>
            <span class="nf">.service</span><span class="p">(</span><span class="n">hello</span><span class="p">)</span>
    <span class="p">})</span>
    <span class="nf">.bind</span><span class="p">(</span><span class="s">"127.0.0.1:8080"</span><span class="p">)</span><span class="o">?</span>
    <span class="nf">.run</span><span class="p">()</span>
    <span class="k">.await</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Explanation:</strong></p>

<ul>
<li>  We import the necessary modules from <code>actix-web</code> and <code>serde</code>.</li>
<li>  We define a <code>Message</code> struct with a <code>text</code> field, using <code>serde</code> to enable serialization and deserialization to JSON.</li>
<li>  The <code>hello</code> function is an asynchronous handler that returns a JSON response containing a greeting.</li>
<li>  The <code>#[get("/hello")]</code> attribute maps the <code>/hello</code> route to the <code>hello</code> function.</li>
<li>  The <code>main</code> function sets up the <code>actix-web</code> server, registers the <code>hello</code> service, and starts the server on <code>127.0.0.1:8080</code>.</li>
</ul>

<p>To run the backend, execute:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cargo run
</code></pre>

</div>



<p>You can now access the API endpoint at <code>http://127.0.0.1:8080/hello</code> and see the JSON response.</p>

<h3>
  
  
  Building a TypeScript Frontend with <code>fetch</code>
</h3>

<p>Now, let's create a simple TypeScript frontend that consumes the Rust backend API.  We'll use the <code>fetch</code> API to make the request.</p>

<p>First, you'll need Node.js and npm installed. Create a new TypeScript project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>typescript-frontend
<span class="nb">cd </span>typescript-frontend
npm init <span class="nt">-y</span>
npm <span class="nb">install </span>typescript <span class="nt">--save-dev</span>
npx tsc <span class="nt">--init</span>
</code></pre>

</div>



<p>Next, install the <code>node-fetch</code> package to use <code>fetch</code> in a Node.js environment. This is useful for running the code locally and testing before deploying to a browser environment.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>node-fetch <span class="nt">--save</span>
npm <span class="nb">install</span> @types/node-fetch <span class="nt">--save-dev</span>
</code></pre>

</div>



<p>Now, create a TypeScript file (<code>src/index.ts</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">fetch</span><span class="p">,</span> <span class="p">{</span> <span class="nx">Response</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">node-fetch</span><span class="dl">'</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">Message</span> <span class="p">{</span>
  <span class="nl">text</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">fetchData</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">response</span><span class="p">:</span> <span class="nx">Response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">http://127.0.0.1:8080/hello</span><span class="dl">'</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`HTTP error! Status: </span><span class="p">${</span><span class="nx">response</span><span class="p">.</span><span class="nx">status</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="kd">const</span> <span class="na">data</span><span class="p">:</span> <span class="nx">Message</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">()</span> <span class="k">as</span> <span class="nx">Message</span><span class="p">;</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Data from Rust backend:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">data</span><span class="p">.</span><span class="nx">text</span><span class="p">);</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error fetching data:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nf">fetchData</span><span class="p">();</span>
</code></pre>

</div>



<p><strong>Explanation:</strong></p>

<ul>
<li>  We import the <code>fetch</code> function from <code>node-fetch</code>.</li>
<li>  We define a <code>Message</code> interface that matches the structure of the JSON response from the Rust backend.</li>
<li>  The <code>fetchData</code> function makes a <code>GET</code> request to the <code>/hello</code> endpoint of the Rust backend.</li>
<li>  We check if the response is successful (status code 200-299).</li>
<li>  We parse the response body as JSON and cast it to the <code>Message</code> interface.</li>
<li>  We log the <code>text</code> field of the message to the console.</li>
<li>  We handle potential errors during the fetch operation.</li>
</ul>

<p>To compile and run the TypeScript code, execute:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx tsc
node dist/index.js
</code></pre>

</div>



<p>You should see the message "Data from Rust backend: Hello from Rust!" printed to the console.</p>

<h3>
  
  
  Data Validation and Type Safety
</h3>

<p>One of the benefits of using TypeScript on the frontend is type safety. You can define interfaces that match the structure of the data returned by the Rust backend, ensuring that your frontend code handles the data correctly.  While the example above directly casts the result of <code>response.json()</code> to the <code>Message</code> interface, for production applications it's often beneficial to use a validation library like <code>zod</code> or <code>io-ts</code> to ensure the data received from the backend conforms to the expected schema.  This can help prevent runtime errors caused by unexpected data formats.</p>

<p>For example, using <code>zod</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">fetch</span><span class="p">,</span> <span class="p">{</span> <span class="nx">Response</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">node-fetch</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">z</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zod</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">MessageSchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
  <span class="na">text</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span>
<span class="p">});</span>

<span class="kd">type</span> <span class="nx">Message</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nx">infer</span><span class="o">&lt;</span><span class="k">typeof</span> <span class="nx">MessageSchema</span><span class="o">&gt;</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">fetchData</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="na">response</span><span class="p">:</span> <span class="nx">Response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">http://127.0.0.1:8080/hello</span><span class="dl">'</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`HTTP error! Status: </span><span class="p">${</span><span class="nx">response</span><span class="p">.</span><span class="nx">status</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="kd">const</span> <span class="nx">rawData</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
    <span class="kd">const</span> <span class="na">data</span><span class="p">:</span> <span class="nx">Message</span> <span class="o">=</span> <span class="nx">MessageSchema</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">rawData</span><span class="p">);</span> <span class="c1">// Validate the data</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Data from Rust backend:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">data</span><span class="p">.</span><span class="nx">text</span><span class="p">);</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error fetching data:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nf">fetchData</span><span class="p">();</span>
</code></pre>

</div>



<p>In this example, <code>zod</code> is used to define a schema for the <code>Message</code> type.  The <code>MessageSchema.parse(rawData)</code> line validates that the data received from the backend conforms to the defined schema. If the data does not conform, <code>zod</code> will throw an error, preventing the application from proceeding with invalid data.  This approach improves the robustness and reliability of your frontend application.</p>

<h3>
  
  
  Building More Complex Systems
</h3>

<p>This simple example demonstrates the basic principles of combining Rust and TypeScript. In more complex systems, you might use Rust to implement computationally intensive tasks, such as image processing, data analysis, or game engine logic. These tasks can be exposed as APIs that are consumed by a TypeScript frontend.  You could also use a framework like Tauri to build cross-platform desktop applications using Rust for the backend and TypeScript for the frontend. The possibilities are vast, and the combination of Rust and TypeScript provides a powerful foundation for building modern, high-performance applications.</p>

<h3>
  
  
  Conclusion
</h3>

<p>Rust and TypeScript offer a powerful combination for modern software development. Rust's performance and safety make it ideal for backend services, while TypeScript's strong typing and tooling improve the maintainability of frontend code. By leveraging the strengths of both languages, you can build applications that are both fast and reliable. As a next step, explore frameworks like Tauri, learn more about data validation libraries, and experiment with building more complex applications that leverage the full potential of Rust and TypeScript.</p>

