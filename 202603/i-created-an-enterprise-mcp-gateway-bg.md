---
Title: I Created An Enterprise MCP Gateway
Description: 
Author: Anthony Max
Date: 2026-03-03T21:52:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>When you start building AI applications beyond simple experiments, everything changes. Models need access to files, databases, APIs, and internal services. That's where the Model Context Protocol (MCP) comes in.</p>

<p>But managing dozens of MCP servers, tools, and integrations in production quickly becomes a nightmare. I spent the last few months building an enterprise MCP gateway using <strong>Bifrost</strong>, and I want to share what I learned.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe0rfn7esopb4tw586o43.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe0rfn7esopb4tw586o43.png" alt="Intro" width="" height=""></a></p>




<h2>
  
  
  💻 The Problem: MCP Without a Gateway is Bad
</h2>

<p>Here's what happens without proper infrastructure:</p>

<p>Your models spend precious tokens discovering available tools. Teams can't control who uses what. An engineer accidentally deletes the wrong database because the model had access it shouldn't have. API costs spike unexpectedly. You have no idea which AI workflows are running where.</p>

<p>The root issue: <strong>MCP was designed for flexibility</strong>. When you scale from a chatbot to production AI systems, you need:</p>

<ul>
<li>
<strong>Centralized tool management</strong> instead of scattered MCP servers</li>
<li>
<strong>Fine-grained access control</strong> so marketing tools don't leak into engineering</li>
<li>
<strong>Rate limiting per tool</strong> to prevent API abuse and runaway costs</li>
<li>
<strong>Complete audit trails</strong> for compliance and debugging</li>
</ul>




<h2>
  
  
  👀 Why Bifrost?
</h2>

<p>Bifrost is a high-performance, Go-based LLM gateway that solves these problems:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Quick start - 30 seconds with -p 8000</span>
npx <span class="nt">-y</span> @maximhq/bifrost

<span class="c"># Opens http://localhost:8000</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr7ja1wpn4u48gwhnti66.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr7ja1wpn4u48gwhnti66.webp" alt="interface" width="800" height="389"></a></p>

<p><a href="https://git.new/bifrost" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">💎 Star Bifrost ☆</a>
</p>

<p>Key advantages:</p>

<ul>
<li>
<strong>40x lower overhead</strong> than another gateways (11µs vs 440µs)</li>
<li><strong>68% less memory usage</strong></li>
<li>
<strong>100% success rate</strong> at 5,000 RPS</li>
<li>
<strong>Code Mode</strong> - Models generate orchestration code instead of step-by-step calls</li>
<li>
<strong>Semantic caching</strong> - 40-60% cost reduction on similar queries</li>
<li>
<strong>Built-in control</strong> - RBAC, rate limiting, cost tracking, audit logs</li>
</ul>




<h2>
  
  
  📦 1. Collect All MCP Servers
</h2>

