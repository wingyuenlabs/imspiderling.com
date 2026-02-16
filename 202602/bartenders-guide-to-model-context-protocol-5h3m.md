---
Title: Bartender's Guide to Model Context Protocol
Description: 
Author: Igor Nosatov
Date: 2026-02-16T21:29:31.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  The Bartender's Guide to Model Context Protocol: Mixing the Perfect API Cocktail 🍸
</h1>

<p>Last month, I was building an AI assistant that needed to interact with our company's internal tools: Jira, Confluence, our PostgreSQL database, and our custom CRM. Each integration was a special snowflake of pain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># The nightmare before MCP
</span><span class="k">class</span> <span class="nc">MyFragileAgent</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">jira_client</span> <span class="o">=</span> <span class="nc">JiraAPI</span><span class="p">(</span><span class="n">auth</span><span class="o">=</span><span class="n">weird_oauth_flow</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">confluence</span> <span class="o">=</span> <span class="nc">ConfluenceWrapper</span><span class="p">(</span><span class="n">needs_custom_session</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">db</span> <span class="o">=</span> <span class="nc">PostgresConnection</span><span class="p">(</span><span class="n">with_connection_pooling</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">crm</span> <span class="o">=</span> <span class="nc">SomeRESTClient</span><span class="p">(</span><span class="n">uses_different_auth_again</span><span class="p">)</span>

    <span class="c1"># 200 lines of glue code later...
</span></code></pre>

</div>



<p>Every tool needed its own authentication dance, error handling strategy, and data transformation layer. My AI agent was less "intelligent assistant" and more "integration hell supervisor."</p>

<p>The real kicker? When Claude or GPT-4 wanted to use these tools, I had to write <em>even more</em> code to translate between their function-calling formats and my janky integrations.</p>

<h2>
  
  
  Enter MCP: The Universal Bartender's Handbook
</h2>

<p>Model Context Protocol is Anthropic's answer to this chaos. Think of it as a standardized way for AI models to interact with external tools, data sources, and services. Instead of every AI application reinventing the wheel, MCP provides:</p>

<ol>
<li>
<strong>A standard server protocol</strong> (the bar's layout)</li>
<li>
<strong>Resource discovery</strong> (the menu)</li>
<li>
<strong>Tool definitions</strong> (the recipes)</li>
<li>
<strong>Streaming context</strong> (the conversation with your bartender)</li>
</ol>

<p>Here's the beautiful part: write an MCP server once, use it with any MCP-compatible client.</p>

<h2>
  
  
  The Anatomy of an MCP Server: Building Your First "Bar"
</h2>

<p>Let's build a simple MCP server that exposes a GitHub repository as a resource. I'll show you the surprisingly elegant structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">mcp.server</span> <span class="kn">import</span> <span class="n">Server</span>
<span class="kn">from</span> <span class="n">mcp.types</span> <span class="kn">import</span> <span class="n">Resource</span><span class="p">,</span> <span class="n">Tool</span><span class="p">,</span> <span class="n">TextContent</span>
<span class="kn">import</span> <span class="n">httpx</span>

<span class="c1"># Initialize our "bar"
</span><span class="n">app</span> <span class="o">=</span> <span class="nc">Server</span><span class="p">(</span><span class="sh">"</span><span class="s">github-mcp-server</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Define what's "on the menu" - our resources
</span><span class="nd">@app.list_resources</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">list_resources</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="n">Resource</span><span class="p">]:</span>
    <span class="k">return</span> <span class="p">[</span>
        <span class="nc">Resource</span><span class="p">(</span>
            <span class="n">uri</span><span class="o">=</span><span class="sh">"</span><span class="s">github://repos/user/repo</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">Repository Info</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">mimeType</span><span class="o">=</span><span class="sh">"</span><span class="s">application/json</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Access repository details</span><span class="sh">"</span>
        <span class="p">)</span>
    <span class="p">]</span>

<span class="c1"># The bartender knows how to serve what you order
</span><span class="nd">@app.read_resource</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">read_resource</span><span class="p">(</span><span class="n">uri</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">uri</span><span class="p">.</span><span class="nf">startswith</span><span class="p">(</span><span class="sh">"</span><span class="s">github://repos/</span><span class="sh">"</span><span class="p">):</span>
        <span class="n">repo_path</span> <span class="o">=</span> <span class="n">uri</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sh">"</span><span class="s">github://repos/</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span>
        <span class="k">async</span> <span class="k">with</span> <span class="n">httpx</span><span class="p">.</span><span class="nc">AsyncClient</span><span class="p">()</span> <span class="k">as</span> <span class="n">client</span><span class="p">:</span>
            <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="n">client</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
                <span class="sa">f</span><span class="sh">"</span><span class="s">https://api.github.com/repos/</span><span class="si">{</span><span class="n">repo_path</span><span class="si">}</span><span class="sh">"</span><span class="p">,</span>
                <span class="n">headers</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">Authorization</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">token </span><span class="si">{</span><span class="n">GITHUB_TOKEN</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>
            <span class="p">)</span>
            <span class="k">return</span> <span class="n">response</span><span class="p">.</span><span class="n">text</span>
    <span class="k">raise</span> <span class="nc">ValueError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Unknown resource: </span><span class="si">{</span><span class="n">uri</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Define tools (the bartender's special skills)
</span><span class="nd">@app.list_tools</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">list_tools</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="n">Tool</span><span class="p">]:</span>
    <span class="k">return</span> <span class="p">[</span>
        <span class="nc">Tool</span><span class="p">(</span>
            <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">create_issue</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Create a GitHub issue</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">inputSchema</span><span class="o">=</span><span class="p">{</span>
                <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">object</span><span class="sh">"</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">properties</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
                    <span class="sh">"</span><span class="s">repo</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">string</span><span class="sh">"</span><span class="p">},</span>
                    <span class="sh">"</span><span class="s">title</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">string</span><span class="sh">"</span><span class="p">},</span>
                    <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">string</span><span class="sh">"</span><span class="p">}</span>
                <span class="p">},</span>
                <span class="sh">"</span><span class="s">required</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="sh">"</span><span class="s">repo</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">title</span><span class="sh">"</span><span class="p">]</span>
            <span class="p">}</span>
        <span class="p">)</span>
    <span class="p">]</span>

<span class="nd">@app.call_tool</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">call_tool</span><span class="p">(</span><span class="n">name</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">arguments</span><span class="p">:</span> <span class="nb">dict</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="n">TextContent</span><span class="p">]:</span>
    <span class="k">if</span> <span class="n">name</span> <span class="o">==</span> <span class="sh">"</span><span class="s">create_issue</span><span class="sh">"</span><span class="p">:</span>
        <span class="c1"># Mix the cocktail (create the issue)
</span>        <span class="k">async</span> <span class="k">with</span> <span class="n">httpx</span><span class="p">.</span><span class="nc">AsyncClient</span><span class="p">()</span> <span class="k">as</span> <span class="n">client</span><span class="p">:</span>
            <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="n">client</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span>
                <span class="sa">f</span><span class="sh">"</span><span class="s">https://api.github.com/repos/</span><span class="si">{</span><span class="n">arguments</span><span class="p">[</span><span class="sh">'</span><span class="s">repo</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">/issues</span><span class="sh">"</span><span class="p">,</span>
                <span class="n">json</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">title</span><span class="sh">"</span><span class="p">:</span> <span class="n">arguments</span><span class="p">[</span><span class="sh">"</span><span class="s">title</span><span class="sh">"</span><span class="p">],</span> <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">arguments</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)},</span>
                <span class="n">headers</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">Authorization</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">token </span><span class="si">{</span><span class="n">GITHUB_TOKEN</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>
            <span class="p">)</span>
            <span class="k">return</span> <span class="p">[</span><span class="nc">TextContent</span><span class="p">(</span>
                <span class="nb">type</span><span class="o">=</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">,</span>
                <span class="n">text</span><span class="o">=</span><span class="sa">f</span><span class="sh">"</span><span class="s">Created issue #</span><span class="si">{</span><span class="n">response</span><span class="p">.</span><span class="nf">json</span><span class="p">()[</span><span class="sh">'</span><span class="s">number</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span>
            <span class="p">)]</span>
</code></pre>

</div>



<h2>
  
  
  The Three Core Ingredients: Resources, Prompts, and Tools
</h2>

<p>MCP servers can serve three types of "drinks":</p>

<h3>
  
  
  1. Resources (The Ingredients)
</h3>

<p>Resources are data sources that the AI can read:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>github://repos/anthropics/mcp  → Repository metadata
file:///home/docs/api.md       → Documentation
postgres://localhost/users     → Database tables
</code></pre>

</div>



<p>Think of resources as the bottles behind the bar. The AI can ask, "What do you have?" and get a structured list.</p>

<h3>
  
  
  2. Prompts (The House Specials)
</h3>

<p>Prompts are pre-defined templates that guide the AI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nd">@app.list_prompts</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">list_prompts</span><span class="p">():</span>
    <span class="k">return</span> <span class="p">[</span>
        <span class="nc">Prompt</span><span class="p">(</span>
            <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">debug_issue</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Help debug a GitHub issue</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">arguments</span><span class="o">=</span><span class="p">[</span>
                <span class="nc">PromptArgument</span><span class="p">(</span>
                    <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">issue_number</span><span class="sh">"</span><span class="p">,</span>
                    <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Issue number to debug</span><span class="sh">"</span><span class="p">,</span>
                    <span class="n">required</span><span class="o">=</span><span class="bp">True</span>
                <span class="p">)</span>
            <span class="p">]</span>
        <span class="p">)</span>
    <span class="p">]</span>

<span class="nd">@app.get_prompt</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">get_prompt</span><span class="p">(</span><span class="n">name</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">arguments</span><span class="p">:</span> <span class="nb">dict</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">name</span> <span class="o">==</span> <span class="sh">"</span><span class="s">debug_issue</span><span class="sh">"</span><span class="p">:</span>
        <span class="n">issue</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch_issue</span><span class="p">(</span><span class="n">arguments</span><span class="p">[</span><span class="sh">"</span><span class="s">issue_number</span><span class="sh">"</span><span class="p">])</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"""</span><span class="s">Analyze this GitHub issue and suggest solutions:

Title: </span><span class="si">{</span><span class="n">issue</span><span class="p">[</span><span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">
Body: </span><span class="si">{</span><span class="n">issue</span><span class="p">[</span><span class="sh">'</span><span class="s">body</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">
Comments: </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">issue</span><span class="p">[</span><span class="sh">'</span><span class="s">comments</span><span class="sh">'</span><span class="p">])</span><span class="si">}</span><span class="s"> comments
Labels: </span><span class="si">{</span><span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">issue</span><span class="p">[</span><span class="sh">'</span><span class="s">labels</span><span class="sh">'</span><span class="p">])</span><span class="si">}</span><span class="s">

Provide a structured debugging approach.</span><span class="sh">"""</span>
</code></pre>

</div>



<h3>
  
  
  3. Tools (The Bartender's Skills)
</h3>

<p>Tools are actions the AI can perform:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Think of tools as verbs: create, update, delete, search, analyze
</span><span class="nf">create_issue</span><span class="p">()</span>
<span class="nf">search_codebase</span><span class="p">()</span>
<span class="nf">run_tests</span><span class="p">()</span>
<span class="nf">deploy_service</span><span class="p">()</span>
</code></pre>

</div>



<h2>
  
  
  The Real Magic: Client-Side Integration
</h2>

<p>Here's where MCP shines. On the client side (Claude, your custom AI app, etc.), connecting to an MCP server is ridiculously simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">mcp</span> <span class="kn">import</span> <span class="n">ClientSession</span><span class="p">,</span> <span class="n">StdioServerParameters</span>
<span class="kn">from</span> <span class="n">mcp.client.stdio</span> <span class="kn">import</span> <span class="n">stdio_client</span>

<span class="c1"># Connect to our "bar"
</span><span class="n">server_params</span> <span class="o">=</span> <span class="nc">StdioServerParameters</span><span class="p">(</span>
    <span class="n">command</span><span class="o">=</span><span class="sh">"</span><span class="s">python</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">github_mcp_server.py</span><span class="sh">"</span><span class="p">]</span>
<span class="p">)</span>

<span class="k">async</span> <span class="k">with</span> <span class="nf">stdio_client</span><span class="p">(</span><span class="n">server_params</span><span class="p">)</span> <span class="nf">as </span><span class="p">(</span><span class="n">read</span><span class="p">,</span> <span class="n">write</span><span class="p">):</span>
    <span class="k">async</span> <span class="k">with</span> <span class="nc">ClientSession</span><span class="p">(</span><span class="n">read</span><span class="p">,</span> <span class="n">write</span><span class="p">)</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
        <span class="c1"># Initialize - like walking into the bar
</span>        <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">initialize</span><span class="p">()</span>

        <span class="c1"># Ask what's available - check the menu
</span>        <span class="n">resources</span> <span class="o">=</span> <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">list_resources</span><span class="p">()</span>
        <span class="n">tools</span> <span class="o">=</span> <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">list_tools</span><span class="p">()</span>

        <span class="c1"># Order something - use a tool
</span>        <span class="n">result</span> <span class="o">=</span> <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">call_tool</span><span class="p">(</span><span class="sh">"</span><span class="s">create_issue</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">repo</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropics/mcp</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">title</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">MCP is awesome!</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Seriously, this protocol rocks.</span><span class="sh">"</span>
        <span class="p">})</span>
