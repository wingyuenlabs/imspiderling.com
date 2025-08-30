---
Title: I built a Backend web Framework from Scratch in C++
Description: 
Author: Hamza Hasanain
Date: 2025-08-30T21:10:32.000Z
Robots: noindex,nofollow
Template: index
---
<h4>
  
  
  wouldn’t go as far as calling it a framework — it’s more of a library
</h4>

<p>I’ve been exploring some backend web frameworks lately and kept asking myself: <em>what do these things actually do under the hood?</em></p>

<p>To find out, I decided to dive into C++ and experiment. After some tinkering, I built a small homegrown backend web library, split into three layers:</p>

<ol>
<li>
<strong>Socket Library</strong> – Handles raw communication between processes.</li>
<li>
<strong>HTTP Server</strong> – Parses HTTP requests, manages headers and bodies, and handles TCP streams.</li>
<li>
<strong>Web Library</strong> – Provides a simple framework for routing, controllers, and serving static files, similar to Express.js.</li>
</ol>

<p>Each layer is built on top of the one beneath it, so understanding the foundation is crucial.</p>

<p>Before we dive into the layers, you can check out the GitHub repos:</p>

<ul>
<li>Socket Library: <a href="https://github.com/HamzaHassanain/hamza-socket-lib" rel="noopener noreferrer">github.com/HamzaHassanain/hamza-socket-lib</a>
</li>
<li>HTTP Server: <a href="https://github.com/HamzaHassanain/hamza-http-server-lib" rel="noopener noreferrer">github.com/HamzaHassanain/hamza-http-server-lib</a>
</li>
<li>Web Library: <a href="https://github.com/HamzaHassanain/hamza-backend-web-library-cpp" rel="noopener noreferrer">github.com/HamzaHassanain/hamza-backend-web-library-cpp</a>
</li>
<li>Example Blog App: <a href="https://github.com/HamzaHassanain/simple-blog-from-scratch" rel="noopener noreferrer">github.com/HamzaHassanain/simple-blog-from-scratch</a>
</li>
</ul>

<p><strong>Please note that this is just my understanding of how things work, how I implemented the stuff</strong></p>

<p>Also note that this project isn’t fully production-ready, but it’s an excellent exercise in <strong>understanding backend frameworks from the ground up</strong>.</p>




<h2>
  
  
  Understanding Sockets: How Processes Talk
</h2>

<p>At the core of networking on Unix-like systems are <strong>file descriptors (FDs)</strong> — small integers a process uses to refer to kernel-managed resources (files, pipes, or sockets). When you call something like <code>fflush(stdout)</code> you’re asking your program’s runtime to push buffered bytes down to the FD that represents <code>stdout</code>; what happens to those bytes next depends on what that FD is connected to (a terminal, a file, or a socket).</p>

<p>A <strong>socket</strong> is one of those kernel-managed resources: it’s a kernel object that your process creates with <code>socket(...)</code> and then uses to send and receive network data. You can think of a socket as an endpoint inside your program; the socket itself is represented by an FD in your process. To tell the kernel where packets should go (or where they came from), a socket is usually <strong>bound</strong> to a network <em>address</em>, which is commonly expressed as three parts:</p>

<ul>
<li>
<strong>Address family</strong> — how to interpret addresses (IPv4 or IPv6, e.g. <code>AF_INET</code> or <code>AF_INET6</code>).</li>
<li>
<strong>IP address</strong> — which host/machine on the network you mean (e.g. <code>127.0.0.1</code>).</li>
<li>
<strong>Port</strong> — which particular service or process on that host should receive the traffic (e.g. <code>80</code>, <code>8080</code>).</li>
</ul>

<blockquote>
<p>Note: Only ports 0–1023 are reserved for well-known services like HTTP (80) or SSH (22). Ports above that are available for general use.</p>
</blockquote>

<h3>
  
  
  Socket types
</h3>

<p>Two socket types are most relevant when writing networked servers:</p>

<p><strong>1) Datagram sockets (UDP)</strong></p>

