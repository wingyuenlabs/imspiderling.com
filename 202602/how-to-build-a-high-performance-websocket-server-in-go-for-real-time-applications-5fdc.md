---
Title: How to Build a High-Performance WebSocket Server in Go for Real-Time Applications
Description: 
Author: Nithin Bharadwaj
Date: 2026-02-20T21:35:54.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>As a best-selling author, I invite you to explore my books on <a href="https://www.amazon.com/stores/Aarav-Joshi/author/B0DQYNVXZ7?ref=ap_rdr&amp;isDramIntegrated=true&amp;shoppingPortalEnabled=true&amp;ccs_id=738636bd-0ca1-4d7b-8efa-481bfc222571" rel="noopener noreferrer">Amazon</a>. Don't forget to follow me on <a href="https://medium.com/@aarav-joshi" rel="noopener noreferrer">Medium</a> and show your support. Thank you! Your support means the world! </p>
</blockquote>

<p>Let's talk about real-time. Think about a live sports score update on your phone, a collaborative document where you see other people's cursors moving, or a group chat where messages appear instantly. For a long time, making this happen on the web was clunky. We had techniques where your browser would constantly ask the server, "Do you have anything new for me?" over and over. It was inefficient and slow.</p>

<p>The modern solution is WebSocket. Imagine you call a friend. You dial, they answer, and the line stays open. You can both talk whenever you want, without hanging up and redialing. A WebSocket is like that open phone line between a browser and a server. Once connected, data can flow in both directions at any time. This is what makes truly interactive, real-time web applications possible.</p>

<p>Building a server that can manage thousands of these open "phone lines" efficiently is a fascinating challenge. We need to answer several questions. How do we start the call correctly every time? How do we keep track of who is connected? If someone shouts a message into a room, how do we make sure everyone else in that room hears it without overloading the system? How do we know if a line has gone dead?</p>

<p>I want to show you how to build such a server in Go. Go is a fantastic language for this. It's built for concurrency, which is just a fancy word for doing many things at once, like handling thousands of simultaneous connections. Its simplicity helps us focus on the core logic rather than complex language features.</p>

<p>Let's start by looking at the central manager of our server. We'll create a <code>WebSocketServer</code> struct. Think of this as the control room.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">WebSocketServer</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">listener</span>    <span class="n">net</span><span class="o">.</span><span class="n">Listener</span>
    <span class="n">connections</span> <span class="o">*</span><span class="n">ConnectionPool</span>
    <span class="n">rooms</span>       <span class="o">*</span><span class="n">RoomManager</span>
    <span class="n">config</span>      <span class="n">ServerConfig</span>
    <span class="n">stats</span>       <span class="n">ServerStats</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This control room has a few key parts. The <code>listener</code> waits for new connection requests. The <code>ConnectionPool</code> is the main switchboard, keeping a record of every active connection. The <code>RoomManager</code> handles grouping connections together, like putting people into different chat rooms. The <code>config</code> holds our settings, like how many connections we allow. The <code>stats</code> help us monitor performance.</p>

<p>The first step in any WebSocket conversation is the handshake. A client, like a web browser, sends a special HTTP request saying, "I want to upgrade this to a WebSocket." Our server must respond correctly. This isn't our application logic yet; it's the protocol saying, "Okay, let's switch from HTTP to a persistent connection."</p>