</code></pre>

</div>



<h2>
  
  
  The Plot Twist: Why This Changes Everything
</h2>

<p>Remember my fragile agent from the beginning? Here's what it looks like with MCP:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">MyElegantAgent</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="c1"># Connect to MCP servers - that's it!
</span>        <span class="n">self</span><span class="p">.</span><span class="n">servers</span> <span class="o">=</span> <span class="p">[</span>
            <span class="nc">MCPClient</span><span class="p">(</span><span class="sh">"</span><span class="s">jira-mcp-server</span><span class="sh">"</span><span class="p">),</span>
            <span class="nc">MCPClient</span><span class="p">(</span><span class="sh">"</span><span class="s">confluence-mcp-server</span><span class="sh">"</span><span class="p">),</span>
            <span class="nc">MCPClient</span><span class="p">(</span><span class="sh">"</span><span class="s">postgres-mcp-server</span><span class="sh">"</span><span class="p">),</span>
            <span class="nc">MCPClient</span><span class="p">(</span><span class="sh">"</span><span class="s">crm-mcp-server</span><span class="sh">"</span><span class="p">)</span>
        <span class="p">]</span>

    <span class="k">async</span> <span class="k">def</span> <span class="nf">ask_claude</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">question</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="c1"># Claude automatically discovers and uses tools