<p>Instead of direct model access to scattered MCP servers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Gateway configuration - single entry point</span>
<span class="n">mcpConfig</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">schemas</span><span class="o">.</span><span class="n">MCPConfig</span><span class="p">{</span>
    <span class="n">ClientConfigs</span><span class="o">:</span> <span class="p">[]</span><span class="n">schemas</span><span class="o">.</span><span class="n">MCPClientConfig</span><span class="p">{</span>
        <span class="p">{</span>
            <span class="n">Name</span><span class="o">:</span>           <span class="s">"filesystem"</span><span class="p">,</span>
            <span class="n">ConnectionType</span><span class="o">:</span> <span class="n">schemas</span><span class="o">.</span><span class="n">MCPConnectionTypeSTDIO</span><span class="p">,</span>
            <span class="n">StdioConfig</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">schemas</span><span class="o">.</span><span class="n">MCPStdioConfig</span><span class="p">{</span>
                <span class="n">Command</span><span class="o">:</span> <span class="s">"npx"</span><span class="p">,</span>
                <span class="n">Args</span><span class="o">:</span>    <span class="p">[]</span><span class="kt">string</span><span class="p">{</span><span class="s">"-y"</span><span class="p">,</span> <span class="s">"@anthropic/mcp-filesystem"</span><span class="p">},</span>
            <span class="p">},</span>
            <span class="n">ToolsToExecute</span><span class="o">:</span> <span class="p">[]</span><span class="kt">string</span><span class="p">{</span><span class="s">"*"</span><span class="p">},</span>
        <span class="p">},</span>
        <span class="p">{</span>
            <span class="n">Name</span><span class="o">:</span>             <span class="s">"web_search"</span><span class="p">,</span>
            <span class="n">ConnectionType</span><span class="o">:</span>   <span class="n">schemas</span><span class="o">.</span><span class="n">MCPConnectionTypeHTTP</span><span class="p">,</span>
            <span class="n">ConnectionString</span><span class="o">:</span> <span class="n">bifrost</span><span class="o">.</span><span class="n">Ptr</span><span class="p">(</span><span class="s">"http://localhost:3001/mcp"</span><span class="p">),</span>
            <span class="n">ToolsToExecute</span><span class="o">:</span>   <span class="p">[]</span><span class="kt">string</span><span class="p">{</span><span class="s">"search"</span><span class="p">,</span> <span class="s">"fetch_url"</span><span class="p">},</span>
        <span class="p">},</span>
    <span class="p">},</span>
<span class="p">}</span>

<span class="n">client</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">bifrost</span><span class="o">.</span><span class="n">Init</span><span class="p">(</span><span class="n">context</span><span class="o">.</span><span class="n">Background</span><span class="p">(),</span> <span class="n">schemas</span><span class="o">.</span><span class="n">BifrostConfig</span><span class="p">{</span>
    <span class="n">Account</span><span class="o">:</span>   <span class="n">account</span><span class="p">,</span>
    <span class="n">MCPConfig</span><span class="o">:</span> <span class="n">mcpConfig</span><span class="p">,</span>
    <span class="n">Logger</span><span class="o">:</span>    <span class="n">bifrost</span><span class="o">.</span><span class="n">NewDefaultLogger</span><span class="p">(</span><span class="n">schemas</span><span class="o">.</span><span class="n">LogLevelInfo</span><span class="p">),</span>
<span class="p">})</span>
</code></pre>

</div>



<p><strong>Benefits:</strong></p>

<ul>
<li>Single source of truth for all tools</li>
<li>Unified security policies</li>
<li>Centralized monitoring and cost tracking</li>
<li>Consistent behavior across all models</li>
</ul>




<h2>
  
  
  ⚙️ 2. Control Tool Access Based on Roles
</h2>

<p>Different teams need different tool access levels. Implement role-based access control:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">roleToToolsMapping</span> <span class="o">:=</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">][]</span><span class="kt">string</span><span class="p">{</span>
    <span class="s">"engineering"</span><span class="o">:</span> <span class="p">{</span><span class="s">"filesystem"</span><span class="p">,</span> <span class="s">"database"</span><span class="p">,</span> <span class="s">"github-api"</span><span class="p">},</span>
    <span class="s">"marketing"</span><span class="o">:</span>   <span class="p">{</span><span class="s">"web-search"</span><span class="p">,</span> <span class="s">"document-generation"</span><span class="p">},</span>
    <span class="s">"finance"</span><span class="o">:</span>     <span class="p">{</span><span class="s">"cost-tracking"</span><span class="p">},</span>
    <span class="s">"admin"</span><span class="o">:</span>       <span class="p">{</span><span class="s">"*"</span><span class="p">},</span>  <span class="c">// All tools</span>
<span class="p">}</span>