<ul>
<li>UDP sockets are <em>connectionless</em>: you can call <code>sendto()</code> with any destination address (IP+port) and the kernel will attempt to deliver that single datagram.</li>
<li>Each <code>recvfrom()</code> or <code>recvmsg()</code> call returns exactly one datagram (so message boundaries are preserved).</li>
<li>There is no handshake, and the network does not guarantee delivery, ordering, or uniqueness — datagrams can be lost, duplicated, or arrive out of order.</li>
<li>It’s common to bind a UDP socket to a port and serve many different remote peers on that single FD; the kernel provides the sender’s address on each receive so you can reply.</li>
</ul>

<p><strong>2) Stream sockets (TCP)</strong></p>

<ul>
<li>TCP sockets are <em>connection-oriented</em>: the client and server perform a 3-way handshake to establish a connection.</li>
<li>After the handshake the kernel exposes a reliable, ordered <strong>byte stream</strong> to your process. TCP ensures bytes are delivered and in order (barring extreme failures), but it does not preserve packet/message boundaries; if you send two <code>write()</code> calls on the sender, the receiver may receive them merged or split across <code>read()</code> calls.</li>
<li>For servers you <code>bind()</code> and <code>listen()</code> on a port. <code>accept()</code> returns a brand-new FD representing the established connection; the listening FD continues to accept more connections. Each client connection has its own kernel socket object and FD.</li>
</ul>

<p><strong>Notes on scale &amp; semantics</strong></p>

<ul>
<li>For UDP you can call <code>connect()</code> on the socket to set a default peer (useful to avoid passing addr to every <code>sendto()</code>), but <code>connect()</code> on a UDP socket only sets the default destination — the underlying semantics remain datagram and connectionless.</li>
<li>For TCP, <code>accept()</code> and the new FD are what you use to <code>read()</code>/<code>write()</code> that client's data; the listening socket never carries per-client data.</li>
<li>Remember: “ordered bytes” (TCP) ≠ “preserved messages” — if you need discrete messages on top of TCP, implement framing (length prefix, delimiters, etc.).</li>
</ul>

<h4>
  
  
  Creating a Simple UDP Socket
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// Creating a simple UDP socket with address in C-style (On Unix)</span>

