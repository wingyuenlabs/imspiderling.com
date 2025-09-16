---
Title: Building MCP Servers: Understanding Transport Layers and Core Components
Description: 
Author: Tandap Noel Bansikah
Date: 2025-09-16T21:20:26.000Z
Robots: noindex,nofollow
Template: index
---
<p>In my <a href="https://dev.to/bansikah/mcp-basics-understanding-the-model-context-protocol-566g">previous article</a>, we explored what the Model Context Protocol (MCP) is and why it's revolutionizing how AI models interact with external systems. Now that we understand MCP's potential, let's dive deep into the implementation details—how to actually build MCP servers that are robust, scalable, and secure.</p>

<p>Building effective MCP servers requires mastering two key areas: the transport mechanisms that enable communication between clients and servers, and the core components (tools, resources, and prompts) that define your server's functionality. Let's explore each in detail.</p>

<h2>
  
  
  Transport Mechanisms: How MCP Connects
</h2>

<p>Think of transport mechanisms as the "delivery methods" for messages between AI models and your MCP server. Just like choosing how to send a package—hand delivery for urgent local items, postal service for standard shipping, or express courier for time-sensitive deliveries—MCP offers different transport options optimized for specific scenarios.</p>

<p>Imagine you're running a restaurant (your MCP server) and customers (AI models) want to place orders. You could take orders through:</p>

<ul>
<li>
<strong>Direct conversation</strong> (STDIO) - fastest, but only one customer at a time</li>
<li>
<strong>Phone calls</strong> (HTTP) - can handle multiple customers, works remotely</li>
<li>
<strong>Live updates on a screen</strong> (SSE) - great for showing order status in real-time</li>
</ul>

<p>The transport layer handles all the communication logistics—message formatting, delivery confirmation, error handling—so you can focus on cooking (your server's business logic) rather than managing the ordering system.</p>

<h3>
  
  
  STDIO Transport: Direct Process Communication
</h3>

<p>STDIO transport is like having a private, dedicated phone line between the AI model and your server. When Claude Desktop wants to use your MCP server, it's like hiring a personal assistant (spawning your server process) who sits right next to them and communicates through simple notes passed back and forth.</p>

<p>Think of it as the difference between shouting across a crowded room versus having a quiet, one-on-one conversation. There's no network noise, no interference from other conversations, and no delay—just direct, immediate communication. This is why STDIO is perfect for desktop applications where speed and reliability matter most.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Server setup with STDIO transport</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Server</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@modelcontextprotocol/sdk/server/index.js</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">StdioServerTransport</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@modelcontextprotocol/sdk/server/stdio.js</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">server</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Server</span><span class="p">({</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">my-mcp-server</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">version</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1.0.0</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">transport</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">StdioServerTransport</span><span class="p">();</span>
<span class="k">await</span> <span class="nx">server</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">transport</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Technical Details:</strong></p>

<ul>
<li>
<strong>Protocol</strong>: JSON-RPC over stdin/stdout streams</li>
<li>
<strong>Process Model</strong>: Client spawns server as child process</li>
<li>
<strong>Latency</strong>: Ultra-low (~1ms) - no network overhead</li>
<li>
<strong>Concurrency</strong>: Single client per server instance</li>
<li>
<strong>Connection</strong>: Automatic cleanup when client terminates</li>
</ul>

<p><strong>When to Use STDIO:</strong></p>

<ul>
<li>Building tools for Claude Desktop or similar local AI apps</li>
<li>Creating development utilities that need fast response times</li>
<li>When you want simple deployment (just an executable file)</li>
<li>For personal productivity tools on your local machine</li>
</ul>

<p><strong>Example Scenario:</strong> You're building a code analysis tool for developers. Using STDIO transport means Claude Desktop can spawn your server instantly when needed, analyze code with minimal delay, and automatically clean up when done.</p>

<h3>
  
  
  HTTP Transport: Web-Scale Communication
</h3>

<p>HTTP transport is like turning your MCP server into a popular restaurant that takes phone orders. Instead of having one dedicated assistant (STDIO), you now have a phone system that can handle multiple customers calling simultaneously from anywhere in the world.</p>

<p>Just like a restaurant can serve customers who call from different locations, HTTP transport allows multiple AI clients to connect to your server over the internet. You can set up security (like requiring a password to place orders), handle busy periods with load balancing (multiple phone lines), and even serve customers in different time zones. The trade-off is slightly slower service due to the "phone call overhead," but you gain the ability to serve many more customers.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// HTTP server implementation</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Server</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@modelcontextprotocol/sdk/server/index.js</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HttpServerTransport</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@modelcontextprotocol/sdk/server/http.js</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">server</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Server</span><span class="p">({</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">web-mcp-server</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">version</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1.0.0</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">transport</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">HttpServerTransport</span><span class="p">({</span>
  <span class="na">port</span><span class="p">:</span> <span class="mi">3000</span><span class="p">,</span>
  <span class="na">host</span><span class="p">:</span> <span class="dl">'</span><span class="s1">localhost</span><span class="dl">'</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">server</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">transport</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Technical Details:</strong></p>

<ul>
<li>
<strong>Protocol</strong>: JSON-RPC over HTTP/HTTPS</li>
<li>
<strong>Connection Model</strong>: Stateless request-response</li>
<li>
<strong>Latency</strong>: Network-dependent (typically 10-100ms+ depending on distance)</li>
<li>
<strong>Concurrency</strong>: Handles multiple simultaneous clients</li>
<li>
<strong>Scaling</strong>: Can use load balancers, reverse proxies, clustering</li>
</ul>

<p><strong>When to Use HTTP:</strong></p>

<ul>
<li>
<strong>Remote Access</strong>: Server and client are on different machines</li>
<li>
<strong>Web Integration</strong>: Embedding MCP functionality in web applications</li>
<li>
<strong>Team Sharing</strong>: Multiple developers need access to the same server</li>
<li>
<strong>Cloud Deployment</strong>: Running servers on AWS, Google Cloud, etc.</li>
<li>
<strong>Production Systems</strong>: Need monitoring, logging, and enterprise features</li>
</ul>

<p><strong>Example Scenario:</strong> Your team needs a shared database query tool. Deploy it as an HTTP MCP server on your cloud infrastructure, and multiple team members can access it remotely through their AI clients. Add authentication headers to control access and use HTTPS for secure communication.</p>

<h3>
  
  
  Server-Sent Events (SSE): Real-Time Streaming
</h3>

<p>SSE transport is like having a live sports commentator giving play-by-play updates. Instead of the AI having to repeatedly ask "Are you done yet?" (like checking order status every few minutes), your server can actively broadcast updates as things happen.</p>

<p>Imagine you're processing a large dataset—with SSE, you can send updates like "10% complete... 25% complete... found interesting pattern... 75% complete..." This keeps the AI informed and engaged, rather than leaving it waiting in silence. It's perfect for long-running tasks where you want to show progress, like a pizza tracker showing when your order is being prepared, baked, and out for delivery.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// SSE transport for real-time updates</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">SseServerTransport</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@modelcontextprotocol/sdk/server/sse.js</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">transport</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SseServerTransport</span><span class="p">({</span>
  <span class="na">endpoint</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/events</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">port</span><span class="p">:</span> <span class="mi">3000</span>
<span class="p">});</span>

