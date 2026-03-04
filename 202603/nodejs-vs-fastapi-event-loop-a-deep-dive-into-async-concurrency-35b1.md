---
Title: Node.js vs FastAPI Event Loop: A Deep Dive into Async Concurrency
Description: 
Author: Dip Chakraborty
Date: 2026-03-04T21:34:50.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you build APIs today, two stacks repeatedly appear in production systems:</p>

<ul>
<li>Node.js</li>
<li>FastAPI</li>
</ul>

<p>Both are widely known for handling high-concurrency workloads, and both rely heavily on event-driven architectures.</p>

<p>At first glance, the two appear almost identical:</p>

<ul>
<li>Both rely on non-blocking I/O</li>
<li>Both encourage asynchronous programming</li>
<li>Both can handle thousands of concurrent connections</li>
</ul>

<p>Because of these similarities, many developers assume they behave in roughly the same way internally.</p>

<p>But the reality is more nuanced!</p>

<p>While the high-level philosophy is similar, the execution model, scheduling behavior, and CPU utilization strategy differ significantly.</p>

<p>Understanding these differences becomes extremely important when designing:</p>

<ul>
<li>large-scale APIs</li>
<li>real-time systems</li>
<li>microservices</li>
<li>AI inference backends</li>
<li>streaming infrastructures</li>
</ul>

<h2>
  
  
  Architectures
</h2>

<p>Node.js executes JavaScript on a single thread.</p>

<p>This design decision was intentional. JavaScript historically ran inside browsers where concurrency was handled through event-driven callbacks rather than threads.</p>

<p>Node adopted the same model.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxszj1snamoczo2gxom9e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxszj1snamoczo2gxom9e.png" alt="Simplest Architecture of Nodejs Event Loop" width="800" height="179"></a></p>

<p>Important characteristics:</p>

<ul>
<li>All JavaScript code runs on one thread</li>
<li>I/O operations are delegated to libuv</li>
<li>When I/O completes, callbacks are pushed back into the event loop</li>
<li>This design works extremely well for I/O-heavy workloads, which most web servers are.</li>
<li>Instead of spawning thousands of threads, Node maintains a single event loop and multiplexes many connections.</li>
</ul>

<p>But there is a trade-off.</p>

<p>If CPU-heavy work blocks the JavaScript thread, the entire server stalls.</p>

<p>Simple Example - </p>

<p><code><br>
while(true) {}<br>
</code></p>

<p>The Node.js event loop is implemented through libuv, a high-performance C library responsible for:</p>

<ul>
<li>asynchronous I/O</li>
<li>networking</li>
<li>file system operations</li>
<li>timers</li>
<li>thread pool scheduling</li>
</ul>

<p>The event loop itself runs through several distinct phases.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg6lcbm2qia3b59lx0u1h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg6lcbm2qia3b59lx0u1h.png" alt="Nodejs Event Loop Phases" width="800" height="217"></a></p>

<p>Timers Phase: Handles callbacks scheduled via:</p>

<p><code>setTimeout()<br>
   setInterval()</code></p>

<p>Pending Callbacks: Processes certain system-level callbacks such as TCP errors.</p>

<p>Poll Phase: This is the heart of the event loop. In this phase, Node:</p>

<ul>
<li>waits for I/O events</li>
<li>retrieves completed operations from the OS</li>
<li>executes the associated callbacks</li>
</ul>

<p>Check Phase: Executes callbacks</p>

<p>Close Callbacks: Handles cleanup events</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffbqqnuliszwo577a581i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffbqqnuliszwo577a581i.png" alt="nodejs request lifecycle" width="800" height="1200"></a></p>

<p>Lets talk about FastAPI. FastAPI itself is just a framework.</p>

<p>The real runtime comes from ASGI servers, typically:</p>

<ul>
<li>Uvicorn</li>
<li>Hypercorn</li>
<li>Gunicorn + Uvicorn workers</li>
</ul>

<p>Each worker is:</p>

<ul>
<li>a separate OS process</li>
<li>running its own Python interpreter</li>
<li>with its own event loop</li>
</ul>