<p>Here's a simplified version of that handshake inside our server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">ws</span> <span class="o">*</span><span class="n">WebSocketServer</span><span class="p">)</span> <span class="n">performHandshake</span><span class="p">(</span><span class="n">rawConn</span> <span class="n">net</span><span class="o">.</span><span class="n">Conn</span><span class="p">)</span> <span class="p">(</span><span class="n">net</span><span class="o">.</span><span class="n">Conn</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="c">// Read the initial HTTP request</span>
    <span class="n">buf</span> <span class="o">:=</span> <span class="nb">make</span><span class="p">([]</span><span class="kt">byte</span><span class="p">,</span> <span class="m">4096</span><span class="p">)</span>
    <span class="n">n</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">rawConn</span><span class="o">.</span><span class="n">Read</span><span class="p">(</span><span class="n">buf</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="no">nil</span><span class="p">,</span> <span class="n">err</span>
    <span class="p">}</span>

    <span class="c">// Check if this is a WebSocket upgrade request</span>
    <span class="n">headers</span> <span class="o">:=</span> <span class="n">parseHeaders</span><span class="p">(</span><span class="n">buf</span><span class="p">[</span><span class="o">:</span><span class="n">n</span><span class="p">])</span>
    <span class="n">key</span> <span class="o">:=</span> <span class="n">headers</span><span class="p">[</span><span class="s">"Sec-WebSocket-Key"</span><span class="p">]</span>
    <span class="k">if</span> <span class="n">key</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
        <span class="k">return</span> <span class="no">nil</span><span class="p">,</span> <span class="n">errors</span><span class="o">.</span><span class="n">New</span><span class="p">(</span><span class="s">"not a websocket request"</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="c">// Calculate the required response key</span>
    <span class="n">acceptKey</span> <span class="o">:=</span> <span class="n">calculateAcceptKey</span><span class="p">(</span><span class="n">key</span><span class="p">)</span>

    <span class="c">// Send back the magic "101 Switching Protocols" response</span>
    <span class="n">response</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span>
        <span class="s">"HTTP/1.1 101 Switching Protocols</span><span class="se">\r\n</span><span class="s">"</span><span class="o">+</span>
            <span class="s">"Upgrade: websocket</span><span class="se">\r\n</span><span class="s">"</span><span class="o">+</span>
            <span class="s">"Connection: Upgrade</span><span class="se">\r\n</span><span class="s">"</span><span class="o">+</span>
            <span class="s">"Sec-WebSocket-Accept: %s</span><span class="se">\r\n\r\n</span><span class="s">"</span><span class="p">,</span>
        <span class="n">acceptKey</span><span class="p">,</span>
    <span class="p">)</span>
    <span class="n">_</span><span class="p">,</span> <span class="n">err</span> <span class="o">=</span> <span class="n">rawConn</span><span class="o">.</span><span class="n">Write</span><span class="p">([]</span><span class="kt">byte</span><span class="p">(</span><span class="n">response</span><span class="p">))</span>
    <span class="k">return</span> <span class="n">rawConn</span><span class="p">,</span> <span class="n">err</span>
<span class="p">}</span>
</code></pre>

</div>



<p>If this exchange succeeds, the ordinary HTTP connection transforms into a persistent WebSocket connection. Now the real work begins. We create an object to represent this single connection. We give it a unique ID, channels for sending and receiving data, and a way to close it cleanly.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">WSConnection</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">ID</span>        <span class="kt">uint64</span>
    <span class="n">Conn</span>      <span class="n">net</span><span class="o">.</span><span class="n">Conn</span>
    <span class="n">UserID</span>    <span class="kt">string</span>
    <span class="n">RoomIDs</span>   <span class="p">[]</span><span class="kt">string</span>
    <span class="n">SendChan</span>  <span class="k">chan</span> <span class="p">[]</span><span class="kt">byte</span>
    <span class="n">CloseChan</span> <span class="k">chan</span> <span class="k">struct</span><span class="p">{}</span>
    <span class="n">mu</span>        <span class="n">sync</span><span class="o">.</span><span class="n">RWMutex</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>SendChan</code> is crucial. In Go, channels are a safe way for different parts of your program to communicate. Instead of having one giant, tangled mess of code trying to write directly to the network connection, we have a dedicated goroutine for writing. This writer goroutine simply waits for messages to appear on the <code>SendChan</code> and sends them out. This pattern keeps things orderly and prevents multiple parts of the code from trying to write at the same time and causing errors.</p>

<p>We start three main goroutines for each connection: one for reading incoming messages, one for writing outgoing messages, and one for sending periodic "pings" to check if the connection is still alive.</p>