<span class="c1">// Stream real-time updates</span>
<span class="nx">transport</span><span class="p">.</span><span class="nf">sendEvent</span><span class="p">({</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">progress</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">data</span><span class="p">:</span> <span class="p">{</span> <span class="na">completed</span><span class="p">:</span> <span class="mi">75</span><span class="p">,</span> <span class="na">total</span><span class="p">:</span> <span class="mi">100</span> <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Technical Details:</strong></p>

<ul>
<li>
<strong>Protocol</strong>: Server-Sent Events over HTTP</li>
<li>
<strong>Connection Model</strong>: Long-lived, server-to-client streaming</li>
<li>
<strong>Latency</strong>: Near real-time (typically &lt;100ms for event delivery)</li>
<li>
<strong>Concurrency</strong>: Multiple clients can subscribe to event streams</li>
<li>
<strong>Reliability</strong>: Built-in reconnection and event ID tracking</li>
</ul>

<p><strong>When to Use SSE:</strong></p>

<ul>
<li>
<strong>Long Operations</strong>: File processing, data analysis, model training</li>
<li>
<strong>Live Updates</strong>: Real-time dashboards, monitoring systems</li>
<li>
<strong>Progress Tracking</strong>: Show completion status for multi-step workflows</li>
<li>
<strong>Event Notifications</strong>: Alert clients about system changes or updates</li>
</ul>

<p><strong>Example Scenario:</strong> You're building an AI-powered data processing pipeline. Use SSE to stream progress updates as files are processed, allowing the AI client to provide real-time feedback to users and make decisions based on intermediate results.</p>

<h2>
  
  
  Transport Selection Guide
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Transport</th>
<th>Latency</th>
<th>Scalability</th>
<th>Deployment</th>
<th>Security</th>
<th>Best For</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>STDIO</strong></td>
<td>~1ms</td>
<td>Single client</td>
<td>Executable file</td>
<td>Process isolation</td>
<td>Local tools, Claude Desktop</td>
</tr>
<tr>
<td><strong>HTTP</strong></td>
<td>10-100ms+</td>
<td>High (load balancing)</td>
<td>Web server + networking</td>
<td>HTTPS + auth headers</td>
<td>Remote access, web apps</td>
</tr>
<tr>
<td><strong>SSE</strong></td>
<td>&lt;100ms</td>
<td>Medium (connection limits)</td>
<td>Web server + event handling</td>
<td>HTTP security + validation</td>
<td>Real-time updates, monitoring</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Decision Framework
</h2>

<p><strong>Choose STDIO when:</strong></p>

<ul>
<li>Your server will run on the same machine as the AI client</li>
<li>You need the fastest possible response times</li>
<li>You want simple deployment (single executable)</li>
<li>Building for Claude Desktop or similar local applications</li>
</ul>

<p><strong>Choose HTTP when:</strong></p>

<ul>
<li>You need remote access (server and client on different machines)</li>
<li>Multiple clients need to use the same server simultaneously
</li>
<li>You're integrating with web applications or cloud services</li>
<li>You need enterprise features like load balancing and monitoring</li>
</ul>

<p><strong>Choose SSE when:</strong></p>

<ul>
<li>Your operations take significant time and need progress updates</li>
<li>You're building real-time monitoring or dashboard systems</li>
<li>You need to push notifications or alerts to clients</li>
<li>Your workflow involves streaming data or continuous updates</li>
</ul>

<h1>
  
  
  Core Components: The Building Blocks
</h1>

<p>Think of MCP servers like a Swiss Army knife for AI models. Just as a Swiss Army knife has different tools for different tasks, MCP servers have three types of components that give AI models different capabilities. Understanding these components is like learning what each tool in the knife does and when to use it.</p>

<h2>
  
  
  Tools: Executable Actions
</h2>

<p>Tools are like the power tools in a workshop—they're what allow the AI to actually <em>do</em> things in the real world. Just as a carpenter has different tools for different jobs (saw for cutting, drill for holes, sander for smoothing), your MCP server provides different tools for different actions.</p>

<p>Think of tools as the AI's "hands"—they extend the AI's capabilities beyond just thinking and talking. Want to search through files? There's a tool for that. Need to send an email? Another tool. Process an image? Yet another tool.</p>

<p>The magic happens when you design tools like a good craftsperson organizes their workshop: each tool has a clear purpose, is reliable when used correctly, and comes with clear instructions (documentation) so even a novice can use it safely and effectively.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Tool definition with comprehensive validation</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">setRequestHandler</span><span class="p">(</span><span class="nx">ListToolsRequestSchema</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">({</span>
  <span class="na">tools</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">search_files</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Search for files using glob patterns</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">inputSchema</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">object</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">properties</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">pattern</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Glob pattern (e.g., "*.ts", "**/*.json")</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">pattern</span><span class="p">:</span> <span class="dl">'</span><span class="s1">^[^/</span><span class="se">\\\\</span><span class="s1">:*?"&lt;&gt;|]+$</span><span class="dl">'</span> <span class="c1">// Security: prevent path traversal</span>
          <span class="p">},</span>
          <span class="na">maxResults</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">number</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">minimum</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
            <span class="na">maximum</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span>
            <span class="na">default</span><span class="p">:</span> <span class="mi">50</span>
          <span class="p">}</span>
        <span class="p">},</span>
        <span class="na">required</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">pattern</span><span class="dl">'</span><span class="p">]</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}));</span>

<span class="c1">// Tool execution with error handling</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">setRequestHandler</span><span class="p">(</span><span class="nx">CallToolRequestSchema</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">name</span><span class="p">,</span> <span class="na">arguments</span><span class="p">:</span> <span class="nx">args</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="k">switch </span><span class="p">(</span><span class="nx">name</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">case</span> <span class="dl">'</span><span class="s1">search_files</span><span class="dl">'</span><span class="p">:</span>
        <span class="k">return</span> <span class="k">await</span> <span class="nf">handleSearchFiles</span><span class="p">(</span><span class="nf">validateArgs</span><span class="p">(</span><span class="nx">args</span><span class="p">));</span>
      <span class="nl">default</span><span class="p">:</span>
        <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Unknown tool: </span><span class="p">${</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">content</span><span class="p">:</span> <span class="p">[{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">text</span><span class="p">:</span> <span class="s2">`Error: </span><span class="p">${</span><span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">}</span><span class="s2">`</span>
      <span class="p">}],</span>
      <span class="na">isError</span><span class="p">:</span> <span class="kc">true</span>
    <span class="p">};</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Tool Design Principles:</strong></p>

<p>1  <strong>Single Responsibility</strong>: Each tool should do one thing well</p>

<ul>
<li>✅ Good: <code>search_files</code> - searches for files using patterns</li>
<li>❌ Bad: <code>file_manager</code> - searches, creates, deletes, and modifies files</li>
</ul>

<p>2  <strong>Input Validation</strong>: Always validate and sanitize inputs<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// Validate file patterns to prevent security issues</span>
   <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="sr">/^</span><span class="se">[</span><span class="sr">a-zA-Z0-9*.</span><span class="se">\-</span><span class="sr">_</span><span class="se">/]</span><span class="sr">+$/</span><span class="p">.</span><span class="nf">test</span><span class="p">(</span><span class="nx">pattern</span><span class="p">))</span> <span class="p">{</span>
     <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Invalid pattern: contains unsafe characters</span><span class="dl">'</span><span class="p">);</span>
   <span class="p">}</span>
</code></pre>

</div>



<p>3  <strong>Error Handling</strong>: Provide clear, actionable error messages<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// Instead of: "Error occurred"</span>
   <span class="c1">// Use: "File not found: /path/to/file.txt. Check the path and permissions."</span>
</code></pre>

</div>



<p>4  <strong>Rich Documentation</strong>: Help AI models understand when and how to use tools<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="nx">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Search for files using glob patterns. Use * for wildcards, ** for recursive search. Example: "**/*.js" finds all JavaScript files recursively.</span><span class="dl">'</span>
</code></pre>

</div>



<p><strong>Technical Implementation Details:</strong></p>

<ul>
<li>Tools use JSON Schema for input validation, providing type safety and automatic validation</li>
<li>The <code>inputSchema</code> property defines exactly what parameters the tool accepts</li>
<li>Return values should always include a <code>content</code> array with structured responses</li>
<li>Use the <code>isError</code> flag to indicate when operations fail</li>
</ul>

<h2>
  
  
  Resources: Accessible Data
</h2>

<p>Resources are like a well-organized library that the AI can browse and read from. Just as a library has books, magazines, DVDs, and digital resources all organized with a clear cataloging system (like the Dewey Decimal System), MCP resources use URI patterns to organize different types of information.</p>

<p>Think of resources as the AI's "reference materials." When a student needs to write a research paper, they don't just need tools (pen, computer, printer)—they need access to information (books, articles, databases). Similarly, AI models need both tools to take actions AND resources to understand context and make informed decisions.</p>

<p>The URI system works like library call numbers: <code>file://documents/readme.md</code> is like saying "go to the Documents section, find the README file," while <code>api://users/123/profile</code> means "check the User Records section, look up person #123's profile." This consistent addressing system helps the AI quickly find exactly what it needs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Resource listing with metadata</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">setRequestHandler</span><span class="p">(</span><span class="nx">ListResourcesRequestSchema</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">({</span>
  <span class="na">resources</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">uri</span><span class="p">:</span> <span class="dl">'</span><span class="s1">file://documents/readme.md</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Project README</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Main project documentation</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text/markdown</span><span class="dl">'</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">uri</span><span class="p">:</span> <span class="dl">'</span><span class="s1">api://users/profile</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">User Profile</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Current user profile data</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span>
    <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}));</span>

<span class="c1">// Resource reading with content negotiation</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">setRequestHandler</span><span class="p">(</span><span class="nx">ReadResourceRequestSchema</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">uri</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">uri</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">file://</span><span class="dl">'</span><span class="p">))</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">filePath</span> <span class="o">=</span> <span class="nx">uri</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">7</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">content</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">fs</span><span class="p">.</span><span class="nf">readFile</span><span class="p">(</span><span class="nx">filePath</span><span class="p">,</span> <span class="dl">'</span><span class="s1">utf8</span><span class="dl">'</span><span class="p">);</span>

    <span class="k">return</span> <span class="p">{</span>
      <span class="na">contents</span><span class="p">:</span> <span class="p">[{</span>
        <span class="nx">uri</span><span class="p">,</span>
        <span class="na">mimeType</span><span class="p">:</span> <span class="nx">mime</span><span class="p">.</span><span class="nf">lookup</span><span class="p">(</span><span class="nx">filePath</span><span class="p">)</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">text/plain</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">text</span><span class="p">:</span> <span class="nx">content</span>
      <span class="p">}]</span>
    <span class="p">};</span>
  <span class="p">}</span>

  <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Unsupported resource URI: </span><span class="p">${</span><span class="nx">uri</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Resource Design Patterns:</strong></p>

<p>1  <strong>URI Schemes</strong>: Create consistent, hierarchical patterns<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// File system resources</span>
   <span class="dl">'</span><span class="s1">file://documents/readme.md</span><span class="dl">'</span>
   <span class="dl">'</span><span class="s1">file://logs/2024/january/app.log</span><span class="dl">'</span>

   <span class="c1">// API resources  </span>
   <span class="dl">'</span><span class="s1">api://users/123/profile</span><span class="dl">'</span>
   <span class="dl">'</span><span class="s1">api://orders/456/items</span><span class="dl">'</span>

   <span class="c1">// Database resources</span>
   <span class="dl">'</span><span class="s1">db://customers/active</span><span class="dl">'</span>
   <span class="dl">'</span><span class="s1">db://products/category/electronics</span><span class="dl">'</span>
</code></pre>

</div>



<p>2  <strong>MIME Types</strong>: Enable proper content handling<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// Let AI clients know what type of data they're getting</span>
   <span class="nx">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span>     <span class="c1">// Structured data</span>
   <span class="nx">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text/markdown</span><span class="dl">'</span>        <span class="c1">// Documentation</span>
   <span class="nx">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text/csv</span><span class="dl">'</span>            <span class="c1">// Tabular data</span>
   <span class="nx">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">image/png</span><span class="dl">'</span>           <span class="c1">// Binary content</span>
</code></pre>

</div>



<p>3  <strong>Caching Strategy</strong>: Implement efficient updates<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// Use ETags to avoid re-reading unchanged resources</span>
   <span class="kd">const</span> <span class="nx">etag</span> <span class="o">=</span> <span class="nf">generateETag</span><span class="p">(</span><span class="nx">content</span><span class="p">);</span>
   <span class="k">if </span><span class="p">(</span><span class="nx">request</span><span class="p">.</span><span class="nx">headers</span><span class="p">[</span><span class="dl">'</span><span class="s1">if-none-match</span><span class="dl">'</span><span class="p">]</span> <span class="o">===</span> <span class="nx">etag</span><span class="p">)</span> <span class="p">{</span>
     <span class="k">return</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="mi">304</span> <span class="p">};</span> <span class="c1">// Not modified</span>
   <span class="p">}</span>
</code></pre>

</div>



<p>4  <strong>Security Considerations</strong>: Always validate access<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// Prevent path traversal attacks</span>
   <span class="kd">const</span> <span class="nx">safePath</span> <span class="o">=</span> <span class="nx">path</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="nx">SAFE_ROOT</span><span class="p">,</span> <span class="nx">requestedPath</span><span class="p">);</span>
   <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">safePath</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="nx">SAFE_ROOT</span><span class="p">))</span> <span class="p">{</span>
     <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Access denied: path outside allowed directory</span><span class="dl">'</span><span class="p">);</span>
   <span class="p">}</span>