<p>Concurrency therefore comes from two layers:</p>

<ul>
<li>Async coroutines inside the worker</li>
<li>Multiple worker processes across CPU cores</li>
<li>This means FastAPI can utilize multiple CPU cores naturally, something Node requires cluster mode to achieve.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbs5wwr75tjg4pkhwecw0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbs5wwr75tjg4pkhwecw0.png" alt="Fast API Architecture" width="800" height="533"></a></p>

<p>FastAPI relies on Python's asyncio framework, which implements asynchronous execution using coroutines rather than callbacks.</p>

<p>A typical endpoint might look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@app.get("/users")
async def get_users():
    result = await db.fetch_all()
    return result
</code></pre>

</div>



<p>Python creates a coroutine object, which behaves like a pausable computation.</p>

<p>The event loop schedules and resumes these coroutines as needed. </p>

<p>Instead of registering callbacks, Python pauses the coroutine and later resumes it.</p>

<p>This produces a more linear and readable programming model.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Felupfcvb3jutxxpltmnf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Felupfcvb3jutxxpltmnf.png" alt="FastAPI Request LifeCycle" width="800" height="436"></a></p>

<p><strong>Callback vs Coroutine Model</strong></p>

<p>This is where the programming model diverges significantly.</p>

<h2>
  
  
  Node.js
</h2>

<p>Early Node applications relied heavily on callbacks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>db.query(sql, (err, result) =&gt; {
   console.log(result)
})

</code></pre>

</div>



<p>This led to the infamous callback hell problem.</p>

<p>Later improvements introduced:</p>

<ul>
<li>Promises</li>
<li>async / await
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const result = await db.query(sql)
</code></pre>

</div>



<p>However, internally Node still schedules operations via callback queues.</p>

<p>The async/await syntax is largely syntactic sugar over Promises and callbacks.</p>

<p>In the other side, Python’s async system is built around coroutines and cooperative scheduling.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>result = await db.query()
</code></pre>

</div>



<p>Coroutines can pause and resume naturally, complex asynchronous flows often become much easier to reason about.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foz6b1xltwuib2rxuy98t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foz6b1xltwuib2rxuy98t.png" alt="Coroutines" width="800" height="436"></a></p>

<h2>
  
  
  Threads
</h2>

<p>Another important architectural distinction is thread utilization.</p>

<p>Node consists of: <br>
1 main JavaScript thread + libuv worker thread pool</p>

<p>The worker pool handles operations such as:</p>

<ul>
<li>file system access</li>
<li>DNS lookups</li>
<li>compression</li>
<li>cryptography</li>
</ul>

<p>Default thread pool size: 4 threads</p>

<p>This can be increased using: <code>UV_THREADPOOL_SIZE</code></p>

<p>However, JavaScript execution itself remains single-threaded.</p>

<p>Even with a larger thread pool, heavy CPU tasks executed in JavaScript can still block the event loop.</p>

<p>FastAPI deployments typically rely on multiple worker processes: <br>
Gunicorn + 4 Uvicorn workers</p>

<p>Each worker contains:</p>

<ul>
<li>1 OS process</li>
<li>1 Python interpreter</li>
<li>1 event loop</li>
</ul>

<p>The result:<br>
<code>Worker 1 → CPU core 1<br>
Worker 2 → CPU core 2<br>
Worker 3 → CPU core 3<br>
Worker 4 → CPU core 4</code></p>

<p>This enables true parallelism across CPU cores, something Python normally struggles with because of the Global Interpreter Lock (GIL).</p>

<p>But since each worker is a separate process, the GIL becomes irrelevant.</p>

<p>Now we can summarize the concurrency strategy of each runtime.</p>

<p>Node.js Concurrency model:</p>

<p>Single-threaded event loop + Non-blocking I/O</p>

<p>This works exceptionally well for:</p>

<ul>
<li>APIs</li>
<li>streaming services</li>
<li>real-time applications</li>
<li>WebSocket servers</li>
</ul>

