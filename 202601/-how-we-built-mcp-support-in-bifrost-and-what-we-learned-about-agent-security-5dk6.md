---
Title: How We Built MCP Support in Bifrost (And What We Learned About Agent Security)
Description: 
Author: Pranay Batta
Date: 2026-01-19T21:27:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>When we started building MCP support for Bifrost, I thought it would be straightforward. Connect to MCP servers, proxy tool calls, done. Turns out, making this production-ready meant solving problems that the MCP spec doesn't even address.</p>


<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fassets.dev.to%2Fassets%2Fgithub-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/maximhq" rel="noopener noreferrer">
        maximhq
      </a> / <a href="https://github.com/maximhq/bifrost" rel="noopener noreferrer">
        bifrost
      </a>
    </h2>
    <h3>
      Fastest LLM gateway (50x faster than LiteLLM) with adaptive load balancer, cluster mode, guardrails, 1000+ models support &amp; &lt;100 µs overhead at 5k RPS.
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">Bifrost</h1>
</div>
<p><a href="https://goreportcard.com/report/github.com/maximhq/bifrost/core" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/7f7e70df9fdaaf4f485f59ca6bc0b5cbbf134d03dd5721da4e31f90f618fc304/68747470733a2f2f676f7265706f7274636172642e636f6d2f62616467652f6769746875622e636f6d2f6d6178696d68712f626966726f73742f636f7265" alt="Go Report Card"></a>
<a href="https://discord.gg/exN5KAydbU" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/282b7719f04b28f5959f5e1e17aee806d65f8eea3b862b57af350df0ab57be6f/68747470733a2f2f646362616467652e6c696d65732e70696e6b2f6170692f7365727665722f68747470733a2f2f646973636f72642e67672f65784e354b41796462553f7374796c653d666c6174" alt="Discord badge"></a>
<a href="https://snyk.io/test/github/maximhq/bifrost" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/fb28496133f724daf03a8107e56978a14f6f2ed7e7283df0573747fa46ff8f86/68747470733a2f2f736e796b2e696f2f746573742f6769746875622f6d6178696d68712f626966726f73742f62616467652e737667" alt="Known Vulnerabilities"></a>
<a href="https://codecov.io/gh/maximhq/bifrost" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/8bc2db302c566210d14c09b278639a3f63f07def5fc635a8869e59c996b3100f/68747470733a2f2f636f6465636f762e696f2f67682f6d6178696d68712f626966726f73742f6272616e63682f6d61696e2f67726170682f62616467652e737667" alt="codecov"></a>
<a rel="noopener noreferrer nofollow" href="https://camo.githubusercontent.com/b0899925aadfed8626116707178a4015d8cf4aaa0b80acb632cb4782c6dc7272/68747470733a2f2f696d672e736869656c64732e696f2f646f636b65722f70756c6c732f6d6178696d68712f626966726f7374"><img src="https://camo.githubusercontent.com/b0899925aadfed8626116707178a4015d8cf4aaa0b80acb632cb4782c6dc7272/68747470733a2f2f696d672e736869656c64732e696f2f646f636b65722f70756c6c732f6d6178696d68712f626966726f7374" alt="Docker Pulls"></a>
<a href="https://app.getpostman.com/run-collection/31642484-2ba0e658-4dcd-49f4-845a-0c7ed745b916?action=collection%2Ffork&amp;source=rip_markdown&amp;collection-url=entityId%3D31642484-2ba0e658-4dcd-49f4-845a-0c7ed745b916%26entityType%3Dcollection%26workspaceId%3D63e853c8-9aec-477f-909c-7f02f543150e" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/82ccefddb001e2caf9d399f1153fdda561cf3da341bb270e18644d516906bc64/68747470733a2f2f72756e2e7073746d6e2e696f2f627574746f6e2e737667" alt="Run In Postman"></a>
<a href="https://artifacthub.io/packages/search?repo=bifrost" rel="nofollow noopener noreferrer"><img src="https://camo.githubusercontent.com/a6a3c734d6bd57fa8e1d508ac0cdba555bdbcd9191b29b32cf37a964b86b9c67/68747470733a2f2f696d672e736869656c64732e696f2f656e64706f696e743f75726c3d68747470733a2f2f61727469666163746875622e696f2f62616467652f7265706f7369746f72792f626966726f7374" alt="Artifact Hub"></a>
<a href="https://github.com/maximhq/bifrost/LICENSE" rel="noopener noreferrer"><img src="https://camo.githubusercontent.com/3cb44c15a532770a066ba8e61bf11506ad5400e5c61d48f6b639101e442bee79/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f6c6963656e73652f6d6178696d68712f626966726f7374" alt="License"></a></p>
<div class="markdown-heading">
<h2 class="heading-element">The fastest way to build AI applications that never go down</h2>
</div>
<p>Bifrost is a high-performance AI gateway that unifies access to 15+ providers (OpenAI, Anthropic, AWS Bedrock, Google Vertex, and more) through a single OpenAI-compatible API. Deploy in seconds with zero configuration and get automatic failover, load balancing, semantic caching, and enterprise-grade features.</p>
<div class="markdown-heading">
<h2 class="heading-element">Quick Start</h2>
</div>
<p><a rel="noopener noreferrer" href="https://github.com/maximhq/bifrost/./docs/media/getting-started.png"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2Fmaximhq%2Fbifrost%2F.%2Fdocs%2Fmedia%2Fgetting-started.png" alt="Get started"></a></p>
<p><strong>Go from zero to production-ready AI gateway in under a minute.</strong></p>
<p><strong>Step 1:</strong> Start Bifrost Gateway</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre><span class="pl-c"><span class="pl-c">#</span> Install and run locally</span>
npx -y @maximhq/bifrost