<span class="n">roleLimits</span> <span class="o">:=</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="kt">int</span><span class="p">{</span>
    <span class="s">"engineering"</span><span class="o">:</span> <span class="p">{</span><span class="s">"filesystem"</span><span class="o">:</span> <span class="m">1000</span><span class="p">,</span> <span class="s">"database"</span><span class="o">:</span> <span class="m">500</span><span class="p">},</span>
    <span class="s">"marketing"</span><span class="o">:</span>   <span class="p">{</span><span class="s">"web_search"</span><span class="o">:</span> <span class="m">100</span><span class="p">},</span>
    <span class="s">"finance"</span><span class="o">:</span>     <span class="p">{</span><span class="s">"cost_tracking"</span><span class="o">:</span> <span class="m">50</span><span class="p">},</span>
<span class="p">}</span>

<span class="c">// Check access</span>
<span class="n">async</span> <span class="n">function</span> <span class="n">checkToolAccess</span><span class="p">(</span><span class="n">userId</span><span class="p">,</span> <span class="n">role</span><span class="p">,</span> <span class="n">toolName</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">const</span> <span class="n">allowedTools</span> <span class="o">=</span> <span class="n">roleToToolsMapping</span><span class="p">[</span><span class="n">role</span><span class="p">];</span>
  <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="n">allowedTools</span><span class="o">.</span><span class="n">includes</span><span class="p">(</span><span class="n">toolName</span><span class="p">))</span> <span class="p">{</span>
    <span class="n">throw</span> <span class="nb">new</span> <span class="n">Error</span><span class="p">(</span><span class="s">`Tool '${toolName}' is denied for role '${role}'`</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Real example - Access denied:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/v1/mcp/tool/execute <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "tool_call": {
      "tool_name": "database",
      "params": {"query": "SELECT * FROM users"}
    },
    "user_role": "marketing"
  }'</span>

<span class="c"># Response (403):</span>
<span class="c"># {</span>
<span class="c">#   "error": "Access Denied",</span>
<span class="c">#   "message": "Tool 'database' is not allowed for role 'marketing'"</span>
<span class="c"># }</span>
</code></pre>

</div>



<p>This single change prevents entire categories of security issues.</p>




<h2>
  
  
  🔎 3. Implement Rate Limiting
</h2>

<p>An AI workflow once got stuck in a loop, hammering the database with thousands of queries per second. The costs spiked $2,000 in 2 hours before we caught it.</p>

<p>Rate limiting is your firewall against your own systems:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">class</span> <span class="n">RateLimiter</span> <span class="p">{</span>
  <span class="n">async</span> <span class="n">checkLimit</span><span class="p">(</span><span class="n">toolName</span><span class="p">,</span> <span class="n">userId</span><span class="p">,</span> <span class="n">limit</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">const</span> <span class="n">key</span> <span class="o">=</span> <span class="s">`${toolName}:${userId}`</span><span class="p">;</span>
    <span class="k">const</span> <span class="n">now</span> <span class="o">=</span> <span class="n">Date</span><span class="o">.</span><span class="n">now</span><span class="p">();</span>
    <span class="k">const</span> <span class="n">windowStart</span> <span class="o">=</span> <span class="n">now</span> <span class="o">-</span> <span class="m">60000</span><span class="p">;</span> <span class="c">// 1 minute</span>

    <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="n">this</span><span class="o">.</span><span class="n">windows</span><span class="o">.</span><span class="n">has</span><span class="p">(</span><span class="n">key</span><span class="p">))</span> <span class="p">{</span>
      <span class="n">this</span><span class="o">.</span><span class="n">windows</span><span class="o">.</span><span class="n">set</span><span class="p">(</span><span class="n">key</span><span class="p">,</span> <span class="p">[]);</span>
    <span class="p">}</span>

    <span class="k">const</span> <span class="n">timestamps</span> <span class="o">=</span> <span class="n">this</span><span class="o">.</span><span class="n">windows</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">key</span><span class="p">)</span>
      <span class="o">.</span><span class="n">filter</span><span class="p">(</span><span class="n">t</span> <span class="o">=&gt;</span> <span class="n">t</span> <span class="o">&gt;</span> <span class="n">windowStart</span><span class="p">);</span>

    <span class="k">if</span> <span class="p">(</span><span class="n">timestamps</span><span class="o">.</span><span class="n">length</span> <span class="o">&gt;=</span> <span class="n">limit</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">return</span> <span class="p">{</span>
        <span class="n">allowed</span><span class="o">:</span> <span class="no">false</span><span class="p">,</span>
        <span class="n">retryAfter</span><span class="o">:</span> <span class="n">Math</span><span class="o">.</span><span class="n">ceil</span><span class="p">((</span><span class="n">timestamps</span><span class="p">[</span><span class="m">0</span><span class="p">]</span> <span class="o">+</span> <span class="m">60000</span> <span class="o">-</span> <span class="n">now</span><span class="p">)</span> <span class="o">/</span> <span class="m">1000</span><span class="p">)</span>
      <span class="p">};</span>
    <span class="p">}</span>

    <span class="n">timestamps</span><span class="o">.</span><span class="n">push</span><span class="p">(</span><span class="n">now</span><span class="p">);</span>
    <span class="k">return</span> <span class="p">{</span> <span class="n">allowed</span><span class="o">:</span> <span class="no">true</span><span class="p">,</span> <span class="n">remaining</span><span class="o">:</span> <span class="n">limit</span> <span class="o">-</span> <span class="n">timestamps</span><span class="o">.</span><span class="n">length</span> <span class="p">};</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Real example - Rate limit exceeded:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/v1/mcp/tool/execute <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "tool_call": {
      "tool_name": "web_search",
      "params": {"query": "another search"}
    },
    "user_id": "user-123",
    "user_role": "marketing"
  }'</span>

