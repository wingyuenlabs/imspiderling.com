---
Title: You-Might-Not-Need-WebSockets-The-Simple-Power-of-Server-Sent-Events
Description: 
Author: member_f8c307c5
Date: 2025-10-07T21:35:41.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://github.com/hyperlane-dev/hyperlane" rel="noopener noreferrer">GitHub Home</a></p>

<h2>
  
  
  You Might Not Need WebSockets: The Simple Power of Server-Sent Events 🤫
</h2>

<p>In our toolbox, there are always a few "star" tools. 🛠️ In the realm of real-time web communication, WebSocket is undoubtedly the brightest star. It's powerful, supports bidirectional communication, and has become the "default answer" for almost all real-time needs. So, when a product manager comes to you and says, "Hey, we need a dashboard that updates in real-time!" the first thing that pops into many programmers' minds is: "Okay, let's use WebSockets!"</p>

<p>But, wait a minute. ✋ As an old-timer who has been navigating this world for decades, I want to ask: do we always need a "Swiss Army knife" to peel an apple? 🍎</p>

<p>I've seen too many scenarios where a simple feature that only requires unidirectional data push from the server to the client—like site notifications, stock price updates, or live sports scores—ends up being implemented with a full-duplex WebSocket. This is not just using a sledgehammer to crack a nut; it's digging a pit of complexity for yourself. Today, I want to make a case for another underestimated hero: <strong>Server-Sent Events (SSE)</strong>. It's simple, efficient, and in many scenarios, a more elegant and appropriate solution than WebSockets.</p>

<h3>
  
  
  Two Common Misconceptions About "Real-Time"
</h3>

<p>Before we embrace SSE, let's look at two common pitfalls developers fall into when trying to achieve "server push."</p>

<h4>
  
  
  Misconception 1: The "Brute Force" Aesthetics of Client-Side Polling
</h4>

<p>This is the most primitive and intuitive method. The client sets up a timer and sends an AJAX request to the server every few seconds, asking, "Hey buddy, got any new data?"<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// The polling nightmare 😫</span>
<span class="nf">setInterval</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/updates</span><span class="dl">'</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
    <span class="c1">// Update the UI with the new data</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">New data:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">data</span><span class="p">);</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error fetching updates:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">},</span> <span class="mi">5000</span><span class="p">);</span> <span class="c1">// Ask every 5 seconds</span>
</code></pre>

</div>



<p>The problems with this approach are all too obvious:</p>

<ol>
<li> <strong>High Latency</strong>: The user might have to wait up to 5 seconds to see an update. Want to reduce latency? Shorten the interval? That will put even more pressure on the server.</li>
<li> <strong>Wasted Resources</strong>: The vast majority of requests will likely return empty-handed because data isn't updated every moment. Every request, with or without new data, includes the full overhead of HTTP headers. It's like calling someone every five minutes to ask "is dinner ready yet?" Annoying and inefficient. 📞</li>
<li> <strong>Poor Scalability</strong>: Imagine thousands of clients tirelessly "harassing" your server like this. Your server will burn a massive amount of CPU and network resources on these repetitive, empty handshakes and queries.</li>
</ol>

<h4>
  
  
  Misconception 2: The "Overkill" of WebSockets
</h4>

<p>To solve the problems of polling, many developers naturally turn to WebSockets. It establishes a persistent, bidirectional connection, allowing the server to actively push data at any time. Perfect! 🎉</p>

<p>But for a scenario that only requires one-way push, the "bidirectional" capability of WebSockets becomes a burden. You're introducing a relatively complex protocol, and you need to handle its connection lifecycle, heartbeats, and reconnection logic. It's like buying a whole cow just to get a glass of milk. 🐄</p>

<p>More importantly, you might inadvertently split your application logic again (as we discussed in the previous article). You create a separate set of handling logic for WebSockets when it could have been perfectly integrated with your existing HTTP logic.</p>

<h3>
  
  
  The Elegance of SSE: Returning to the Heart of HTTP ✨
</h3>

<p>Now, let's give a grand welcome to today's protagonist: SSE. SSE is not some brand-new black magic; it's part of the HTTP protocol itself, a W3C standard. Its core idea is simple to the extreme: <strong>the client initiates a GET request, the server holds on to this connection, and then continuously "streams" data to the client through this connection.</strong></p>

<p>It's like a phone call that never hangs up. The client just listens, and the server does the talking. It perfectly solves the problem of one-way data push, and it runs entirely on top of the standard HTTP protocol.</p>

<p>Implementing an SSE endpoint in Hyperlane is a piece of cake. Look at this code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="k">crate</span><span class="p">::{</span><span class="nn">tokio</span><span class="p">::</span><span class="nn">time</span><span class="p">::</span><span class="n">sleep</span><span class="p">,</span> <span class="o">*</span><span class="p">};</span>
<span class="k">use</span> <span class="nn">std</span><span class="p">::</span><span class="nn">time</span><span class="p">::</span><span class="n">Duration</span><span class="p">;</span>