<span class="pl-c"><span class="pl-c">#</span> Or use Docker</span>
docker run -p 8080:8080 maximhq/bifrost</pre>

</div>
<p><strong>Step 2:</strong> Configure via Web UI</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre><span class="pl-c"><span class="pl-c">#</span> Open the built-in web interface</span>
open http://localhost:8080</pre>

</div>
<p><strong>Step 3:</strong> Make your first API call</p>
<div class="highlight highlight-source-shell notranslate position-relative overflow-auto js-code-highlight">
<pre>curl -X POST http://localhost:8080/v1/chat/completions \
  -H <span class="pl-s"><span class="pl-pds">"</span>Content-Type: application/json<span class="pl-pds">"</span></span> \
  -d <span class="pl-s"><span class="pl-pds">'</span>{</span>
<span class="pl-s">    "model": "openai/gpt-4o-mini",</span>
<span class="pl-s">    "messages": [{"role": "user", "content": "Hello, Bifrost!"}]</span>
<span class="pl-s">  }<span class="pl-pds">'</span></span></pre>

</div>
<p><strong>That's it!</strong> Your AI gateway is running with a web interface for visual configuration, real-time monitoring…</p>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/maximhq/bifrost" rel="noopener noreferrer">View on GitHub</a></div>
</div>


<p>AI agents hit a hard limit when you try to connect them to real infrastructure. You want your agent to query a database, read files, or call an API? You're writing custom integration code for every single use case.</p>

<p>Model Context Protocol changes this. It's a standard way for AI models to discover and use external tools at runtime. Instead of hardcoding integrations, you spin up MCP servers that expose tools - and the AI model just figures out what's available and uses it.</p>

<p>But when we looked at how teams were actually deploying MCP in production, we saw three big problems:</p>

<p><strong>Security disasters waiting to happen.</strong> Most implementations just let the AI model execute any tool it wants. No oversight. No logging. One bad prompt and you're updating production databases or deleting files.</p>

<p><strong>Zero observability.</strong> When something breaks, you have no idea which tool was called, what parameters were used, or what failed. Debugging becomes archaeology.</p>