</code></pre>

</div>



<p><strong>Technical Implementation:</strong></p>

<ul>
<li>Resources are read-only from the AI client's perspective</li>
<li>Use the <code>ListResourcesRequestSchema</code> to advertise available resources</li>
<li>Implement <code>ReadResourceRequestSchema</code> to serve resource content</li>
<li>Support content negotiation through MIME types for optimal AI processing</li>
</ul>

<h2>
  
  
  Prompts: Conversation Templates
</h2>

<p>Prompts are like having a collection of conversation scripts or templates that help guide interactions. Think of them as the "Mad Libs" of AI conversations—you create a template with blanks to fill in, and the AI can use these templates to start productive conversations.</p>

<p>Imagine you're a manager who frequently needs to conduct different types of meetings. Instead of improvising each time, you might have templates: "Performance Review Template," "Project Kickoff Template," "Problem-Solving Template." Each template has a structure and key questions, but you fill in the specific details for each situation.</p>

<p>MCP prompts work the same way—they provide proven conversation patterns that AI models can use as starting points, customized with specific context for each situation.</p>

<p><strong>Technical Architecture:</strong></p>

<ul>
<li>Prompts use template variables for dynamic content generation</li>
<li>They return complete conversation contexts with role-based messages</li>
<li>Parameters enable customization without changing the core template</li>
<li>The AI client invokes prompts to establish structured conversation flows</li>
</ul>