<span class="k">pub</span> <span class="k">async</span> <span class="k">fn</span> <span class="nf">sse_route</span><span class="p">(</span><span class="n">ctx</span><span class="p">:</span> <span class="n">Context</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// 1. Set the correct Content-Type to tell the browser this is an event stream</span>
    <span class="k">let</span> <span class="n">_</span> <span class="o">=</span> <span class="n">ctx</span>
        <span class="nf">.set_response_header</span><span class="p">(</span><span class="n">CONTENT_TYPE</span><span class="p">,</span> <span class="n">TEXT_EVENT_STREAM</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.set_response_status_code</span><span class="p">(</span><span class="mi">200</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.send</span><span class="p">()</span> <span class="c1">// Send the headers first to establish the connection</span>
        <span class="k">.await</span><span class="p">;</span>

    <span class="c1">// 2. Enter a loop to continuously push data</span>
    <span class="k">for</span> <span class="n">i</span> <span class="k">in</span> <span class="mi">0</span><span class="o">..</span><span class="mi">10</span> <span class="p">{</span>
        <span class="c1">// Construct the `data:` field according to the SSE specification</span>
        <span class="k">let</span> <span class="n">event_data</span> <span class="o">=</span> <span class="nd">format!</span><span class="p">(</span><span class="s">"data: Event number {}{}"</span><span class="p">,</span> <span class="n">i</span><span class="p">,</span> <span class="n">HTTP_DOUBLE_BR</span><span class="p">);</span>

        <span class="c1">// 3. Use our familiar send_body to send the event</span>
        <span class="k">let</span> <span class="n">_</span> <span class="o">=</span> <span class="n">ctx</span>
            <span class="nf">.set_response_body</span><span class="p">(</span><span class="n">event_data</span><span class="p">)</span>
            <span class="k">.await</span>
            <span class="nf">.send_body</span><span class="p">()</span>
            <span class="k">.await</span><span class="p">;</span>

        <span class="c1">// Simulate waiting for new data</span>
        <span class="nf">sleep</span><span class="p">(</span><span class="nn">Duration</span><span class="p">::</span><span class="nf">from_secs</span><span class="p">(</span><span class="mi">1</span><span class="p">))</span><span class="k">.await</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1">// 4. When we want to end, we just close the connection</span>
    <span class="k">let</span> <span class="n">_</span> <span class="o">=</span> <span class="n">ctx</span><span class="nf">.closed</span><span class="p">()</span><span class="k">.await</span><span class="p">;</span>
    <span class="nd">println!</span><span class="p">(</span><span class="s">"SSE stream finished."</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This code is as beautiful as a poem. 😍 Let's savor its elegance:</p>

<ul>
<li>
<strong>It's Just HTTP</strong>: It's a standard HTTP route. What does that mean? It means we can use all the knowledge we already have! We can add <code>auth_middleware</code> for authentication, <code>log_middleware</code> for logging. Its security and management are seamlessly integrated into the existing HTTP system.</li>
<li>
<strong>Unified API</strong>: See <code>send_body()</code>? It's back! Hyperlane uses a single, unified API to handle all types of "send" operations, whether it's an HTTP response, a WebSocket message, or an SSE event. This consistency dramatically reduces the developer's mental overhead.</li>
<li>
<strong>Simple and Clear</strong>: The entire logic is crystal clear. Set headers -&gt; send headers -&gt; loop and send data bodies -&gt; close connection. There is no magic, and everything is under control.</li>
</ul>

<p>Now look at the client-side code, which is also ridiculously simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// The browser's native EventSource API</span>
<span class="kd">const</span> <span class="nx">eventSource</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">EventSource</span><span class="p">(</span><span class="dl">'</span><span class="s1">http://127.0.0.1:60000/sse</span><span class="dl">'</span><span class="p">);</span>

<span class="c1">// Callback for when the connection is successfully opened</span>
<span class="nx">eventSource</span><span class="p">.</span><span class="nx">onopen</span> <span class="o">=</span> <span class="nf">function </span><span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">SSE Connection opened. Waiting for events... 📡</span><span class="dl">'</span><span class="p">);</span>
<span class="p">};</span>

<span class="c1">// Callback for when a message is received</span>
<span class="nx">eventSource</span><span class="p">.</span><span class="nx">onmessage</span> <span class="o">=</span> <span class="nf">function </span><span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// event.data is the content of the `data:` field sent by our server</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Received event:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">data</span><span class="p">);</span>
<span class="p">};</span>

<span class="c1">// Callback for when an error occurs</span>
<span class="nx">eventSource</span><span class="p">.</span><span class="nx">onerror</span> <span class="o">=</span> <span class="nf">function </span><span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">readyState</span> <span class="o">===</span> <span class="nx">EventSource</span><span class="p">.</span><span class="nx">CLOSED</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">SSE Connection was closed. 👋</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">SSE Error occurred:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">event</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>And the best part? <strong>The <code>EventSource</code> API natively supports automatic reconnection!</strong> 🤯 If the connection is interrupted due to network jitter, the browser will automatically try to reconnect after a few seconds. You need to write almost no extra code for this robustness. This is an effect that requires you to manually implement heartbeat and reconnection logic in WebSockets!</p>

<h3>
  
  
  Choose the Right Tool, Not the Most Famous One
</h3>

<p>I'm not saying that SSE can completely replace WebSockets. When your application needs <strong>the client to send data to the server at high frequency</strong>, or requires complex bidirectional communication, WebSockets are still the undisputed king. 👑</p>

<p>But what I am saying is that as professional engineers, we should have the ability to evaluate requirements and choose the most appropriate tool. For the vast number of scenarios that only require a "server-to-client" unidirectional data stream—real-time notifications, news feeds, status updates, data dashboards—SSE is often the simpler, lighter, more robust, and more easily integrated choice.</p>

<p>A good framework doesn't force you to solve all problems in the same way. It provides you with a sharp and professional toolset and allows you to easily choose the one that fits best. Hyperlane's seamless support for SSE is a testament to this design philosophy.</p>

<p>So, the next time you face a real-time requirement, please stop and think: do I really need a whole cow, or is a fresh glass of milk enough? Make the wise choice, and you will find your code is simpler, your system is more stable, and you are much happier. 😌</p>

<p><a href="https://github.com/hyperlane-dev/hyperlane" rel="noopener noreferrer">GitHub Home</a></p>