</span>        <span class="c1"># from all connected MCP servers
</span>        <span class="n">tools</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="k">for</span> <span class="n">server</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">servers</span><span class="p">:</span>
            <span class="n">tools</span><span class="p">.</span><span class="nf">extend</span><span class="p">(</span><span class="k">await</span> <span class="n">server</span><span class="p">.</span><span class="nf">list_tools</span><span class="p">())</span>

        <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="n">anthropic_client</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
            <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-3-5-sonnet-20241022</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">tools</span><span class="o">=</span><span class="n">tools</span><span class="p">,</span>
            <span class="n">messages</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">question</span><span class="p">}]</span>
        <span class="p">)</span>
        <span class="k">return</span> <span class="n">response</span>
</code></pre>

</div>



<p><strong>Zero custom glue code.</strong> The MCP servers handle their own authentication, error handling, and data formatting. Claude natively understands how to call MCP tools.</p>

<h2>
  
  
  The Secret Sauce: Composability
</h2>

<p>Here's the bartender analogy's payoff: just like a good bar can source ingredients from multiple suppliers, an MCP client can connect to multiple servers simultaneously:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────┐
│   Claude    │
│  (or GPT)   │
└──────┬──────┘
       │
       ├──────► MCP Server: GitHub
       ├──────► MCP Server: PostgreSQL
       ├──────► MCP Server: Slack
       ├──────► MCP Server: Filesystem
       └──────► MCP Server: Your Custom API