<p><strong>Operational complexity.</strong> Managing connections to dozens of MCP servers, handling failures, controlling access - none of this is solved by the protocol itself.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv9w2w524ybex714nvkw3.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv9w2w524ybex714nvkw3.gif" alt="boom" width="500" height="376"></a></p>

<p>That's why we built MCP support directly into Bifrost. It's not just a client - it's a complete control plane for production agent infrastructure.</p>

<h2>
  
  
  Four Connection Types
</h2>

<p>We support four ways to connect to MCP servers, and each one solves different problems:</p>

<h3>
  
  
  InProcess Connections
</h3>

<p>These run tools directly in Bifrost's memory. If you're writing tools in Go, you register them with typed handlers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">CalculatorArgs</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Operation</span> <span class="kt">string</span>  <span class="s">`json:"operation"`</span>
    <span class="n">A</span>         <span class="kt">float64</span> <span class="s">`json:"a"`</span>
    <span class="n">B</span>         <span class="kt">float64</span> <span class="s">`json:"b"`</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">calculatorHandler</span><span class="p">(</span><span class="n">args</span> <span class="n">CalculatorArgs</span><span class="p">)</span> <span class="p">(</span><span class="kt">string</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">switch</span> <span class="n">args</span><span class="o">.</span><span class="n">Operation</span> <span class="p">{</span>
    <span class="k">case</span> <span class="s">"add"</span><span class="o">:</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"%.2f"</span><span class="p">,</span> <span class="n">args</span><span class="o">.</span><span class="n">A</span> <span class="o">+</span> <span class="n">args</span><span class="o">.</span><span class="n">B</span><span class="p">),</span> <span class="no">nil</span>
    <span class="k">case</span> <span class="s">"multiply"</span><span class="o">:</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"%.2f"</span><span class="p">,</span> <span class="n">args</span><span class="o">.</span><span class="n">A</span> <span class="o">*</span> <span class="n">args</span><span class="o">.</span><span class="n">B</span><span class="p">),</span> <span class="no">nil</span>
    <span class="k">default</span><span class="o">:</span>
        <span class="k">return</span> <span class="s">""</span><span class="p">,</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"unsupported operation"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="n">client</span><span class="o">.</span><span class="n">RegisterMCPTool</span><span class="p">(</span><span class="s">"calculator"</span><span class="p">,</span> <span class="s">"Perform arithmetic"</span><span class="p">,</span> <span class="n">calculatorHandler</span><span class="p">,</span> <span class="n">schema</span><span class="p">)</span>
</code></pre>

</div>


<p>Latency is around 0.1ms. No network overhead. Compile-time type checking catches bugs before runtime. This is perfect for internal business logic that doesn't need to run in a separate process.</p>
<h3>
  
  
  STDIO Connections
</h3>

<p>These launch external processes and talk via stdin/stdout. Great for local tools, scripts, or when you're using Python/Node.js MCP servers. Latency is typically 1-10ms, which is still way faster than network calls.</p>

<p>We use this for filesystem operations. The agent can read files, list directories, or search code - all through a Node.js MCP server running locally.</p>
<h3>
  
  
  HTTP Connections
</h3>

<p>These talk to remote MCP servers over HTTP. Latency depends on your network, usually 10-500ms. But you get scalability - run multiple instances behind a load balancer, handle failures gracefully, isolate tool execution from the gateway.</p>

<p>Our database tools run this way. Microservice with proper authentication, read-only access by default, connection pooling for performance.</p>
<h3>
  
  
  SSE Connections
</h3>

<p>Server-Sent Events for real-time data. Maintains a persistent connection and streams updates. Perfect for monitoring tools, live dashboards, or anything that needs event-driven updates without constant polling.</p>
<h2>
  
  
  The Security Model That Actually Works
</h2>

<p>Here's the thing about MCP that nobody talks about: by default, letting an AI model execute tools automatically is insane.</p>

<p>Bifrost's security model is simple - tool calls are suggestions, not commands. When an AI model wants to use a tool, it returns the request to your application. You decide whether to execute it. This gives you human oversight for anything dangerous.</p>