<span class="c"># Response (429 - Rate Limited):</span>
<span class="c"># {</span>
<span class="c">#   "error": "Rate Limit Exceeded",</span>
<span class="c">#   "message": "Tool 'web_search' limit exceeded (100/min)",</span>
<span class="c">#   "retryAfter": 45</span>
<span class="c"># }</span>
</code></pre>

</div>



<p>The rate limiter caught what would have been a $5,000+ incident in under 30 seconds.</p>




<h2>
  
  
  📊 4. Track Costs and Audit Everything
</h2>

<p>Production AI systems need accountability. Who ran what? When? How much did it cost?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">AuditLog</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Timestamp</span>  <span class="n">time</span><span class="o">.</span><span class="n">Time</span>
    <span class="n">UserId</span>     <span class="kt">string</span>
    <span class="n">UserRole</span>   <span class="kt">string</span>
    <span class="n">ToolName</span>   <span class="kt">string</span>
    <span class="n">Success</span>    <span class="kt">bool</span>
    <span class="n">Cost</span>       <span class="kt">float64</span>
    <span class="n">Duration</span>   <span class="n">time</span><span class="o">.</span><span class="n">Duration</span>
    <span class="n">Error</span>      <span class="kt">string</span>
<span class="p">}</span>

<span class="n">async</span> <span class="n">function</span> <span class="n">executeTool</span><span class="p">(</span><span class="n">toolName</span><span class="p">,</span> <span class="n">params</span><span class="p">,</span> <span class="n">context</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">const</span> <span class="n">startTime</span> <span class="o">=</span> <span class="n">Date</span><span class="o">.</span><span class="n">now</span><span class="p">();</span>

  <span class="n">try</span> <span class="p">{</span>
    <span class="k">const</span> <span class="n">result</span> <span class="o">=</span> <span class="n">await</span> <span class="n">toolExecutor</span><span class="o">.</span><span class="n">execute</span><span class="p">(</span><span class="n">toolName</span><span class="p">,</span> <span class="n">params</span><span class="p">);</span>
    <span class="k">const</span> <span class="n">duration</span> <span class="o">=</span> <span class="n">Date</span><span class="o">.</span><span class="n">now</span><span class="p">()</span> <span class="o">-</span> <span class="n">startTime</span><span class="p">;</span>
    <span class="k">const</span> <span class="n">cost</span> <span class="o">=</span> <span class="n">calculateCost</span><span class="p">(</span><span class="n">toolName</span><span class="p">,</span> <span class="n">params</span><span class="p">);</span>

    <span class="n">await</span> <span class="n">auditLogger</span><span class="o">.</span><span class="n">log</span><span class="p">({</span>
      <span class="n">userId</span><span class="o">:</span> <span class="n">context</span><span class="o">.</span><span class="n">userId</span><span class="p">,</span>
      <span class="n">userRole</span><span class="o">:</span> <span class="n">context</span><span class="o">.</span><span class="n">userRole</span><span class="p">,</span>
      <span class="n">toolName</span><span class="p">,</span>
      <span class="n">success</span><span class="o">:</span> <span class="no">true</span><span class="p">,</span>
      <span class="n">cost</span><span class="p">,</span>
      <span class="n">duration</span>
    <span class="p">});</span>

    <span class="k">return</span> <span class="n">result</span><span class="p">;</span>
  <span class="p">}</span> <span class="n">catch</span> <span class="p">(</span><span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">await</span> <span class="n">auditLogger</span><span class="o">.</span><span class="n">log</span><span class="p">({</span>
      <span class="n">userId</span><span class="o">:</span> <span class="n">context</span><span class="o">.</span><span class="n">userId</span><span class="p">,</span>
      <span class="n">toolName</span><span class="p">,</span>
      <span class="n">success</span><span class="o">:</span> <span class="no">false</span><span class="p">,</span>
      <span class="kt">error</span><span class="o">:</span> <span class="kt">error</span><span class="o">.</span><span class="n">message</span>
    <span class="p">});</span>
    <span class="n">throw</span> <span class="kt">error</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Example - Cost breakdown:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>GET /v1/analytics/costs?team_id<span class="o">=</span>team-engineering&amp;period<span class="o">=</span>month