<p>The read loop continuously checks the network socket for new data. WebSocket data comes in structured chunks called frames. A frame tells us if this is a text message, binary data, a ping, a pong, or a close request. Our read loop decodes these frames.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">ws</span> <span class="o">*</span><span class="n">WebSocketServer</span><span class="p">)</span> <span class="n">readHandler</span><span class="p">(</span><span class="n">conn</span> <span class="o">*</span><span class="n">WSConnection</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">buffer</span> <span class="o">:=</span> <span class="nb">make</span><span class="p">([]</span><span class="kt">byte</span><span class="p">,</span> <span class="n">ws</span><span class="o">.</span><span class="n">config</span><span class="o">.</span><span class="n">MaxMessageSize</span><span class="p">)</span>
    <span class="k">for</span> <span class="p">{</span>
        <span class="n">conn</span><span class="o">.</span><span class="n">Conn</span><span class="o">.</span><span class="n">SetReadDeadline</span><span class="p">(</span><span class="n">time</span><span class="o">.</span><span class="n">Now</span><span class="p">()</span><span class="o">.</span><span class="n">Add</span><span class="p">(</span><span class="n">ws</span><span class="o">.</span><span class="n">config</span><span class="o">.</span><span class="n">ReadTimeout</span><span class="p">))</span>
        <span class="n">n</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">conn</span><span class="o">.</span><span class="n">Conn</span><span class="o">.</span><span class="n">Read</span><span class="p">(</span><span class="n">buffer</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">break</span> <span class="c">// Connection is likely closed</span>
        <span class="p">}</span>

        <span class="n">frame</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">parseFrame</span><span class="p">(</span><span class="n">buffer</span><span class="p">[</span><span class="o">:</span><span class="n">n</span><span class="p">])</span>
        <span class="k">switch</span> <span class="n">frame</span><span class="o">.</span><span class="n">OpCode</span> <span class="p">{</span>
        <span class="k">case</span> <span class="n">OpCodeText</span><span class="o">:</span>
            <span class="n">ws</span><span class="o">.</span><span class="n">handleTextMessage</span><span class="p">(</span><span class="n">conn</span><span class="p">,</span> <span class="n">frame</span><span class="o">.</span><span class="n">Payload</span><span class="p">)</span>
        <span class="k">case</span> <span class="n">OpCodeClose</span><span class="o">:</span>
            <span class="n">ws</span><span class="o">.</span><span class="n">handleClose</span><span class="p">(</span><span class="n">conn</span><span class="p">,</span> <span class="n">frame</span><span class="o">.</span><span class="n">Payload</span><span class="p">)</span>
            <span class="k">return</span>
        <span class="k">case</span> <span class="n">OpCodePing</span><span class="o">:</span>
            <span class="n">ws</span><span class="o">.</span><span class="n">handlePing</span><span class="p">(</span><span class="n">conn</span><span class="p">,</span> <span class="n">frame</span><span class="o">.</span><span class="n">Payload</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>
    <span class="n">ws</span><span class="o">.</span><span class="n">closeConnection</span><span class="p">(</span><span class="n">conn</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>When we get a text message, we need to understand what the client wants to do. In our example, we'll support a few simple actions: joining a room, leaving a room, and broadcasting a message to a room. We can define a simple message format using JSON.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"join"</span><span class="p">,</span><span class="w"> </span><span class="nl">"room_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"game_lobby"</span><span class="p">,</span><span class="w"> </span><span class="nl">"user_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"alice123"</span><span class="p">}</span><span class="w">
</span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"broadcast"</span><span class="p">,</span><span class="w"> </span><span class="nl">"room_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"game_lobby"</span><span class="p">,</span><span class="w"> </span><span class="nl">"content"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Hello everyone!"</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Our <code>handleTextMessage</code> function would parse this JSON and call the appropriate handler. Let's look at what happens for a "join" request.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">ws</span> <span class="o">*</span><span class="n">WebSocketServer</span><span class="p">)</span> <span class="n">handleJoin</span><span class="p">(</span><span class="n">conn</span> <span class="o">*</span><span class="n">WSConnection</span><span class="p">,</span> <span class="n">msg</span> <span class="o">*</span><span class="n">Message</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">roomID</span> <span class="o">:=</span> <span class="n">msg</span><span class="o">.</span><span class="n">Data</span><span class="p">[</span><span class="s">"room_id"</span><span class="p">]</span><span class="o">.</span><span class="p">(</span><span class="kt">string</span><span class="p">)</span>
    <span class="n">userID</span> <span class="o">:=</span> <span class="n">msg</span><span class="o">.</span><span class="n">Data</span><span class="p">[</span><span class="s">"user_id"</span><span class="p">]</span><span class="o">.</span><span class="p">(</span><span class="kt">string</span><span class="p">)</span>

    <span class="n">conn</span><span class="o">.</span><span class="n">mu</span><span class="o">.</span><span class="n">Lock</span><span class="p">()</span>
    <span class="n">conn</span><span class="o">.</span><span class="n">UserID</span> <span class="o">=</span> <span class="n">userID</span>
    <span class="n">conn</span><span class="o">.</span><span class="n">RoomIDs</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">conn</span><span class="o">.</span><span class="n">RoomIDs</span><span class="p">,</span> <span class="n">roomID</span><span class="p">)</span>
    <span class="n">conn</span><span class="o">.</span><span class="n">mu</span><span class="o">.</span><span class="n">Unlock</span><span class="p">()</span>

    <span class="c">// Tell the room manager about this new member</span>
    <span class="n">ws</span><span class="o">.</span><span class="n">rooms</span><span class="o">.</span><span class="n">AddConnection</span><span class="p">(</span><span class="n">roomID</span><span class="p">,</span> <span class="n">conn</span><span class="o">.</span><span class="n">ID</span><span class="p">)</span>
    <span class="c">// Update the connection pool's index</span>
    <span class="n">ws</span><span class="o">.</span><span class="n">connections</span><span class="o">.</span><span class="n">AddToRoom</span><span class="p">(</span><span class="n">roomID</span><span class="p">,</span> <span class="n">conn</span><span class="o">.</span><span class="n">ID</span><span class="p">)</span>

    <span class="c">// Send a confirmation back to the client</span>
    <span class="n">response</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">json</span><span class="o">.</span><span class="n">Marshal</span><span class="p">(</span><span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="kt">string</span><span class="p">{</span><span class="s">"type"</span><span class="o">:</span> <span class="s">"joined"</span><span class="p">,</span> <span class="s">"room_id"</span><span class="o">:</span> <span class="n">roomID</span><span class="p">})</span>
    <span class="n">conn</span><span class="o">.</span><span class="n">SendChan</span> <span class="o">&lt;-</span> <span class="n">response</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now, what about broadcasting a message? This is where performance becomes critical. If a room has 10,000 members, we need to send the message to 10,000 connections as quickly as possible, without blocking the sender.</p>

<p>Our <code>RoomManager</code> keeps a list of connection IDs for each room. When a broadcast comes in, we get that list.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">ws</span> <span class="o">*</span><span class="n">WebSocketServer</span><span class="p">)</span> <span class="n">handleBroadcast</span><span class="p">(</span><span class="n">conn</span> <span class="o">*</span><span class="n">WSConnection</span><span class="p">,</span> <span class="n">msg</span> <span class="o">*</span><span class="n">Message</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">roomID</span> <span class="o">:=</span> <span class="n">msg</span><span class="o">.</span><span class="n">Data</span><span class="p">[</span><span class="s">"room_id"</span><span class="p">]</span><span class="o">.</span><span class="p">(</span><span class="kt">string</span><span class="p">)</span>
    <span class="n">content</span> <span class="o">:=</span> <span class="n">msg</span><span class="o">.</span><span class="n">Data</span><span class="p">[</span><span class="s">"content"</span><span class="p">]</span><span class="o">.</span><span class="p">(</span><span class="kt">string</span><span class="p">)</span>

    <span class="n">connections</span> <span class="o">:=</span> <span class="n">ws</span><span class="o">.</span><span class="n">rooms</span><span class="o">.</span><span class="n">GetConnections</span><span class="p">(</span><span class="n">roomID</span><span class="p">)</span>
    <span class="n">broadcastMsg</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">json</span><span class="o">.</span><span class="n">Marshal</span><span class="p">(</span><span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="k">interface</span><span class="p">{}{</span>
        <span class="s">"type"</span><span class="o">:</span>    <span class="s">"broadcast"</span><span class="p">,</span>
        <span class="s">"content"</span><span class="o">:</span> <span class="n">content</span><span class="p">,</span>
        <span class="s">"sender"</span><span class="o">:</span>  <span class="n">conn</span><span class="o">.</span><span class="n">UserID</span><span class="p">,</span>
    <span class="p">})</span>

    <span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">connID</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">connections</span> <span class="p">{</span>
        <span class="k">if</span> <span class="n">targetConn</span> <span class="o">:=</span> <span class="n">ws</span><span class="o">.</span><span class="n">connections</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">connID</span><span class="p">);</span> <span class="n">targetConn</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
            <span class="k">select</span> <span class="p">{</span>
            <span class="k">case</span> <span class="n">targetConn</span><span class="o">.</span><span class="n">SendChan</span> <span class="o">&lt;-</span> <span class="n">broadcastMsg</span><span class="o">:</span>
                <span class="c">// Successfully queued the message</span>
            <span class="k">default</span><span class="o">:</span>
                <span class="c">// The SendChan is full. The client might be slow.</span>
                <span class="c">// We can drop the message or handle the backlog.</span>
                <span class="n">log</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Client buffer full, dropping message for"</span><span class="p">,</span> <span class="n">connID</span><span class="p">)</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Notice the <code>select</code> statement with a <code>default</code> case. This is a non-blocking send. If the client's <code>SendChan</code> is full (maybe their network is slow), we don't want to wait. We simply skip them and move on. This prevents one slow client from holding up messages for everyone else. You might choose to close the connection instead, depending on your application's needs.</p>

<p>Keeping connections alive is another important job. Networks are unreliable. A client might lose WiFi or close their laptop. We need to detect this to free up resources. We do this with a heartbeat. The <code>pingHandler</code> sends a small "ping" frame at regular intervals.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">ws</span> <span class="o">*</span><span class="n">WebSocketServer</span><span class="p">)</span> <span class="n">pingHandler</span><span class="p">(</span><span class="n">conn</span> <span class="o">*</span><span class="n">WSConnection</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">ticker</span> <span class="o">:=</span> <span class="n">time</span><span class="o">.</span><span class="n">NewTicker</span><span class="p">(</span><span class="n">ws</span><span class="o">.</span><span class="n">config</span><span class="o">.</span><span class="n">PingInterval</span><span class="p">)</span>
    <span class="k">defer</span> <span class="n">ticker</span><span class="o">.</span><span class="n">Stop</span><span class="p">()</span>

    <span class="k">for</span> <span class="p">{</span>
        <span class="k">select</span> <span class="p">{</span>
        <span class="k">case</span> <span class="o">&lt;-</span><span class="n">ticker</span><span class="o">.</span><span class="n">C</span><span class="o">:</span>
            <span class="n">pingFrame</span> <span class="o">:=</span> <span class="n">createFrame</span><span class="p">(</span><span class="n">OpCodePing</span><span class="p">,</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">(</span><span class="s">"ping"</span><span class="p">),</span> <span class="no">true</span><span class="p">)</span>
            <span class="n">conn</span><span class="o">.</span><span class="n">Conn</span><span class="o">.</span><span class="n">SetWriteDeadline</span><span class="p">(</span><span class="n">time</span><span class="o">.</span><span class="n">Now</span><span class="p">()</span><span class="o">.</span><span class="n">Add</span><span class="p">(</span><span class="n">ws</span><span class="o">.</span><span class="n">config</span><span class="o">.</span><span class="n">WriteTimeout</span><span class="p">))</span>
            <span class="n">conn</span><span class="o">.</span><span class="n">Conn</span><span class="o">.</span><span class="n">Write</span><span class="p">(</span><span class="n">pingFrame</span><span class="p">)</span>

            <span class="c">// Start a timer, waiting for the "pong" reply</span>
            <span class="k">go</span> <span class="n">ws</span><span class="o">.</span><span class="n">waitForPong</span><span class="p">(</span><span class="n">conn</span><span class="p">)</span>
        <span class="k">case</span> <span class="o">&lt;-</span><span class="n">conn</span><span class="o">.</span><span class="n">CloseChan</span><span class="o">:</span>
            <span class="k">return</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">ws</span> <span class="o">*</span><span class="n">WebSocketServer</span><span class="p">)</span> <span class="n">waitForPong</span><span class="p">(</span><span class="n">conn</span> <span class="o">*</span><span class="n">WSConnection</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">select</span> <span class="p">{</span>
    <span class="k">case</span> <span class="o">&lt;-</span><span class="n">time</span><span class="o">.</span><span class="n">After</span><span class="p">(</span><span class="n">ws</span><span class="o">.</span><span class="n">config</span><span class="o">.</span><span class="n">PongTimeout</span><span class="p">)</span><span class="o">:</span>
        <span class="n">ws</span><span class="o">.</span><span class="n">closeConnection</span><span class="p">(</span><span class="n">conn</span><span class="p">)</span> <span class="c">// No pong received, close it.</span>
    <span class="k">case</span> <span class="o">&lt;-</span><span class="n">conn</span><span class="o">.</span><span class="n">CloseChan</span><span class="o">:</span>
        <span class="k">return</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>If we send a ping and don't get a pong response within a set time, we assume the connection is dead and close it. This cleanup is vital. The <code>closeConnection</code> function is the orderly shutdown procedure. It removes the connection from all rooms, deletes it from the main connection pool, closes the network socket, and updates our statistics.</p>

<p>Talking about statistics, let's see how we can monitor our server. We can use atomic counters to track metrics without using heavy locks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">ServerStats</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">ConnectionsTotal</span>   <span class="kt">uint64</span>
    <span class="n">ConnectionsActive</span>  <span class="kt">uint64</span>
    <span class="n">MessagesSent</span>       <span class="kt">uint64</span>
    <span class="n">MessagesReceived</span>   <span class="kt">uint64</span>
    <span class="n">MessagesPerSecond</span>  <span class="kt">float64</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">ws</span> <span class="o">*</span><span class="n">WebSocketServer</span><span class="p">)</span> <span class="n">collectStats</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">ticker</span> <span class="o">:=</span> <span class="n">time</span><span class="o">.</span><span class="n">NewTicker</span><span class="p">(</span><span class="m">1</span> <span class="o">*</span> <span class="n">time</span><span class="o">.</span><span class="n">Second</span><span class="p">)</span>
    <span class="k">var</span> <span class="n">lastMessageCount</span> <span class="kt">uint64</span>
    <span class="k">for</span> <span class="k">range</span> <span class="n">ticker</span><span class="o">.</span><span class="n">C</span> <span class="p">{</span>
        <span class="n">current</span> <span class="o">:=</span> <span class="n">atomic</span><span class="o">.</span><span class="n">LoadUint64</span><span class="p">(</span><span class="o">&amp;</span><span class="n">ws</span><span class="o">.</span><span class="n">stats</span><span class="o">.</span><span class="n">MessagesSent</span><span class="p">)</span> <span class="o">+</span> <span class="n">atomic</span><span class="o">.</span><span class="n">LoadUint64</span><span class="p">(</span><span class="o">&amp;</span><span class="n">ws</span><span class="o">.</span><span class="n">stats</span><span class="o">.</span><span class="n">MessagesReceived</span><span class="p">)</span>
        <span class="n">delta</span> <span class="o">:=</span> <span class="n">current</span> <span class="o">-</span> <span class="n">lastMessageCount</span>
        <span class="n">ws</span><span class="o">.</span><span class="n">stats</span><span class="o">.</span><span class="n">MessagesPerSecond</span> <span class="o">=</span> <span class="kt">float64</span><span class="p">(</span><span class="n">delta</span><span class="p">)</span> <span class="c">// Messages in the last second</span>
        <span class="n">lastMessageCount</span> <span class="o">=</span> <span class="n">current</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>We can expose these stats via a simple HTTP endpoint on a different port, which is safe and common for health checks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">go</span> <span class="k">func</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">http</span><span class="o">.</span><span class="n">HandleFunc</span><span class="p">(</span><span class="s">"/health"</span><span class="p">,</span> <span class="k">func</span><span class="p">(</span><span class="n">w</span> <span class="n">http</span><span class="o">.</span><span class="n">ResponseWriter</span><span class="p">,</span> <span class="n">r</span> <span class="o">*</span><span class="n">http</span><span class="o">.</span><span class="n">Request</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">stats</span> <span class="o">:=</span> <span class="n">server</span><span class="o">.</span><span class="n">GetStats</span><span class="p">()</span>
        <span class="n">fmt</span><span class="o">.</span><span class="n">Fprintf</span><span class="p">(</span><span class="n">w</span><span class="p">,</span> <span class="s">`{"active_connections": %d}`</span><span class="p">,</span> <span class="n">stats</span><span class="o">.</span><span class="n">ConnectionsActive</span><span class="p">)</span>
    <span class="p">})</span>
    <span class="n">http</span><span class="o">.</span><span class="n">ListenAndServe</span><span class="p">(</span><span class="s">":8081"</span><span class="p">,</span> <span class="no">nil</span><span class="p">)</span>
<span class="p">}()</span>
</code></pre>

</div>



<p>Finally, we start everything in our main function.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">server</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">NewWebSocketServer</span><span class="p">(</span><span class="s">":8080"</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="n">err</span><span class="p">)</span>
    <span class="p">}</span>
    <span class="n">log</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"Server starting on :8080"</span><span class="p">)</span>
    <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="n">server</span><span class="o">.</span><span class="n">Start</span><span class="p">())</span>
<span class="p">}</span>
</code></pre>

</div>



<p>When you run this, you have a functional WebSocket server. It accepts connections, groups them into rooms, broadcasts messages efficiently, and cleans up after itself. This is a solid foundation. From here, you could add more features: user authentication before joining rooms, private direct messages between users, saving message history to a database, or scaling horizontally by connecting multiple server instances together through a message bus like Redis.</p>

<h2>
  
  
  Building this piece by piece helps you understand what makes real-time systems tick. You see how connections are managed as resources, how messages are routed without bottlenecks, and how the system stays healthy under load. It's a satisfying project that brings a fundamental web technology to life.
</h2>

<p>📘 <strong>Checkout my <a href="https://youtu.be/WpR6F4ky4uM" rel="noopener noreferrer">latest ebook</a> for free on my channel!</strong><br><br>
Be sure to <strong>like</strong>, <strong>share</strong>, <strong>comment</strong>, and <strong>subscribe</strong> to the channel!</p>




<h2>
  
  
  101 Books
</h2>

<p><strong>101 Books</strong> is an AI-driven publishing company co-founded by author <strong>Aarav Joshi</strong>. By leveraging advanced AI technology, we keep our publishing costs incredibly low—some books are priced as low as <strong>$4</strong>—making quality knowledge accessible to everyone.</p>

<p>Check out our book <strong><a href="https://www.amazon.com/dp/B0DQQF9K3Z" rel="noopener noreferrer">Golang Clean Code</a></strong> available on Amazon. </p>

<p>Stay tuned for updates and exciting news. When shopping for books, search for <strong>Aarav Joshi</strong> to find more of our titles. Use the provided link to enjoy <strong>special discounts</strong>!</p>

<h2>
  
  
  Our Creations
</h2>

<p>Be sure to check out our creations:</p>

<p><strong><a href="https://www.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central</a></strong> | <strong><a href="https://spanish.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central Spanish</a></strong> | <strong><a href="https://german.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central German</a></strong> | <strong><a href="https://smartliving.investorcentral.co.uk/" rel="noopener noreferrer">Smart Living</a></strong> | <strong><a href="https://epochsandechoes.com/" rel="noopener noreferrer">Epochs &amp; Echoes</a></strong> | <strong><a href="https://www.puzzlingmysteries.com/" rel="noopener noreferrer">Puzzling Mysteries</a></strong> | <strong><a href="http://hindutva.epochsandechoes.com/" rel="noopener noreferrer">Hindutva</a></strong> | <strong><a href="https://elitedev.in/" rel="noopener noreferrer">Elite Dev</a></strong> | <strong><a href="https://java.elitedev.in/" rel="noopener noreferrer">Java Elite Dev</a></strong> | <strong><a href="https://golang.elitedev.in/" rel="noopener noreferrer">Golang Elite Dev</a></strong> | <strong><a href="https://python.elitedev.in/" rel="noopener noreferrer">Python Elite Dev</a></strong> | <strong><a href="https://js.elitedev.in/" rel="noopener noreferrer">JS Elite Dev</a></strong> | <strong><a href="https://jsschools.com/" rel="noopener noreferrer">JS Schools</a></strong></p>




<h3>
  
  
  We are on Medium
</h3>

<p><strong><a href="https://techkoalainsights.com/" rel="noopener noreferrer">Tech Koala Insights</a></strong> | <strong><a href="https://world.epochsandechoes.com/" rel="noopener noreferrer">Epochs &amp; Echoes World</a></strong> | <strong><a href="https://medium.investorcentral.co.uk/" rel="noopener noreferrer">Investor Central Medium</a></strong> | <strong><a href="https://medium.com/puzzling-mysteries" rel="noopener noreferrer">Puzzling Mysteries Medium</a></strong> | <strong><a href="https://science.epochsandechoes.com/" rel="noopener noreferrer">Science &amp; Epochs Medium</a></strong> | <strong><a href="https://modernhindutva.substack.com/" rel="noopener noreferrer">Modern Hindutva</a></strong></p>