<p>But we also support "agent mode" where you can whitelist specific tools for automatic execution. The configuration looks like this:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">ToolManagerConfig</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">schemas</span><span class="o">.</span><span class="n">MCPToolManagerConfig</span><span class="p">{</span>
    <span class="n">ToolsToAutoExecute</span><span class="o">:</span> <span class="p">[]</span><span class="kt">string</span><span class="p">{</span>
        <span class="s">"filesystem/read_file"</span><span class="p">,</span>
        <span class="s">"database/query_readonly"</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="n">ToolExecutionTimeout</span><span class="o">:</span> <span class="m">30</span> <span class="o">*</span> <span class="n">time</span><span class="o">.</span><span class="n">Second</span><span class="p">,</span>
<span class="p">}</span>
</code></pre>

</div>


<p>Only the tools you explicitly approve run automatically. Everything else requires manual approval. This prevents disasters while keeping agents fast for safe operations.</p>
<h2>
  
  
  Request-Level Filtering
</h2>

<p>This is where things get interesting for production deployments. You can filter which tools are available per request:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8080/v1/chat/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"x-bf-mcp-include-clients: secure-database,audit-logger"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"x-bf-mcp-include-tools: secure-database/*,audit-logger/log_access"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"model": "gpt-4o-mini", "messages": [...]}'</span>
</code></pre>

</div>


<p>The agent only sees the tools you specify. Customer-facing chatbots don't get access to admin tools. Financial applications can restrict agents to read-only operations. You can even use wildcards - <code>database/*</code> includes all database tools.</p>

<p>This solves the privilege escalation problem that makes a lot of security teams nervous about AI agents.</p>
<h2>
  
  
  Code Mode vs Agent Mode
</h2>

<p>We built two execution patterns because different use cases need different approaches:</p>

<p><strong>Agent mode</strong> has the AI call one tool at a time. It sees the result, thinks, calls the next tool. Great for interactive agents where you want visibility at each step.</p>