<span class="o">{</span>
  <span class="s2">"total_cost"</span>: <span class="s2">"</span><span class="nv">$127</span><span class="s2">.45"</span>,
  <span class="s2">"budget"</span>: <span class="s2">"</span><span class="nv">$1000</span><span class="s2">.00"</span>,
  <span class="s2">"remaining"</span>: <span class="s2">"</span><span class="nv">$872</span><span class="s2">.55"</span>,
  <span class="s2">"usage_by_tool"</span>: <span class="o">[</span>
    <span class="o">{</span>
      <span class="s2">"tool"</span>: <span class="s2">"web_search"</span>,
      <span class="s2">"calls"</span>: 1234,
      <span class="s2">"cost"</span>: <span class="s2">"</span><span class="nv">$12</span><span class="s2">.34"</span>
    <span class="o">}</span>,
    <span class="o">{</span>
      <span class="s2">"tool"</span>: <span class="s2">"database"</span>,
      <span class="s2">"calls"</span>: 567,
      <span class="s2">"cost"</span>: <span class="s2">"</span><span class="nv">$56</span><span class="s2">.70"</span>
    <span class="o">}</span>
  <span class="o">]</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This visibility was transformative. Teams saw exactly what they were spending. Anomalies became obvious.</p>




<h2>
  
  
  🖋️ The Complete Flow
</h2>

<p>Here's what tool execution looks like with all control layers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">app</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="s">"/v1/mcp/tool/execute"</span><span class="p">,</span> <span class="n">async</span> <span class="p">(</span><span class="n">req</span><span class="p">,</span> <span class="n">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">const</span> <span class="p">{</span> <span class="n">toolName</span><span class="p">,</span> <span class="n">params</span><span class="p">,</span> <span class="n">userId</span><span class="p">,</span> <span class="n">userRole</span><span class="p">,</span> <span class="n">teamId</span> <span class="p">}</span> <span class="o">=</span> <span class="n">req</span><span class="o">.</span><span class="n">body</span><span class="p">;</span>

  <span class="n">try</span> <span class="p">{</span>
    <span class="c">// 1. Check role-based access</span>
    <span class="n">await</span> <span class="n">checkToolAccess</span><span class="p">(</span><span class="n">userId</span><span class="p">,</span> <span class="n">userRole</span><span class="p">,</span> <span class="n">toolName</span><span class="p">);</span>

    <span class="c">// 2. Check rate limits</span>
    <span class="k">const</span> <span class="n">limit</span> <span class="o">=</span> <span class="n">roleLimits</span><span class="p">[</span><span class="n">userRole</span><span class="p">]</span><span class="err">?</span><span class="o">.</span><span class="p">[</span><span class="n">toolName</span><span class="p">];</span>
    <span class="k">const</span> <span class="n">rateLimitCheck</span> <span class="o">=</span> <span class="n">await</span> <span class="n">limiter</span><span class="o">.</span><span class="n">checkLimit</span><span class="p">(</span><span class="n">toolName</span><span class="p">,</span> <span class="n">userId</span><span class="p">,</span> <span class="n">limit</span><span class="p">);</span>
    <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="n">rateLimitCheck</span><span class="o">.</span><span class="n">allowed</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">return</span> <span class="n">res</span><span class="o">.</span><span class="n">status</span><span class="p">(</span><span class="m">429</span><span class="p">)</span><span class="o">.</span><span class="n">json</span><span class="p">({</span>
        <span class="kt">error</span><span class="o">:</span> <span class="s">"Rate Limit Exceeded"</span><span class="p">,</span>
        <span class="n">retryAfter</span><span class="o">:</span> <span class="n">rateLimitCheck</span><span class="o">.</span><span class="n">retryAfter</span>
      <span class="p">});</span>
    <span class="p">}</span>

    <span class="c">// 3. Check budget</span>
    <span class="k">const</span> <span class="n">cost</span> <span class="o">=</span> <span class="n">estimateCost</span><span class="p">(</span><span class="n">toolName</span><span class="p">,</span> <span class="n">params</span><span class="p">);</span>
    <span class="k">const</span> <span class="n">budgetCheck</span> <span class="o">=</span> <span class="n">await</span> <span class="n">budgetTracker</span><span class="o">.</span><span class="n">deductCost</span><span class="p">(</span><span class="n">teamId</span><span class="p">,</span> <span class="n">toolName</span><span class="p">,</span> <span class="n">cost</span><span class="p">);</span>
    <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="n">budgetCheck</span><span class="o">.</span><span class="n">allowed</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">return</span> <span class="n">res</span><span class="o">.</span><span class="n">status</span><span class="p">(</span><span class="m">402</span><span class="p">)</span><span class="o">.</span><span class="n">json</span><span class="p">({</span>
        <span class="kt">error</span><span class="o">:</span> <span class="s">"Budget Exceeded"</span>
      <span class="p">});</span>
    <span class="p">}</span>

    <span class="c">// 4. Execute tool</span>
    <span class="k">const</span> <span class="n">result</span> <span class="o">=</span> <span class="n">await</span> <span class="n">executeTool</span><span class="p">(</span><span class="n">toolName</span><span class="p">,</span> <span class="n">params</span><span class="p">);</span>

    <span class="c">// 5. Log the action</span>
    <span class="n">await</span> <span class="n">auditLogger</span><span class="o">.</span><span class="n">log</span><span class="p">({</span>
      <span class="n">userId</span><span class="p">,</span> <span class="n">userRole</span><span class="p">,</span> <span class="n">teamId</span><span class="p">,</span> <span class="n">toolName</span><span class="p">,</span>
      <span class="n">success</span><span class="o">:</span> <span class="no">true</span><span class="p">,</span> <span class="n">cost</span><span class="p">,</span> <span class="n">duration</span>
    <span class="p">});</span>

    <span class="n">res</span><span class="o">.</span><span class="n">json</span><span class="p">({</span> <span class="n">success</span><span class="o">:</span> <span class="no">true</span><span class="p">,</span> <span class="n">data</span><span class="o">:</span> <span class="n">result</span> <span class="p">});</span>

  <span class="p">}</span> <span class="n">catch</span> <span class="p">(</span><span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="c">// Log failures too</span>
    <span class="n">await</span> <span class="n">auditLogger</span><span class="o">.</span><span class="n">log</span><span class="p">({</span>
      <span class="n">userId</span><span class="p">,</span> <span class="n">userRole</span><span class="p">,</span> <span class="n">teamId</span><span class="p">,</span> <span class="n">toolName</span><span class="p">,</span>
      <span class="n">success</span><span class="o">:</span> <span class="no">false</span><span class="p">,</span> <span class="kt">error</span><span class="o">:</span> <span class="kt">error</span><span class="o">.</span><span class="n">message</span>
    <span class="p">});</span>

    <span class="n">res</span><span class="o">.</span><span class="n">status</span><span class="p">(</span><span class="m">400</span><span class="p">)</span><span class="o">.</span><span class="n">json</span><span class="p">({</span> <span class="n">success</span><span class="o">:</span> <span class="no">false</span><span class="p">,</span> <span class="kt">error</span><span class="o">:</span> <span class="kt">error</span><span class="o">.</span><span class="n">message</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  ✅ Code Mode
</h2>

<p>Instead of calling tools one by one, models generate TypeScript code that orchestrates them:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Model generates this automatically</span>
<span class="kd">const</span> <span class="nx">tools</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">listToolFiles</span><span class="p">();</span>  <span class="c1">// List available tools</span>
<span class="kd">const</span> <span class="nx">githubTool</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">readToolFile</span><span class="p">(</span><span class="dl">'</span><span class="s1">github</span><span class="dl">'</span><span class="p">);</span>  <span class="c1">// Read definition</span>

<span class="c1">// Execute a complete workflow</span>
<span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">executeToolCode</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">repos</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">github</span><span class="p">.</span><span class="nf">search_repos</span><span class="p">({</span> 
    <span class="na">query</span><span class="p">:</span> <span class="dl">"</span><span class="s2">golang bifrost</span><span class="dl">"</span><span class="p">,</span> 
    <span class="na">maxResults</span><span class="p">:</span> <span class="mi">5</span> 
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">formatted</span> <span class="o">=</span> <span class="nx">repos</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">repo</span> <span class="o">=&gt;</span> <span class="p">({</span>
    <span class="na">name</span><span class="p">:</span> <span class="nx">repo</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
    <span class="na">stars</span><span class="p">:</span> <span class="nx">repo</span><span class="p">.</span><span class="nx">stargazers_count</span><span class="p">,</span>
    <span class="na">url</span><span class="p">:</span> <span class="nx">repo</span><span class="p">.</span><span class="nx">html_url</span>
  <span class="p">}));</span>

  <span class="k">return</span> <span class="p">{</span> <span class="na">repositories</span><span class="p">:</span> <span class="nx">formatted</span><span class="p">,</span> <span class="na">count</span><span class="p">:</span> <span class="nx">formatted</span><span class="p">.</span><span class="nx">length</span> <span class="p">};</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Benefits:</strong></p>

<ul>
<li>~40% reduction in token usage</li>
<li>Single execution vs multiple calls</li>
<li>Better control and debugging</li>
<li>Faster execution</li>
</ul>




<h2>
  
  
  📊 Key Metrics
</h2>

<p>Bifrost performance at 5,000 RPS:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>LiteLLM</th>
<th>Bifrost</th>
<th>Improvement</th>
</tr>
</thead>
<tbody>
<tr>
<td>Gateway Overhead</td>
<td>~440 µs</td>
<td>~11 µs</td>
<td><strong>40x faster</strong></td>
</tr>
<tr>
<td>Memory Usage</td>
<td>Baseline</td>
<td>-68%</td>
<td><strong>68% less</strong></td>
</tr>
<tr>
<td>Queue Wait</td>
<td>47 µs</td>
<td>1.67 µs</td>
<td><strong>28x faster</strong></td>
</tr>
<tr>
<td>Success Rate</td>
<td>89%</td>
<td>100%</td>
<td><strong>Perfect</strong></td>
</tr>
</tbody>
</table></div>

<p>Why Go language?</p>

<ul>
<li>Goroutines: lightweight concurrency (~2 KB each)</li>
<li>Compiled binary: no startup overhead</li>
<li>Memory efficient: 68% less than another</li>
<li>True parallelism across CPU cores</li>
</ul>




<h2>
  
  
  ⚙️ Getting Started
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Install Bifrost (30 seconds)</span>
npx <span class="nt">-y</span> @maximhq/bifrost

<span class="c"># 2. Configure API keys (.env)</span>
<span class="nv">OPENAI_API_KEY</span><span class="o">=</span>sk-...
<span class="nv">ANTHROPIC_API_KEY</span><span class="o">=</span>sk-ant-...

<span class="c"># 3. Open dashboard</span>
open http://localhost:8000

<span class="c"># 4. Make your first call</span>
curl <span class="nt">-X</span> POST http://localhost:8000/v1/chat/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Hello Bifrost!"}]
  }'</span>

<span class="c"># 5. Drop-in replacement</span>
<span class="c"># Change this:</span>
base_url <span class="o">=</span> <span class="s2">"https://api.openai.com"</span>
<span class="c"># To this:</span>
base_url <span class="o">=</span> <span class="s2">"http://localhost:8000/openai"</span>
</code></pre>

</div>






<h2>
  
  
  💻 What I'd Do Differently
</h2>

<ol>
<li>
<strong>Start with cost tracking from day one</strong> - Retrofit is painful</li>
<li>
<strong>Make rate limits configurable</strong> - Teams have different needs</li>
<li>
<strong>Implement caching aggressively</strong> - Semantic caching saves 40%+</li>
<li>
<strong>Build hierarchical permissions</strong> - Flat models don't scale</li>
<li>
<strong>Set up real-time alerting</strong> - Don't wait for weekly reviews</li>
</ol>




<h2>
  
  
  ✅ The Real Benefit
</h2>

<p>At the end of the day, the gateway isn't about being fancy. It's about <strong>control</strong>.</p>

<p>When you centralize tool management, you get:</p>

<ul>
<li>
<strong>Security</strong> - Tools isolated by role, mistakes bounded</li>
<li>
<strong>Visibility</strong> - Every action logged and costs tracked</li>
<li>
<strong>Optimization</strong> - See what's expensive and fix it</li>
<li>
<strong>Debugging</strong> - Complete audit trail for incidents</li>
</ul>

<p>For us, this infrastructure turned AI from "a cool demo" into something we could deploy to production with confidence.</p>




<h2>
  
  
  🔗 Resources
</h2>

<ul>
<li>
<strong>Bifrost GitHub</strong>: <a href="https://github.com/maximhq/bifrost" rel="noopener noreferrer">https://github.com/maximhq/bifrost</a>
</li>
<li>
<strong>Documentation</strong>: <a href="https://docs.getbifrost.ai" rel="noopener noreferrer">https://docs.getbifrost.ai</a>
</li>
<li>
<strong>MCP Spec</strong>: <a href="https://modelcontextprotocol.io" rel="noopener noreferrer">https://modelcontextprotocol.io</a>
</li>
</ul>




<p><strong>Are you building AI infrastructure at scale?</strong> Let me know in the comments!</p>

<p>Thanks for reading!</p>