<p>Node excels at:</p>

<ul>
<li>high I/O throughput</li>
<li>real-time systems</li>
<li>streaming pipelines</li>
<li>WebSocket infrastructure</li>
<li>Typical use cases include:</li>
<li>chat servers</li>
<li>API gateways</li>
<li>live dashboards</li>
<li>collaborative applications</li>
</ul>

<p>Node also benefits from a massive ecosystem and mature tooling.</p>

<p>However, CPU-heavy tasks can block the event loop.</p>

<p>Examples include:</p>

<ul>
<li>large JSON parsing</li>
<li>image processing</li>
<li>encryption</li>
<li>machine learning inference</li>
</ul>

<p>These tasks often require:</p>

<ul>
<li>worker threads</li>
<li>microservices</li>
<li>background job systems</li>
</ul>

<p>FastAPI concurrency combines:</p>

<p>async coroutines + multiple worker processes</p>

<p>This creates a system where:</p>

<p>Worker 1 → blocked by CPU task<br>
Worker 2 → still serving requests<br>
Worker 3 → still serving requests<br>
Worker 4 → still serving requests</p>

<p>The system remains responsive because workload is distributed across processes.</p>

<p>Major advantages include:</p>

<ul>
<li>seamless integration with machine learning frameworks</li>
<li>strong type hints</li>
<li>automatic request validation via Pydantic</li>
<li>excellent developer ergonomics</li>
</ul>

<p>Performance is surprisingly strong.</p>

<p>Btw, One common misunderstanding is that FastAPI automatically runs everything asynchronously. That is not true.</p>

<p>Consider this endpoint:</p>

<p><code>def endpoint(): ...</code></p>

<p>This is not asynchronous. FastAPI will run it inside a thread pool executor.</p>

<p>In this case:</p>

<p>Blocking function -&gt; Executed in thread pool -&gt; Event loop remains free</p>

<p>To use the event loop directly, endpoints must be written as:</p>

<p><code>async def endpoint():</code></p>

<p>Understanding the full stack layers also helps.</p>

<p>Nodejs: </p>

<p><code>V8 Engine -&gt; Node.js Runtime -&gt; libuv -&gt; Operating System</code></p>

<p>V8 executes JavaScript. libuv handles async I/O</p>

<p>FastAPI: </p>

<p><code>Python Interpreter -&gt; FastAPI Framework -&gt; ASGI -&gt; Uvicorn Server -&gt; asyncio / uvloop -&gt; Operating System</code></p>

<p>A interesting detail: Many FastAPI deployments use uvloop, an event loop implementation written in C.</p>

<p>And fun thing is, uvloop is built on top of libuv — the same library used by Node.js.</p>

<p>So in many modern deployments:</p>

<p>FastAPI + uvloop = Python + libuv</p>

<p>Different language. Same underlying asynchronous engine.</p>

<p>So, Both Node.js and FastAPI follow the same fundamental philosophy:</p>

<p>event-driven, non-blocking I/O.</p>

<p>But they scale concurrency in different ways.</p>

<p>Node.js -</p>

<ul>
<li>single JavaScript execution thread</li>
<li>libuv thread pool for I/O</li>
<li>parallelism requires cluster mode or worker threads</li>
</ul>

<p>FastAPI</p>

<ul>
<li>asyncio coroutines</li>
<li>multiple worker processes</li>
<li>natural multi-core scalability</li>
</ul>

<p>In practice:</p>

<p>Use Node.js when building real-time systems and working within the JavaScript ecosystem.</p>

<p>Use FastAPI when building services around Python, data science, machine learning, or AI workloads.</p>

<p>What matters most is understanding how their concurrency models actually behave under load because architecture decisions made early tend to shape how systems scale later.</p>

<p>I’d love to hear your thoughts, which runtime do you prefer for high-concurrency APIs and why?</p>

<p>If you enjoyed this article, you can read more of my blogs here:<br>
<a href="https://writings.dipchakraborty.com" rel="noopener noreferrer">https://writings.dipchakraborty.com</a></p>