<span class="kt">int</span> <span class="n">sockfd</span> <span class="o">=</span> <span class="n">socket</span><span class="p">(</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">SOCK_DGRAM</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
<span class="k">if</span> <span class="p">(</span><span class="n">sockfd</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Handle error</span>
<span class="p">}</span>

<span class="k">struct</span> <span class="nc">sockaddr_in</span> <span class="n">addr</span><span class="p">;</span>
<span class="n">memset</span><span class="p">(</span><span class="o">&amp;</span><span class="n">addr</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="k">sizeof</span><span class="p">(</span><span class="n">addr</span><span class="p">));</span>
<span class="n">addr</span><span class="p">.</span><span class="n">sin_family</span> <span class="o">=</span> <span class="n">AF_INET</span><span class="p">;</span>
<span class="n">addr</span><span class="p">.</span><span class="n">sin_port</span> <span class="o">=</span> <span class="n">htons</span><span class="p">(</span><span class="mi">8080</span><span class="p">);</span>
<span class="n">inet_pton</span><span class="p">(</span><span class="n">AF_INET</span><span class="p">,</span> <span class="s">"127.0.0.1"</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">addr</span><span class="p">.</span><span class="n">sin_addr</span><span class="p">);</span>

<span class="k">if</span> <span class="p">(</span><span class="n">bind</span><span class="p">(</span><span class="n">sockfd</span><span class="p">,</span> <span class="p">(</span><span class="k">struct</span> <span class="nc">sockaddr</span><span class="o">*</span><span class="p">)</span><span class="o">&amp;</span><span class="n">addr</span><span class="p">,</span> <span class="k">sizeof</span><span class="p">(</span><span class="n">addr</span><span class="p">))</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Handle error</span>
<span class="p">}</span>

<span class="c1">// Use the socket...</span>

<span class="c1">// Cleanup</span>
<span class="n">close</span><span class="p">(</span><span class="n">sockfd</span><span class="p">);</span>
<span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
</code></pre>

</div>



<h4>
  
  
  Creating a Simple UDP Socket (using my socket library)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// Creating a simple UDP socket with address Using My library (same logic as above but wrapped in my library)</span>

<span class="k">using</span> <span class="k">namespace</span> <span class="n">hh_socket</span><span class="p">;</span>

<span class="n">socket_address</span> <span class="nf">addr</span><span class="p">(</span> <span class="n">port</span><span class="p">(</span><span class="mi">8080</span><span class="p">)</span> <span class="p">,</span>  <span class="n">ip_address</span><span class="p">(</span><span class="s">"127.0.0.1"</span><span class="p">)</span> <span class="p">,</span> <span class="n">family</span><span class="p">(</span><span class="n">IPV4</span><span class="p">)</span> <span class="p">);</span>

<span class="n">socket</span> <span class="nf">sock</span><span class="p">(</span><span class="n">addr</span> <span class="p">,</span> <span class="n">Protocol</span><span class="o">::</span><span class="n">UDP</span><span class="p">);</span>

<span class="c1">// Cleanup is handled by destructor</span>
</code></pre>

</div>



<h4>
  
  
  Creating a Simple TCP Server (using my socket library)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// Creating a simple TCP server, that echoes back messages</span>

<span class="k">using</span> <span class="k">namespace</span> <span class="n">hh_socket</span><span class="p">;</span>

<span class="n">socket_address</span> <span class="nf">addr</span><span class="p">(</span> <span class="n">port</span><span class="p">(</span><span class="mi">8080</span><span class="p">)</span> <span class="p">,</span>  <span class="n">ip_address</span><span class="p">(</span><span class="s">"0.0.0.0"</span><span class="p">)</span> <span class="p">,</span> <span class="n">family</span><span class="p">(</span><span class="n">IPV4</span><span class="p">)</span> <span class="p">);</span>

<span class="n">socket</span> <span class="nf">sock</span><span class="p">(</span><span class="n">addr</span> <span class="p">,</span> <span class="n">Protocol</span><span class="o">::</span><span class="n">TCP</span><span class="p">);</span>

<span class="n">sock</span><span class="p">.</span><span class="n">listen</span><span class="p">();</span>

<span class="k">while</span> <span class="p">(</span><span class="nb">true</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">connection</span><span class="o">&gt;</span> <span class="n">conn</span> <span class="o">=</span> <span class="n">sock</span><span class="p">.</span><span class="n">accept</span><span class="p">();</span> <span class="c1">// blocking call</span>

    <span class="n">data_buffer</span> <span class="n">buf</span> <span class="o">=</span> <span class="n">conn</span><span class="o">-&gt;</span><span class="n">receive</span><span class="p">();</span> <span class="c1">// blocking call</span>
    <span class="n">data_buffer</span> <span class="n">echo_message</span> <span class="p">(</span><span class="s">"Echo: "</span><span class="p">);</span>

    <span class="n">echo_message</span><span class="p">.</span><span class="n">append</span><span class="p">(</span><span class="n">buf</span><span class="p">);</span>

    <span class="n">conn</span><span class="o">-&gt;</span><span class="n">send</span><span class="p">(</span><span class="n">echo_message</span><span class="p">);</span> <span class="c1">// echo back</span>
    <span class="n">conn</span><span class="o">-&gt;</span><span class="n">close</span><span class="p">();</span>
<span class="p">}</span>

<span class="c1">// Cleanup is handled by destructor</span>
</code></pre>

</div>






<h3>
  
  
  Handling Blocking Operations
</h3>

<p>Blocking means an I/O call (like <code>read()</code> or <code>accept()</code>) makes your program wait until the operation completes.</p>

<p>When a socket call blocks, the current thread simply sits idle until the OS has data to return or the requested action completes. For servers that handle many clients, blocking on a single thread quickly becomes a bottleneck. To handle many connections efficiently, you can use:</p>

<p><strong>1. Multithreading</strong></p>

<ul>
<li>Idea: handle each connection in its own thread or use a pool of worker threads.</li>
<li>Pros: simple mental model — each handler can use blocking calls; easy to write.</li>
<li>Cons: high memory/context-switch cost for many connections; synchronization complexity.</li>
</ul>

<p><strong>2. I/O multiplexing</strong></p>

<ul>
<li>Idea: a single thread (or a few threads) waits on many file descriptors and reacts when any become ready. Tools: <code>epoll</code> (Linux), <code>IOCP</code> (Windows), <code>kqueue</code> (macOS). There is also <code>select</code> (Windows/Unix) it is less efficient for large numbers of connections.</li>
<li>Pros: low thread overhead; great for many concurrent connections.</li>
<li>Cons: more complex control flow; must handle partial reads/writes.</li>
</ul>

<p><strong>3. Async I/O</strong></p>

<ul>
<li>Idea: submit read/write requests to the kernel and receive completion events later (no thread is blocked waiting). <code>io_uring</code> on modern Linux is a powerful example.</li>
<li>Pros: excellent throughput and low latency; scales well.</li>
<li>Cons: API is more advanced; portability issues.</li>
</ul>

<p>For my project, I used <strong>I/O multiplexing</strong>, allowing a single-threaded event loop to handle hundreds of connections efficiently.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// a simple epoll server (using my epoll_server class) for handling multiple sockets.</span>

<span class="k">class</span> <span class="nc">chat_server</span> <span class="o">:</span> <span class="k">public</span> <span class="n">epoll_server</span> <span class="p">{</span>
    <span class="n">std</span><span class="o">::</span><span class="n">unordered_map</span><span class="o">&lt;</span><span class="kt">int</span><span class="p">,</span> <span class="n">std</span><span class="o">::</span><span class="n">string</span><span class="o">&gt;</span> <span class="n">usernames</span><span class="p">;</span>

<span class="nl">protected:</span>
    <span class="kt">void</span> <span class="n">on_connection_opened</span><span class="p">(</span><span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">connection</span><span class="o">&gt;</span> <span class="n">conn</span><span class="p">)</span> <span class="k">override</span> <span class="p">{</span>
        <span class="n">send_message</span><span class="p">(</span><span class="n">conn</span><span class="p">,</span> <span class="n">data_buffer</span><span class="p">(</span><span class="s">"Enter username: "</span><span class="p">));</span>
    <span class="p">}</span>

    <span class="kt">void</span> <span class="nf">on_message_received</span><span class="p">(</span><span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">connection</span><span class="o">&gt;</span> <span class="n">conn</span><span class="p">,</span> <span class="k">const</span> <span class="n">data_buffer</span> <span class="o">&amp;</span><span class="n">db</span><span class="p">)</span> <span class="k">override</span> <span class="p">{</span>
        <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">msg</span> <span class="o">=</span> <span class="n">db</span><span class="p">.</span><span class="n">to_string</span><span class="p">();</span>

        <span class="k">if</span> <span class="p">(</span><span class="n">usernames</span><span class="p">.</span><span class="n">find</span><span class="p">(</span><span class="n">conn</span><span class="o">-&gt;</span><span class="n">get_fd</span><span class="p">())</span> <span class="o">==</span> <span class="n">usernames</span><span class="p">.</span><span class="n">end</span><span class="p">())</span> <span class="p">{</span>
            <span class="c1">// First message is username</span>
            <span class="n">usernames</span><span class="p">[</span><span class="n">conn</span><span class="o">-&gt;</span><span class="n">get_fd</span><span class="p">()]</span> <span class="o">=</span> <span class="n">msg</span><span class="p">;</span>
            <span class="n">broadcast</span><span class="p">(</span><span class="n">usernames</span><span class="p">[</span><span class="n">conn</span><span class="o">-&gt;</span><span class="n">get_fd</span><span class="p">()]</span> <span class="o">+</span> <span class="s">" joined the chat"</span><span class="p">);</span>
        <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
            <span class="c1">// Regular chat message</span>
            <span class="n">broadcast</span><span class="p">(</span><span class="n">usernames</span><span class="p">[</span><span class="n">conn</span><span class="o">-&gt;</span><span class="n">get_fd</span><span class="p">()]</span> <span class="o">+</span> <span class="s">": "</span> <span class="o">+</span> <span class="n">msg</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="kt">void</span> <span class="nf">on_connection_closed</span><span class="p">(</span><span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">connection</span><span class="o">&gt;</span> <span class="n">conn</span><span class="p">)</span> <span class="k">override</span> <span class="p">{</span>
        <span class="k">auto</span> <span class="n">it</span> <span class="o">=</span> <span class="n">usernames</span><span class="p">.</span><span class="n">find</span><span class="p">(</span><span class="n">conn</span><span class="o">-&gt;</span><span class="n">get_fd</span><span class="p">());</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">it</span> <span class="o">!=</span> <span class="n">usernames</span><span class="p">.</span><span class="n">end</span><span class="p">())</span> <span class="p">{</span>
            <span class="n">broadcast</span><span class="p">(</span><span class="n">it</span><span class="o">-&gt;</span><span class="n">second</span> <span class="o">+</span> <span class="s">" left the chat"</span><span class="p">);</span>
            <span class="n">usernames</span><span class="p">.</span><span class="n">erase</span><span class="p">(</span><span class="n">it</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>

<span class="k">private</span><span class="o">:</span>
    <span class="kt">void</span> <span class="nf">broadcast</span><span class="p">(</span><span class="k">const</span> <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="o">&amp;</span><span class="n">message</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">data_buffer</span> <span class="n">msg</span><span class="p">(</span><span class="n">message</span><span class="p">);</span>
        <span class="k">for</span> <span class="p">(</span><span class="k">const</span> <span class="k">auto</span> <span class="o">&amp;</span><span class="p">[</span><span class="n">fd</span><span class="p">,</span> <span class="n">conn_state</span><span class="p">]</span> <span class="o">:</span> <span class="n">conns</span><span class="p">)</span> <span class="p">{</span>
            <span class="n">send_message</span><span class="p">(</span><span class="n">conn_state</span><span class="p">.</span><span class="n">conn</span><span class="p">,</span> <span class="n">msg</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">};</span>

</code></pre>

</div>






<h2>
  
  
  Building the HTTP Server
</h2>

<p>TCP streams are just sequences of bytes. An HTTP request might be <strong>fragmented across multiple TCP packets</strong>, so the server must:</p>

<ol>
<li>Reassemble the byte stream.</li>
<li>Extract request headers.</li>
<li>Parse the body (if present).</li>
<li>Handle limits (max body size, header size).</li>
</ol>

<p>The HTTP server integrates tightly with the <strong>socket library</strong>, that is, it extends the <strong>hh_socket::epoll_server</strong> functionality by reusing its efficient connection handling and abstraction. This shows how layering simplifies complexity: the HTTP server focuses on protocol logic, while sockets manage the low-level networking.</p>

<p><strong>Note that my implementation is not fully compliant with the HTTP specification, it just provides a basic framework for handling HTTP requests.</strong></p>

<h4>
  
  
  High-level (use the project's parser):
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// Example: receive a buffer from a connection and let the project's parser assemble</span>
<span class="c1">// requests that may span multiple TCP reads.</span>
<span class="n">hh_socket</span><span class="o">::</span><span class="n">data_buffer</span> <span class="n">buf</span> <span class="o">=</span> <span class="n">conn</span><span class="o">-&gt;</span><span class="n">receive</span><span class="p">();</span>
<span class="n">hh_http</span><span class="o">::</span><span class="n">http_message_handler</span> <span class="n">parser</span><span class="p">;</span>

<span class="c1">// `handle` returns an http_handled_data describing either a complete request</span>
<span class="c1">// or that more bytes are required (completed == false).</span>
<span class="k">auto</span> <span class="n">result</span> <span class="o">=</span> <span class="n">parser</span><span class="p">.</span><span class="n">handle</span><span class="p">(</span><span class="n">conn</span><span class="p">,</span> <span class="n">buf</span><span class="p">);</span>
<span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="n">result</span><span class="p">.</span><span class="n">completed</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Not enough data yet - wait for the next read and call parser.handle again</span>
<span class="p">}</span> <span class="k">else</span> <span class="nf">if</span> <span class="p">(</span><span class="n">result</span><span class="p">.</span><span class="n">method</span><span class="p">.</span><span class="n">rfind</span><span class="p">(</span><span class="s">"BAD_"</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span> <span class="o">==</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Parser returned an error token (e.g. BAD_METHOD_OR_URI_OR_VERSION)</span>
    <span class="c1">// Application can craft an error response here</span>
<span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
    <span class="c1">// Complete request: use result.method, result.uri, result.headers, result.body</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  Low-level (manual reassembly sketch):
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// Read bytes into a string buffer until we detect the header terminator \r\n\r\n</span>
<span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">buffer</span> <span class="o">=</span> <span class="n">conn</span><span class="o">-&gt;</span><span class="n">receive</span><span class="p">().</span><span class="n">to_string</span><span class="p">();</span>
<span class="kt">size_t</span> <span class="n">headers_end</span> <span class="o">=</span> <span class="n">buffer</span><span class="p">.</span><span class="n">find</span><span class="p">(</span><span class="s">"</span><span class="se">\r\n\r\n</span><span class="s">"</span><span class="p">);</span>
<span class="k">if</span> <span class="p">(</span><span class="n">headers_end</span> <span class="o">!=</span> <span class="n">std</span><span class="o">::</span><span class="n">string</span><span class="o">::</span><span class="n">npos</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">header_block</span> <span class="o">=</span> <span class="n">buffer</span><span class="p">.</span><span class="n">substr</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="n">headers_end</span><span class="p">);</span>
    <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">rest</span> <span class="o">=</span> <span class="n">buffer</span><span class="p">.</span><span class="n">substr</span><span class="p">(</span><span class="n">headers_end</span> <span class="o">+</span> <span class="mi">4</span><span class="p">);</span> <span class="c1">// body (maybe partial)</span>

    <span class="c1">// parse request-line and headers (split on \r\n, then on ':')</span>
    <span class="c1">// find Content-Length (if present) to determine expected body size</span>
    <span class="kt">size_t</span> <span class="n">content_length</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="c1">// parse from header_block if present</span>

    <span class="c1">// Keep reading until we have the full body</span>
    <span class="k">while</span> <span class="p">(</span><span class="n">rest</span><span class="p">.</span><span class="n">size</span><span class="p">()</span> <span class="o">&lt;</span> <span class="n">content_length</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">rest</span> <span class="o">+=</span> <span class="n">conn</span><span class="o">-&gt;</span><span class="n">receive</span><span class="p">().</span><span class="n">to_string</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">body</span> <span class="o">=</span> <span class="n">rest</span><span class="p">.</span><span class="n">substr</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="n">content_length</span><span class="p">);</span>
    <span class="c1">// Now header_block contains headers and `body` contains the full payload</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  HTTP Server example (using my http_server class)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">"http_server.hpp"</span><span class="cp">
</span>
<span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">using</span> <span class="k">namespace</span> <span class="n">hh_http</span><span class="p">;</span>

    <span class="c1">// this automatically sets up the server to listen for incoming connections</span>
    <span class="c1">// also this handles big request body, and also, you can normally send a big response,</span>
    <span class="c1">// as the epoll server itself handles sending such big chunks of data</span>
    <span class="n">http_server</span> <span class="n">server</span><span class="p">(</span><span class="mi">8081</span><span class="p">,</span> <span class="s">"0.0.0.0"</span><span class="p">);</span>

    <span class="c1">// Set up all the callbacks</span>
    <span class="n">server</span><span class="p">.</span><span class="n">set_request_callback</span><span class="p">([](</span> <span class="n">http_request</span> <span class="o">&amp;</span><span class="n">req</span> <span class="p">,</span> <span class="n">http_response</span> <span class="o">&amp;</span><span class="n">res</span> <span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Handle incoming HTTP requests</span>
        <span class="n">res</span><span class="p">.</span><span class="n">set_body</span><span class="p">(</span><span class="s">"Hello, World!</span><span class="se">\n</span><span class="s">"</span><span class="p">);</span>
        <span class="n">res</span><span class="p">.</span><span class="n">set_status</span><span class="p">(</span><span class="mi">200</span><span class="p">,</span> <span class="s">"OK"</span><span class="p">);</span>
        <span class="n">res</span><span class="p">.</span><span class="n">send</span><span class="p">();</span> <span class="c1">// send the headers, and the body to the client</span>
        <span class="n">res</span><span class="p">.</span><span class="n">end</span><span class="p">();</span>
    <span class="p">});</span>
    <span class="n">server</span><span class="p">.</span><span class="n">set_error_callback</span><span class="p">([](</span><span class="k">const</span> <span class="n">std</span><span class="o">::</span><span class="n">exception</span> <span class="o">&amp;</span><span class="n">e</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">std</span><span class="o">::</span><span class="n">cerr</span> <span class="o">&lt;&lt;</span> <span class="s">"Error: "</span> <span class="o">&lt;&lt;</span> <span class="n">e</span><span class="p">.</span><span class="n">what</span><span class="p">()</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
    <span class="p">});</span>

    <span class="c1">// Start the server (this will block)</span>
    <span class="n">server</span><span class="p">.</span><span class="n">listen</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  The Web Library Layer
</h2>

<p>The top layer provides <strong>routing, controllers, and static file serving</strong>, similar to Express.js. Key features:</p>

<ul>
<li>
<strong>MVC-like architecture</strong> – Organizes code into controllers, views, and models for better maintainability.</li>
<li>
<strong>Routing system</strong> – Maps incoming HTTP requests to controller actions.</li>
<li>
<strong>Static file serving</strong> – Delivers HTML, CSS, and JavaScript assets alongside dynamic content.</li>
</ul>

<p><strong>Simple Example server:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">"web-lib.hpp"</span><span class="cp">
</span>
<span class="c1">// Define request handlers (controller actions)</span>
<span class="n">hh_web</span><span class="o">::</span><span class="n">exit_code</span> <span class="nf">get_users_handler</span><span class="p">(</span>
    <span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">hh_web</span><span class="o">::</span><span class="n">web_request</span><span class="o">&gt;</span> <span class="n">req</span><span class="p">,</span>
    <span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">hh_web</span><span class="o">::</span><span class="n">web_response</span><span class="o">&gt;</span> <span class="n">res</span><span class="p">)</span> <span class="p">{</span>

    <span class="n">res</span><span class="o">-&gt;</span><span class="n">send_json</span><span class="p">(</span><span class="s">"{</span><span class="se">\"</span><span class="s">users</span><span class="se">\"</span><span class="s">: [</span><span class="se">\"</span><span class="s">Alice</span><span class="se">\"</span><span class="s">, </span><span class="se">\"</span><span class="s">Bob</span><span class="se">\"</span><span class="s">]}"</span><span class="p">);</span>
    <span class="k">return</span> <span class="n">hh_web</span><span class="o">::</span><span class="n">exit_code</span><span class="o">::</span><span class="n">EXIT</span><span class="p">;</span>
<span class="p">}</span>

<span class="n">hh_web</span><span class="o">::</span><span class="n">exit_code</span> <span class="nf">create_user_handler</span><span class="p">(</span>
    <span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">hh_web</span><span class="o">::</span><span class="n">web_request</span><span class="o">&gt;</span> <span class="n">req</span><span class="p">,</span>
    <span class="n">std</span><span class="o">::</span><span class="n">shared_ptr</span><span class="o">&lt;</span><span class="n">hh_web</span><span class="o">::</span><span class="n">web_response</span><span class="o">&gt;</span> <span class="n">res</span><span class="p">)</span> <span class="p">{</span>

    <span class="c1">// Extract user data from request body</span>
    <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">body</span> <span class="o">=</span> <span class="n">req</span><span class="o">-&gt;</span><span class="n">get_body</span><span class="p">();</span>
    <span class="n">res</span><span class="o">-&gt;</span><span class="n">send_json</span><span class="p">(</span><span class="s">"{</span><span class="se">\"</span><span class="s">success</span><span class="se">\"</span><span class="s">: true, </span><span class="se">\"</span><span class="s">message</span><span class="se">\"</span><span class="s">: </span><span class="se">\"</span><span class="s">User created</span><span class="se">\"</span><span class="s">}"</span><span class="p">);</span>
    <span class="k">return</span> <span class="n">hh_web</span><span class="o">::</span><span class="n">exit_code</span><span class="o">::</span><span class="n">EXIT</span><span class="p">;</span>
<span class="p">}</span>

<span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>

    <span class="k">auto</span> <span class="n">server</span> <span class="o">=</span> <span class="n">std</span><span class="o">::</span><span class="n">make_shared</span><span class="o">&lt;</span><span class="n">hh_web</span><span class="o">::</span><span class="n">web_server</span><span class="o">&lt;&gt;&gt;</span><span class="p">(</span><span class="mi">8080</span><span class="p">,</span> <span class="s">"0.0.0.0"</span><span class="p">);</span>

    <span class="k">auto</span> <span class="n">router</span> <span class="o">=</span> <span class="n">std</span><span class="o">::</span><span class="n">make_shared</span><span class="o">&lt;</span><span class="n">hh_web</span><span class="o">::</span><span class="n">web_router</span><span class="o">&lt;&gt;&gt;</span><span class="p">();</span>

    <span class="c1">// Map routes to controller actions</span>
    <span class="k">using</span> <span class="n">V</span> <span class="o">=</span> <span class="n">std</span><span class="o">::</span><span class="n">vector</span><span class="o">&lt;</span><span class="n">hh_web</span><span class="o">::</span><span class="n">web_request_handler_t</span><span class="o">&lt;&gt;&gt;</span><span class="p">;</span>

    <span class="n">router</span><span class="o">-&gt;</span><span class="n">get</span><span class="p">(</span><span class="s">"/api/users"</span><span class="p">,</span> <span class="n">V</span><span class="p">{</span><span class="n">get_users_handler</span><span class="p">});</span>
    <span class="n">router</span><span class="o">-&gt;</span><span class="n">post</span><span class="p">(</span><span class="s">"/api/users"</span><span class="p">,</span> <span class="n">V</span><span class="p">{</span><span class="n">create_user_handler</span><span class="p">});</span>

    <span class="c1">// It auto-detects the static files (based on the extention, then sends it)</span>
    <span class="n">server</span><span class="o">-&gt;</span><span class="n">use_static</span><span class="p">(</span><span class="s">"static"</span><span class="p">);</span>

    <span class="c1">// Route with path parameters</span>
    <span class="n">router</span><span class="o">-&gt;</span><span class="n">get</span><span class="p">(</span><span class="s">"/api/users/:id"</span><span class="p">,</span> <span class="n">V</span><span class="p">{[](</span><span class="k">auto</span> <span class="n">req</span><span class="p">,</span> <span class="k">auto</span> <span class="n">res</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">hh_web</span><span class="o">::</span><span class="n">exit_code</span> <span class="p">{</span>
        <span class="k">auto</span> <span class="n">params</span> <span class="o">=</span> <span class="n">req</span><span class="o">-&gt;</span><span class="n">get_path_params</span><span class="p">();</span>
        <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">user_id</span> <span class="o">=</span> <span class="n">params</span><span class="p">.</span><span class="n">at</span><span class="p">(</span><span class="s">"id"</span><span class="p">);</span>
        <span class="n">res</span><span class="o">-&gt;</span><span class="n">send_json</span><span class="p">(</span><span class="s">"{</span><span class="se">\"</span><span class="s">user_id</span><span class="se">\"</span><span class="s">: </span><span class="se">\"</span><span class="s">"</span> <span class="o">+</span> <span class="n">user_id</span> <span class="o">+</span> <span class="s">"</span><span class="se">\"</span><span class="s">}"</span><span class="p">);</span>
        <span class="k">return</span> <span class="n">hh_web</span><span class="o">::</span><span class="n">exit_code</span><span class="o">::</span><span class="n">EXIT</span><span class="p">;</span>
    <span class="p">}});</span>

    <span class="n">server</span><span class="o">-&gt;</span><span class="n">use_router</span><span class="p">(</span><span class="n">router</span><span class="p">);</span>
    <span class="n">server</span><span class="o">-&gt;</span><span class="n">listen</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Why This Structure Matters
</h2>

<p>Here’s what this layered design teaches:</p>

<ul>
<li>
<strong>Sockets</strong> handle raw communication and events efficiently.</li>
<li>
<strong>HTTP server</strong> interprets protocol-level data reliably from TCP streams.</li>
<li>
<strong>Web library</strong> allows developers to structure their application cleanly and add features without worrying about low-level details.</li>
</ul>




<p>Even after a short time learning backend programming, this project clarified:</p>

<ul>
<li>Networking can be simplified to <strong>basic operations</strong> and abstractions.</li>
<li>
<strong>TCP streams need careful parsing</strong>, not just reading packets.</li>
<li>Layering responsibilities makes large systems manageable and testable.</li>
</ul>