</code></pre>

</div>



<p>Each server is an independent service. Add or remove them without touching your AI code.</p>

<h2>
  
  
  Real-World Recipe: A Multi-Tool Development Assistant
</h2>

<p>Let's build something practical - a dev assistant that can:</p>

<ol>
<li>Read your codebase (filesystem MCP server)</li>
<li>Search Stack Overflow (web search MCP server)</li>
<li>Create issues in GitHub (GitHub MCP server)</li>
<li>Query your production database (PostgreSQL MCP server)
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># File: dev_assistant.py
</span><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">anthropic</span> <span class="kn">import</span> <span class="n">Anthropic</span>
<span class="kn">from</span> <span class="n">mcp</span> <span class="kn">import</span> <span class="n">ClientSession</span><span class="p">,</span> <span class="n">StdioServerParameters</span>
<span class="kn">from</span> <span class="n">mcp.client.stdio</span> <span class="kn">import</span> <span class="n">stdio_client</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">dev_assistant</span><span class="p">():</span>
    <span class="c1"># Start multiple MCP servers
</span>    <span class="n">servers</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">filesystem</span><span class="sh">"</span><span class="p">:</span> <span class="nc">StdioServerParameters</span><span class="p">(</span>
            <span class="n">command</span><span class="o">=</span><span class="sh">"</span><span class="s">npx</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">-y</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">@modelcontextprotocol/server-filesystem</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">/path/to/code</span><span class="sh">"</span><span class="p">]</span>
        <span class="p">),</span>
        <span class="sh">"</span><span class="s">github</span><span class="sh">"</span><span class="p">:</span> <span class="nc">StdioServerParameters</span><span class="p">(</span>
            <span class="n">command</span><span class="o">=</span><span class="sh">"</span><span class="s">npx</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">-y</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">@modelcontextprotocol/server-github</span><span class="sh">"</span><span class="p">]</span>
        <span class="p">),</span>
        <span class="sh">"</span><span class="s">postgres</span><span class="sh">"</span><span class="p">:</span> <span class="nc">StdioServerParameters</span><span class="p">(</span>
            <span class="n">command</span><span class="o">=</span><span class="sh">"</span><span class="s">npx</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">-y</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">@modelcontextprotocol/server-postgres</span><span class="sh">"</span><span class="p">,</span> 
                  <span class="sh">"</span><span class="s">postgresql://localhost/mydb</span><span class="sh">"</span><span class="p">]</span>
        <span class="p">)</span>
    <span class="p">}</span>

    <span class="n">sessions</span> <span class="o">=</span> <span class="p">{}</span>
    <span class="n">all_tools</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="c1"># Connect to all servers