<p><strong>When to Use Prompts:</strong></p>

<ul>
<li>
<strong>Standardizing Workflows</strong>: Create consistent approaches to common tasks</li>
<li>
<strong>Domain Expertise</strong>: Embed specialized knowledge into conversation starters</li>
<li>
<strong>Complex Processes</strong>: Guide AI through multi-step procedures</li>
<li>
<strong>Quality Control</strong>: Ensure AI interactions follow best practices
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Prompt definition with variables</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">setRequestHandler</span><span class="p">(</span><span class="nx">ListPromptsRequestSchema</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">({</span>
  <span class="na">prompts</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">code_review</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Generate code review comments</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">arguments</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">language</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Programming language</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">required</span><span class="p">:</span> <span class="kc">true</span>
        <span class="p">},</span>
        <span class="p">{</span>
          <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">focus_areas</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Areas to focus on (security, performance, etc.)</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">required</span><span class="p">:</span> <span class="kc">false</span>
        <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}));</span>

<span class="c1">// Prompt rendering with context injection</span>
<span class="nx">server</span><span class="p">.</span><span class="nf">setRequestHandler</span><span class="p">(</span><span class="nx">GetPromptRequestSchema</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">name</span><span class="p">,</span> <span class="na">arguments</span><span class="p">:</span> <span class="nx">args</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">name</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">code_review</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">language</span> <span class="o">=</span> <span class="nx">args</span><span class="p">?.</span><span class="nx">language</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">javascript</span><span class="dl">'</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">focusAreas</span> <span class="o">=</span> <span class="nx">args</span><span class="p">?.</span><span class="nx">focus_areas</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">general best practices</span><span class="dl">'</span><span class="p">;</span>

    <span class="k">return</span> <span class="p">{</span>
      <span class="na">description</span><span class="p">:</span> <span class="s2">`Code review for </span><span class="p">${</span><span class="nx">language</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="na">messages</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="na">role</span><span class="p">:</span> <span class="dl">'</span><span class="s1">user</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">content</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span>
            <span class="na">text</span><span class="p">:</span> <span class="s2">`Please review this </span><span class="p">${</span><span class="nx">language</span><span class="p">}</span><span class="s2"> code focusing on </span><span class="p">${</span><span class="nx">focusAreas</span><span class="p">}</span><span class="s2">. 
                   Provide specific, actionable feedback with examples.`</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">};</span>
  <span class="p">}</span>

  <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Unknown prompt: </span><span class="p">${</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Integration Patterns: Making Components Work Together
</h2>

<p>The real power of MCP comes from how these components work together, like instruments in an orchestra. Each component has its role, but the magic happens when they're coordinated to create something greater than the sum of their parts.</p>

<h3>
  
  
  Tool-Resource Synergy
</h3>

<p>Think of the relationship between tools and resources like a detective and evidence. The detective (AI) uses tools (investigation methods) to gather and analyze resources (evidence, witness statements, documents). Sometimes using a tool creates new evidence that becomes a resource for further investigation.</p>

<p>Tools and resources work together to create powerful workflows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Tool that creates resources dynamically</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">handleAnalyzeCode</span><span class="p">(</span><span class="nx">args</span><span class="p">:</span> <span class="p">{</span> <span class="nl">filePath</span><span class="p">:</span> <span class="kr">string</span> <span class="p">})</span> <span class="p">{</span>
  <span class="c1">// Tool action: analyze the code</span>
  <span class="kd">const</span> <span class="nx">analysis</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">analyzeCodeFile</span><span class="p">(</span><span class="nx">args</span><span class="p">.</span><span class="nx">filePath</span><span class="p">);</span>

  <span class="c1">// Create a resource with results</span>
  <span class="kd">const</span> <span class="nx">resourceUri</span> <span class="o">=</span> <span class="s2">`analysis://reports/</span><span class="p">${</span><span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()}</span><span class="s2">`</span><span class="p">;</span>
  <span class="nx">resourceCache</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">resourceUri</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">content</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">analysis</span><span class="p">,</span> <span class="kc">null</span><span class="p">,</span> <span class="mi">2</span><span class="p">),</span>
    <span class="na">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">timestamp</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">()</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">content</span><span class="p">:</span> <span class="p">[{</span>
      <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">text</span><span class="p">:</span> <span class="s2">`Analysis complete. View results at: </span><span class="p">${</span><span class="nx">resourceUri</span><span class="p">}</span><span class="s2">`</span>
    <span class="p">}]</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Real-World Example:</strong><br>