<p><strong>Code mode</strong> lets the AI write TypeScript that orchestrates multiple tools:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">files</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">listFiles</span><span class="p">(</span><span class="dl">"</span><span class="s2">/project</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">all</span><span class="p">(</span>
    <span class="nx">files</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">file</span> <span class="o">=&gt;</span> <span class="nf">analyzeFile</span><span class="p">(</span><span class="nx">file</span><span class="p">))</span>
<span class="p">);</span>
<span class="k">return</span> <span class="nf">summarize</span><span class="p">(</span><span class="nx">results</span><span class="p">);</span>
</code></pre>

</div>


<p>The code runs atomically. Way faster for batch operations. Lower latency because you're not doing multiple LLM round-trips. But you lose per-tool approval - the code either runs or it doesn't.</p>

<p>We use code mode for data analysis workflows where the agent needs to read dozens of files, run queries, and generate reports. Agent mode for customer support where we want to review database queries before they execute.</p>
<h2>
  
  
  What Surprised Us
</h2>

<p><strong>Tool discovery is faster than we expected.</strong> Getting the list of 50+ tools from an MCP server takes under 100ms. The AI model sees available tools instantly.</p>

<p><strong>STDIO latency is surprisingly low.</strong> We thought process communication would be slow, but 1-2ms overhead is barely noticeable. HTTP adds way more latency from network round-trips.</p>

<p><strong>Type safety prevents so many bugs.</strong> InProcess tools with Go structs caught issues at compile time that would have been runtime failures with JSON validation.</p>

<p><strong>Request filtering is more powerful than global config.</strong> Being able to change which tools are available per request gives you way more control than static configuration.</p>
<h2>
  
  
  Performance In Production
</h2>

<p>We're running this ourselves for internal tooling. Some numbers from real usage:</p>

<ul>
<li>STDIO filesystem tools: 1.2ms average latency</li>
<li>HTTP database tools: 15ms average (local network)</li>
<li>InProcess calculation tools: 0.08ms average</li>
<li>Memory per STDIO connection: ~2MB</li>
<li>Tool discovery for 50 tools: 85ms</li>
</ul>

<p>These are with Go's runtime. Python and Node.js MCP servers will be slower, but the architecture scales well.</p>
<h2>
  
  
  Try It Yourself
</h2>

<p><a href="https://docs.getbifrost.ai/mcp/overview" rel="noopener noreferrer">MCP support</a> is live in Bifrost. The setup is straightforward - configure your MCP servers, set which tools can auto-execute, and start making requests.</p>


<div class="crayons-card c-embed text-styles text-styles--secondary">
    <div class="c-embed__content">
        <div class="c-embed__cover">
          <a href="https://docs.getbifrost.ai/mcp/overview" class="c-link align-middle" rel="noopener noreferrer">
            <img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fbifrost.mintlify.app%2Fmintlify-assets%2F_next%2Fimage%3Furl%3D%252F_mintlify%252Fapi%252Fog%253Fdivision%253DMCP%252BGateway%2526title%253DOverview%2526description%253DEnable%252BAI%252Bmodels%252Bto%252Bdiscover%252Band%252Bexecute%252Bexternal%252Btools%252Bdynamically.%252BTransform%252Bstatic%252Bchat%252Bmodels%252Binto%252Baction-capable%252Bagents.%2526logoLight%253Dhttps%25253A%25252F%25252Fmintcdn.com%25252Fbifrost%25252FqFMmk8bNSnvgFYDI%25252Fmedia%25252Fbifrost-logo.png%25253Ffit%25253Dmax%252526auto%25253Dformat%252526n%25253DqFMmk8bNSnvgFYDI%252526q%25253D85%252526s%25253D6af701a560aee4103444fa017b226cf0%2526logoDark%253Dhttps%25253A%25252F%25252Fmintcdn.com%25252Fbifrost%25252FqFMmk8bNSnvgFYDI%25252Fmedia%25252Fbifrost-logo-dark.png%25253Ffit%25253Dmax%252526auto%25253Dformat%252526n%25253DqFMmk8bNSnvgFYDI%252526q%25253D85%252526s%25253D84d264aad5f421c526dd17893e7c5739%2526primaryColor%253D%2525230C3B43%2526lightColor%253D%25252307C983%2526backgroundLight%253D%252523ffffff%2526backgroundDark%253D%252523090d0d%26w%3D1200%26q%3D100" height="630" class="m-0" width="1200">
          </a>
        </div>
      <div class="c-embed__body">
        <h2 class="fs-xl lh-tight">
          <a href="https://docs.getbifrost.ai/mcp/overview" rel="noopener noreferrer" class="c-link">
            Overview - Bifrost
          </a>
        </h2>
          <p class="truncate-at-3">
            Enable AI models to discover and execute external tools dynamically. Transform static chat models into action-capable agents.
          </p>
        <div class="color-secondary fs-s flex items-center">
            <img alt="favicon" class="c-embed__favicon m-0 mr-2 radius-0" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdocs.getbifrost.ai%2Fmintlify-assets%2F_mintlify%2Ffavicons%2Fbifrost%2FkXIL-Qs6IzN4ZLK4%2F_generated%2Ffavicon%2Ffavicon-16x16.png" width="16" height="16">
          docs.getbifrost.ai
        </div>
      </div>
    </div>
</div>



<p>We documented all four connection types, agent vs code mode, and request-level filtering in the <a href="https://docs.getbifrost.ai/features/mcp" rel="noopener noreferrer">MCP docs</a>. There are examples for common patterns - filesystem, database, web APIs.</p>

<p>If you're building agents that need to interact with real infrastructure, this makes it way cleaner than custom integration code. And you get the observability and security controls you need for production.</p>

<p>The code is open source - check out the implementation if you're curious how the internals work. We're shipping updates based on real-world usage, so feedback is helpful.</p>