</span>    <span class="k">for</span> <span class="n">name</span><span class="p">,</span> <span class="n">params</span> <span class="ow">in</span> <span class="n">servers</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
        <span class="n">read</span><span class="p">,</span> <span class="n">write</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">stdio_client</span><span class="p">(</span><span class="n">params</span><span class="p">).</span><span class="nf">__aenter__</span><span class="p">()</span>
        <span class="n">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nc">ClientSession</span><span class="p">(</span><span class="n">read</span><span class="p">,</span> <span class="n">write</span><span class="p">).</span><span class="nf">__aenter__</span><span class="p">()</span>
        <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">initialize</span><span class="p">()</span>
        <span class="n">sessions</span><span class="p">[</span><span class="n">name</span><span class="p">]</span> <span class="o">=</span> <span class="n">session</span>

        <span class="c1"># Gather all available tools
</span>        <span class="n">tools_list</span> <span class="o">=</span> <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">list_tools</span><span class="p">()</span>
        <span class="n">all_tools</span><span class="p">.</span><span class="nf">extend</span><span class="p">(</span><span class="n">tools_list</span><span class="p">)</span>

    <span class="c1"># Now Claude can use any tool from any server!
</span>    <span class="n">client</span> <span class="o">=</span> <span class="nc">Anthropic</span><span class="p">()</span>

    <span class="n">question</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
    Look at the recent error logs in logs/error.log,
    search for similar issues on Stack Overflow,
    check if this affects our user_sessions table,
    and create a GitHub issue if it</span><span class="sh">'</span><span class="s">s a bug.
    </span><span class="sh">"""</span>

    <span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
        <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">claude-3-5-sonnet-20241022</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">max_tokens</span><span class="o">=</span><span class="mi">4096</span><span class="p">,</span>
        <span class="n">tools</span><span class="o">=</span><span class="n">all_tools</span><span class="p">,</span>  <span class="c1"># Claude sees tools from ALL servers
</span>        <span class="n">messages</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">question</span><span class="p">}]</span>
    <span class="p">)</span>

    <span class="c1"># Handle tool calls (simplified)
</span>    <span class="k">for</span> <span class="n">block</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">content</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">block</span><span class="p">.</span><span class="nb">type</span> <span class="o">==</span> <span class="sh">"</span><span class="s">tool_use</span><span class="sh">"</span><span class="p">:</span>
            <span class="c1"># Route to correct server
</span>            <span class="n">server</span> <span class="o">=</span> <span class="nf">find_server_for_tool</span><span class="p">(</span><span class="n">block</span><span class="p">.</span><span class="n">name</span><span class="p">,</span> <span class="n">sessions</span><span class="p">)</span>
            <span class="n">result</span> <span class="o">=</span> <span class="k">await</span> <span class="n">server</span><span class="p">.</span><span class="nf">call_tool</span><span class="p">(</span><span class="n">block</span><span class="p">.</span><span class="n">name</span><span class="p">,</span> <span class="n">block</span><span class="p">.</span><span class="nb">input</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Tool result: </span><span class="si">{</span><span class="n">result</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">asyncio</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="nf">dev_assistant</span><span class="p">())</span>
</code></pre>

</div>



<h2>
  
  
  The Gotchas (Things Your Bartender Won't Tell You)
</h2>

<h3>
  
  
  1. Authentication Management
</h3>

<p>MCP servers handle their own auth, but you need to configure it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">claude_desktop_config.json</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"github"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npx"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"args"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"-y"</span><span class="p">,</span><span class="w"> </span><span class="s2">"@modelcontextprotocol/server-github"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"env"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"GITHUB_TOKEN"</span><span class="p">:</span><span class="w"> </span><span class="s2">"your_token_here"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Pro tip:</strong> Use environment variables or secret managers, never hardcode tokens.</p>

<h3>
  
  
  2. Error Handling
</h3>

<p>MCP servers can fail. Always implement retry logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">tenacity</span> <span class="kn">import</span> <span class="n">retry</span><span class="p">,</span> <span class="n">stop_after_attempt</span><span class="p">,</span> <span class="n">wait_exponential</span>

<span class="nd">@retry</span><span class="p">(</span><span class="n">stop</span><span class="o">=</span><span class="nf">stop_after_attempt</span><span class="p">(</span><span class="mi">3</span><span class="p">),</span> <span class="n">wait</span><span class="o">=</span><span class="nf">wait_exponential</span><span class="p">(</span><span class="n">multiplier</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="nb">min</span><span class="o">=</span><span class="mi">2</span><span class="p">,</span> <span class="nb">max</span><span class="o">=</span><span class="mi">10</span><span class="p">))</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">call_tool_with_retry</span><span class="p">(</span><span class="n">session</span><span class="p">,</span> <span class="n">tool_name</span><span class="p">,</span> <span class="n">args</span><span class="p">):</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="k">return</span> <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">call_tool</span><span class="p">(</span><span class="n">tool_name</span><span class="p">,</span> <span class="n">args</span><span class="p">)</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Tool call failed: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">raise</span>
</code></pre>

</div>



<h3>
  
  
  3. Performance Considerations
</h3>

<p>Each MCP server is a separate process. Connection overhead matters:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># BAD: Connect for every request
</span><span class="k">async</span> <span class="k">def</span> <span class="nf">bad_pattern</span><span class="p">():</span>
    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">100</span><span class="p">):</span>
        <span class="k">async</span> <span class="k">with</span> <span class="nf">stdio_client</span><span class="p">(</span><span class="n">params</span><span class="p">)</span> <span class="nf">as </span><span class="p">(</span><span class="n">read</span><span class="p">,</span> <span class="n">write</span><span class="p">):</span>
            <span class="k">async</span> <span class="k">with</span> <span class="nc">ClientSession</span><span class="p">(</span><span class="n">read</span><span class="p">,</span> <span class="n">write</span><span class="p">)</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
                <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">call_tool</span><span class="p">(</span><span class="sh">"</span><span class="s">some_tool</span><span class="sh">"</span><span class="p">,</span> <span class="p">{})</span>

<span class="c1"># GOOD: Reuse connections
</span><span class="k">async</span> <span class="k">def</span> <span class="nf">good_pattern</span><span class="p">():</span>
    <span class="k">async</span> <span class="k">with</span> <span class="nf">stdio_client</span><span class="p">(</span><span class="n">params</span><span class="p">)</span> <span class="nf">as </span><span class="p">(</span><span class="n">read</span><span class="p">,</span> <span class="n">write</span><span class="p">):</span>
        <span class="k">async</span> <span class="k">with</span> <span class="nc">ClientSession</span><span class="p">(</span><span class="n">read</span><span class="p">,</span> <span class="n">write</span><span class="p">)</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
            <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">100</span><span class="p">):</span>
                <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">call_tool</span><span class="p">(</span><span class="sh">"</span><span class="s">some_tool</span><span class="sh">"</span><span class="p">,</span> <span class="p">{})</span>
</code></pre>

</div>



<h2>
  
  
  The Future Bar: Where MCP is Heading
</h2>

<p>The MCP ecosystem is exploding. Here's what's brewing:</p>

<ol>
<li>
<strong>Pre-built servers</strong> for popular services (already 30+ official implementations)</li>
<li>
<strong>Sampling support</strong> - servers can request AI completions (mind-bending recursion!)</li>
<li>
<strong>Streaming resources</strong> - real-time data feeds</li>
<li>
<strong>Multi-modal resources</strong> - images, audio, video</li>
</ol>

<p>Imagine:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Future MCP server for security scanning
</span><span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">call_tool</span><span class="p">(</span><span class="sh">"</span><span class="s">scan_dependencies</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">project_path</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">/my/app</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">ai_explanation</span><span class="sh">"</span><span class="p">:</span> <span class="bp">True</span>  <span class="c1"># Server asks Claude to explain vulnerabilities!
</span><span class="p">})</span>
</code></pre>

</div>



<h2>
  
  
  Your Takeaway Menu
</h2>

<p><strong>Start Here:</strong></p>

<ol>
<li>Install the MCP SDK: <code>pip install mcp</code>
</li>
<li>Try an existing server: <a href="https://github.com/modelcontextprotocol" rel="noopener noreferrer">GitHub's MCP Explorer</a>
</li>
<li>Build a simple server for your most-used API</li>
</ol>

<p><strong>Quick Wins:</strong></p>

<ul>
<li>Wrap your internal APIs as MCP servers</li>
<li>Connect Claude to your company's knowledge base</li>
<li>Build tool chains without custom integration code</li>
</ul>

<p><strong>Watch Out For:</strong></p>

<ul>
<li>Credential management (use env vars!)</li>
<li>Connection pooling for production</li>
<li>Tool naming conflicts when running multiple servers</li>
</ul>

<h2>
  
  
  The Last Call
</h2>

<p>Model Context Protocol isn't just another API standard - it's a fundamental shift in how we build AI applications. Instead of building monolithic AI systems with hardcoded integrations, we're moving toward a composable ecosystem where:</p>

<ul>
<li>
<strong>Servers</strong> are reusable, focused, and maintained independently</li>
<li>
<strong>Clients</strong> (AI models) discover and use tools dynamically</li>
<li>
<strong>Developers</strong> assemble capabilities like mixing cocktails</li>
</ul>

<p>The bartender doesn't need to know how to make every drink ever invented. They just need to know the fundamentals and have access to the right ingredients.</p>

<p>Your AI agents are the same way.</p>