Imagine building a code review assistant. The AI uses the <code>analyze_code</code> tool to examine a file, which creates an analysis resource. Then it uses the <code>read_resource</code> capability to access the detailed analysis, and finally uses a <code>generate_report</code> tool to create a formatted review document. Each step builds on the previous one, creating a powerful workflow.</p>
<h3>
  
  
  Prompt-Driven Workflows
</h3>

<p>Prompts act like a GPS for complex tasks—they provide step-by-step directions to help the AI navigate from problem to solution. Just as GPS breaks down a long journey into manageable turns ("In 500 feet, turn left, then continue straight for 2 miles"), prompts break down complex workflows into clear, actionable steps.</p>

<p>Prompts can guide AI models through complex multi-step processes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Multi-step workflow prompt</span>
<span class="kd">const</span> <span class="nx">workflowPrompt</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">debug_issue</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">arguments</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">error_message</span><span class="dl">'</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">true</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">component</span><span class="dl">'</span><span class="p">,</span> <span class="na">required</span><span class="p">:</span> <span class="kc">false</span> <span class="p">}</span>
  <span class="p">],</span>
  <span class="na">messages</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">role</span><span class="p">:</span> <span class="dl">'</span><span class="s1">system</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">content</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">text</span><span class="p">:</span> <span class="dl">'</span><span class="s1">You are a debugging expert. Follow systematic troubleshooting procedures.</span><span class="dl">'</span>
      <span class="p">}</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">role</span><span class="p">:</span> <span class="dl">'</span><span class="s1">user</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">content</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">text</span><span class="p">:</span> <span class="s2">`Debug this </span><span class="p">${</span><span class="nx">component</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">system</span><span class="dl">'</span><span class="p">}</span><span class="s2"> issue: "</span><span class="p">${</span><span class="nx">error_message</span><span class="p">}</span><span class="s2">"

               Follow this systematic approach:
               1. Use search_files to find relevant code files
               2. Use read_file to examine the problematic code
               3. Use get_logs to check for related errors
               4. Use check_dependencies to verify system state
               5. Provide a diagnosis with specific fix recommendations`</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">]</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>Practical Application:</strong><br>
When a developer encounters a bug, they can invoke the <code>debug_issue</code> prompt with the error message. The AI follows the structured workflow, systematically using different tools to investigate, and provides a comprehensive diagnosis. This ensures consistent, thorough debugging every time.</p>
<h2>
  
  
  Performance and Security: Building for the Real World
</h2>

<p>Building an MCP server is like constructing a building—you need both a solid foundation (security) and efficient systems (performance) to handle real-world usage. Just as architects consider both structural integrity and energy efficiency, MCP developers must balance security and performance.</p>
<h3>
  
  
  Performance Optimization
</h3>

<p>Performance optimization is like organizing your kitchen for efficiency. Just as a chef keeps frequently used ingredients within easy reach and pre-prepares common components, your MCP server should cache expensive operations and optimize for common use patterns.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Implement caching for expensive operations</span>
<span class="kd">const</span> <span class="nx">cache</span> <span class="o">=</span> <span class="k">new</span> <span class="nb">Map</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="kr">any</span><span class="p">;</span> <span class="nl">timestamp</span><span class="p">:</span> <span class="kr">number</span> <span class="p">}</span><span class="o">&gt;</span><span class="p">();</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">cachedOperation</span><span class="p">(</span><span class="nx">key</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">operation</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">cached</span> <span class="o">=</span> <span class="nx">cache</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">key</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">now</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">cached</span> <span class="o">&amp;&amp;</span> <span class="p">(</span><span class="nx">now</span> <span class="o">-</span> <span class="nx">cached</span><span class="p">.</span><span class="nx">timestamp</span><span class="p">)</span> <span class="o">&lt;</span> <span class="mi">300000</span><span class="p">)</span> <span class="p">{</span> <span class="c1">// 5 min cache</span>
    <span class="k">return</span> <span class="nx">cached</span><span class="p">.</span><span class="nx">data</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">operation</span><span class="p">();</span>
  <span class="nx">cache</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">key</span><span class="p">,</span> <span class="p">{</span> <span class="nx">data</span><span class="p">,</span> <span class="na">timestamp</span><span class="p">:</span> <span class="nx">now</span> <span class="p">});</span>
  <span class="k">return</span> <span class="nx">data</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Security Best Practices
</h2>

<p>Security in MCP servers is like being a careful bouncer at an exclusive club. You need to check IDs (validate inputs), ensure people don't wander into restricted areas (prevent path traversal), and maintain order while still providing good service to legitimate guests.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Input validation and sanitization</span>
<span class="kd">function</span> <span class="nf">validateFilePath</span><span class="p">(</span><span class="nx">path</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="kr">string</span> <span class="p">{</span>
  <span class="c1">// Prevent path traversal</span>
  <span class="kd">const</span> <span class="nx">normalized</span> <span class="o">=</span> <span class="nx">path</span><span class="p">.</span><span class="nf">normalize</span><span class="p">(</span><span class="nx">path</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">normalized</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="dl">'</span><span class="s1">..</span><span class="dl">'</span><span class="p">)</span> <span class="o">||</span> <span class="nx">normalized</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">/</span><span class="dl">'</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Invalid path: path traversal detected</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Ensure path is within allowed directory</span>
  <span class="kd">const</span> <span class="nx">resolved</span> <span class="o">=</span> <span class="nx">path</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="nx">ALLOWED_ROOT</span><span class="p">,</span> <span class="nx">normalized</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">resolved</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="nx">ALLOWED_ROOT</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Access denied: path outside allowed directory</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">resolved</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Putting It All Together: A Practical Example
</h2>

<p>Let's walk through building a complete MCP server for a development team's needs:</p>

<p><strong>Scenario</strong>: Your team needs an AI assistant that can help with code reviews, manage project documentation, and monitor system health.</p>

<p><strong>Transport Decision</strong>: </p>

<ul>
<li>Use <strong>HTTP transport</strong> because multiple developers need access</li>
<li>Deploy on your internal cloud infrastructure</li>
<li>Enable HTTPS with team authentication</li>
</ul>

<p><strong>Component Design</strong>:</p>

<p>1  <strong>Tools</strong> (Actions the AI can perform):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// Code analysis</span>
   <span class="dl">'</span><span class="s1">analyze_code</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">examines</span> <span class="nx">code</span> <span class="nx">quality</span> <span class="nx">and</span> <span class="nx">security</span>
   <span class="dl">'</span><span class="s1">run_tests</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">executes</span> <span class="nx">test</span> <span class="nx">suites</span>
   <span class="dl">'</span><span class="s1">deploy_staging</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">deploys</span> <span class="nx">to</span> <span class="nx">staging</span> <span class="nx">environment</span>

   <span class="c1">// Documentation</span>
   <span class="dl">'</span><span class="s1">update_docs</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">modifies</span> <span class="nx">project</span> <span class="nx">documentation</span>
   <span class="dl">'</span><span class="s1">generate_changelog</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">creates</span> <span class="nx">release</span> <span class="nx">notes</span>

   <span class="c1">// Monitoring  </span>
   <span class="dl">'</span><span class="s1">check_system_health</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">monitors</span> <span class="nx">server</span> <span class="nx">status</span>
   <span class="dl">'</span><span class="s1">get_error_logs</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">retrieves</span> <span class="nx">recent</span> <span class="nx">error</span> <span class="nx">logs</span>
</code></pre>

</div>



<p>2  <strong>Resources</strong> (Information the AI can access):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="c1">// Code and documentation</span>
   <span class="dl">'</span><span class="s1">file://src/**/*.ts</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">source</span> <span class="nx">code</span> <span class="nx">files</span>
   <span class="dl">'</span><span class="s1">file://docs/**/*.md</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">documentation</span> <span class="nx">files</span>

   <span class="c1">// System data</span>
   <span class="dl">'</span><span class="s1">api://monitoring/metrics</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">performance</span> <span class="nx">metrics</span>
   <span class="dl">'</span><span class="s1">api://logs/errors/recent</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">recent</span> <span class="nx">error</span> <span class="nx">logs</span>
   <span class="dl">'</span><span class="s1">db://deployments/history</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">deployment</span> <span class="nx">history</span>
</code></pre>

</div>



<p>3  <strong>Prompts</strong> (Structured workflows):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>   <span class="dl">'</span><span class="s1">code_review_checklist</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">systematic</span> <span class="nx">code</span> <span class="nx">review</span> <span class="nx">process</span>
   <span class="dl">'</span><span class="s1">incident_response</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">structured</span> <span class="nx">incident</span> <span class="nx">handling</span>
   <span class="dl">'</span><span class="s1">release_preparation</span><span class="dl">'</span> <span class="o">-</span> <span class="nx">pre</span><span class="o">-</span><span class="nx">release</span> <span class="nx">validation</span> <span class="nx">steps</span>
</code></pre>

</div>



<p><strong>Result</strong>: Developers can ask the AI to "review this pull request" (uses code_review_checklist prompt), "check if the system is healthy" (uses monitoring tools and resources), or "prepare for release" (follows release_preparation workflow).</p>

<h2>
  
  
  Conclusion
</h2>

<p>Building effective MCP servers is like designing a well-equipped workshop—you need the right tools for the job, organized materials you can easily find, and proven procedures for complex tasks. </p>

<p><strong>Key Takeaways:</strong></p>

<ul>
<li>
<strong>Transport choice matters</strong>: STDIO for local speed, HTTP for remote access, SSE for real-time updates</li>
<li>
<strong>Design for clarity</strong>: Each tool should have a single, clear purpose</li>
<li>
<strong>Organize systematically</strong>: Use consistent URI patterns for resources</li>
<li>
<strong>Guide with prompts</strong>: Provide structured workflows for complex tasks</li>
<li>
<strong>Secure by default</strong>: Always validate inputs and control access</li>
<li>
<strong>Optimize for real use</strong>: Cache expensive operations and monitor performance</li>
</ul>

<p>Start simple with a few essential tools and resources, then expand based on actual usage patterns. The best MCP servers solve real problems elegantly rather than trying to do everything at once.</p>

<p><a href="https://github.com/iinsys/ai-mcp" rel="noopener noreferrer">Visit this repository to learn more about MCP servers</a></p>

