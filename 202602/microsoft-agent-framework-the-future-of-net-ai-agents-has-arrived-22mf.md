---
Title: Microsoft Agent Framework: The Future of .NET AI Agents Has Arrived
Description: 
Author: Brian Spann
Date: 2026-02-16T21:27:19.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>If you've been building AI applications in .NET, 2025 was a whirlwind year. We had Semantic Kernel reaching maturity, AutoGen capturing attention with its multi-agent patterns, and then... the announcement that changed everything: <strong>Microsoft Agent Framework</strong>—the unification of both projects into a single, cohesive SDK.</p>

<p>This isn't just a rebrand. It's a fundamental rethinking of how we build AI agents in .NET, combining the production-grade foundations of Semantic Kernel with the research-driven orchestration patterns of AutoGen.</p>

<p>In this first article of our 4-part series, we'll explore:</p>

<ul>
<li>Why Microsoft unified these projects</li>
<li>What Agent Framework inherits from each predecessor</li>
<li>How to get started with your first agent</li>
<li>Migration paths if you're coming from SK or AutoGen</li>
</ul>




<h2>
  
  
  The Evolution: A Brief History
</h2>

<h3>
  
  
  Semantic Kernel (2023-2025)
</h3>

<p>Semantic Kernel emerged as Microsoft's answer to LangChain—a strongly-typed, enterprise-ready framework for building AI applications. It brought us:</p>

<ul>
<li>
<strong>Plugins and native functions</strong>: Type-safe tool calling</li>
<li>
<strong>Planners</strong>: AI-driven task decomposition</li>
<li>
<strong>Memory connectors</strong>: Vector store integrations</li>
<li>
<strong>Filters and middleware</strong>: Production observability</li>
<li>
<strong>Multi-model support</strong>: OpenAI, Azure OpenAI, local models</li>
</ul>

<p>SK found its niche in enterprise applications where C# developers needed to integrate LLMs into existing .NET ecosystems. It was stable, well-documented, and played nicely with dependency injection.</p>

<h3>
  
  
  AutoGen (2023-2025)
</h3>

<p>While Semantic Kernel focused on single-agent patterns, AutoGen (from Microsoft Research) explored a different frontier: <strong>multi-agent orchestration</strong>. Its innovations included:</p>

<ul>
<li>
<strong>Conversational agents</strong>: Agents that talk to each other</li>
<li>
<strong>GroupChat patterns</strong>: Multiple agents collaborating on tasks</li>
<li>
<strong>Human-in-the-loop</strong>: Seamless integration of human oversight</li>
<li>
<strong>Code execution</strong>: Agents that could write and run code</li>
</ul>

<p>AutoGen was experimental, research-driven, and pushed boundaries. But it wasn't always production-ready, and its Python-first approach left .NET developers wanting.</p>

<h3>
  
  
  The Unification (October 2025)
</h3>

<p>At .NET Conf 2025, Microsoft announced what many had hoped for: <strong>a unified agent framework</strong> that combines the best of both worlds. The key insight? These weren't competing projects—they were complementary pieces of a larger puzzle.</p>




<h2>
  
  
  What Agent Framework Brings to the Table
</h2>

<h3>
  
  
  From Semantic Kernel:
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>What It Means for You</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Type-safe tools</strong></td>
<td>Define functions with attributes, get automatic schema generation</td>
</tr>
<tr>
<td><strong>State management</strong></td>
<td>Session persistence, conversation history, context windows</td>
</tr>
<tr>
<td><strong>Filters &amp; middleware</strong></td>
<td>Logging, caching, content moderation at the pipeline level</td>
</tr>
<tr>
<td><strong>Telemetry integration</strong></td>
<td>OpenTelemetry support out of the box</td>
</tr>
<tr>
<td><strong>Model abstraction</strong></td>
<td>Swap providers without changing agent code</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  From AutoGen:
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>What It Means for You</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Multi-agent patterns</strong></td>
<td>Pre-built orchestration: round-robin, selector, broadcast</td>
</tr>
<tr>
<td><strong>Workflows</strong></td>
<td>Explicit, durable task orchestration</td>
</tr>
<tr>
<td><strong>Agent-to-Agent (A2A)</strong></td>
<td>Standardized protocol for agent communication</td>
</tr>
<tr>
<td><strong>Magentic One</strong></td>
<td>Research-proven patterns for complex tasks</td>
</tr>
<tr>
<td><strong>Research foundation</strong></td>
<td>Patterns validated through academic research</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  New in Agent Framework:
</h3>

<ul>
<li>
<strong>MCP Integration</strong>: First-class support for Model Context Protocol</li>
<li>
<strong>Workflow Engine</strong>: Visual, durable, checkpointed workflows</li>
<li>
<strong>Azure AI Foundry</strong>: Seamless cloud deployment</li>
<li>
<strong>Microsoft.Extensions.AI</strong>: Built on the new AI abstractions</li>
</ul>




<h2>
  
  
  Getting Started
</h2>

<h3>
  
  
  Installation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create a new project</span>
dotnet new console <span class="nt">-n</span> MyFirstAgent
<span class="nb">cd </span>MyFirstAgent

<span class="c"># Add the Agent Framework package</span>
dotnet add package Microsoft.Agents.AI <span class="nt">--prerelease</span>

<span class="c"># Add Azure OpenAI support</span>
dotnet add package Microsoft.Extensions.AI.AzureAIInference
</code></pre>

</div>



<h3>
  
  
  Your First Agent
</h3>

<p>Let's build a simple assistant agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">Microsoft.Agents.AI</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Microsoft.Extensions.AI</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Azure.Identity</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Azure.AI.Inference</span><span class="p">;</span>

<span class="c1">// Create the chat client</span>
<span class="n">IChatClient</span> <span class="n">chatClient</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatCompletionsClient</span><span class="p">(</span>
    <span class="k">new</span> <span class="nf">Uri</span><span class="p">(</span><span class="n">Environment</span><span class="p">.</span><span class="nf">GetEnvironmentVariable</span><span class="p">(</span><span class="s">"AZURE_OPENAI_ENDPOINT"</span><span class="p">)!),</span>
    <span class="k">new</span> <span class="nf">DefaultAzureCredential</span><span class="p">()</span>
<span class="p">).</span><span class="nf">AsChatClient</span><span class="p">(</span><span class="s">"gpt-4o"</span><span class="p">);</span>

<span class="c1">// Create an agent</span>
<span class="kt">var</span> <span class="n">agent</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatClientAgent</span><span class="p">(</span><span class="n">chatClient</span><span class="p">,</span> <span class="k">new</span> <span class="n">ChatClientAgentOptions</span>
<span class="p">{</span>
    <span class="n">Name</span> <span class="p">=</span> <span class="s">"Assistant"</span><span class="p">,</span>
    <span class="n">Instructions</span> <span class="p">=</span> <span class="s">"""
</span>        <span class="n">You</span> <span class="n">are</span> <span class="n">a</span> <span class="n">helpful</span> <span class="n">AI</span> <span class="n">assistant</span><span class="p">.</span> <span class="n">You</span> <span class="n">provide</span> <span class="n">clear</span><span class="p">,</span> 
        <span class="n">accurate</span> <span class="n">answers</span> <span class="k">and</span> <span class="n">ask</span> <span class="n">clarifying</span> <span class="n">questions</span> <span class="k">when</span> <span class="n">needed</span><span class="p">.</span>
        <span class="s">"""
</span><span class="p">});</span>

<span class="c1">// Invoke the agent</span>
<span class="kt">var</span> <span class="n">response</span> <span class="p">=</span> <span class="k">await</span> <span class="n">agent</span><span class="p">.</span><span class="nf">InvokeAsync</span><span class="p">(</span>
    <span class="s">"What are the key differences between async and parallel programming in C#?"</span><span class="p">);</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">Content</span><span class="p">);</span>
</code></pre>

</div>



<p>This looks deceptively simple, but there's a lot happening under the hood:</p>

<ul>
<li>The agent manages conversation context</li>
<li>Token counting and context window management is automatic</li>
<li>The response includes usage metrics for cost tracking</li>
</ul>

<h3>
  
  
  Adding Tools
</h3>

<p>Agents become powerful when they can take actions. Let's add some tools:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System.ComponentModel</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">CalculatorTools</span>
<span class="p">{</span>
    <span class="p">[</span><span class="n">AgentTool</span><span class="p">]</span>
    <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"Adds two numbers together"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="kt">double</span> <span class="nf">Add</span><span class="p">(</span>
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"The first number"</span><span class="p">)]</span> <span class="kt">double</span> <span class="n">a</span><span class="p">,</span> 
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"The second number"</span><span class="p">)]</span> <span class="kt">double</span> <span class="n">b</span><span class="p">)</span> 
        <span class="p">=&gt;</span> <span class="n">a</span> <span class="p">+</span> <span class="n">b</span><span class="p">;</span>

    <span class="p">[</span><span class="n">AgentTool</span><span class="p">]</span>
    <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"Multiplies two numbers"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="kt">double</span> <span class="nf">Multiply</span><span class="p">(</span>
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"The first number"</span><span class="p">)]</span> <span class="kt">double</span> <span class="n">a</span><span class="p">,</span> 
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"The second number"</span><span class="p">)]</span> <span class="kt">double</span> <span class="n">b</span><span class="p">)</span> 
        <span class="p">=&gt;</span> <span class="n">a</span> <span class="p">*</span> <span class="n">b</span><span class="p">;</span>

    <span class="p">[</span><span class="n">AgentTool</span><span class="p">]</span>
    <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"Calculates the square root of a number"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="kt">double</span> <span class="nf">SquareRoot</span><span class="p">(</span>
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"The number to calculate the square root of"</span><span class="p">)]</span> <span class="kt">double</span> <span class="n">n</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">n</span> <span class="p">&lt;</span> <span class="m">0</span><span class="p">)</span>
            <span class="k">throw</span> <span class="k">new</span> <span class="nf">ArgumentException</span><span class="p">(</span><span class="s">"Cannot calculate square root of negative number"</span><span class="p">);</span>
        <span class="k">return</span> <span class="n">Math</span><span class="p">.</span><span class="nf">Sqrt</span><span class="p">(</span><span class="n">n</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now register the tools with the agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">agent</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatClientAgent</span><span class="p">(</span><span class="n">chatClient</span><span class="p">,</span> <span class="k">new</span> <span class="n">ChatClientAgentOptions</span>
<span class="p">{</span>
    <span class="n">Name</span> <span class="p">=</span> <span class="s">"MathAssistant"</span><span class="p">,</span>
    <span class="n">Instructions</span> <span class="p">=</span> <span class="s">"You are a math assistant. Use the provided tools to perform calculations."</span>
<span class="p">});</span>

<span class="c1">// Add tools from a class</span>
<span class="n">agent</span><span class="p">.</span><span class="n">AddTools</span><span class="p">&lt;</span><span class="n">CalculatorTools</span><span class="p">&gt;();</span>

<span class="c1">// Or add individual functions</span>
<span class="n">agent</span><span class="p">.</span><span class="nf">AddTool</span><span class="p">(</span>
    <span class="s">"get_current_time"</span><span class="p">,</span>
    <span class="s">"Gets the current UTC time"</span><span class="p">,</span>
    <span class="p">()</span> <span class="p">=&gt;</span> <span class="n">DateTime</span><span class="p">.</span><span class="n">UtcNow</span><span class="p">.</span><span class="nf">ToString</span><span class="p">(</span><span class="s">"O"</span><span class="p">));</span>

<span class="c1">// Now the agent can use these tools</span>
<span class="kt">var</span> <span class="n">response</span> <span class="p">=</span> <span class="k">await</span> <span class="n">agent</span><span class="p">.</span><span class="nf">InvokeAsync</span><span class="p">(</span>
    <span class="s">"What is the square root of 144, multiplied by 3?"</span><span class="p">);</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">Content</span><span class="p">);</span>
<span class="c1">// Output: The square root of 144 is 12, multiplied by 3 equals 36.</span>
</code></pre>

</div>



<h3>
  
  
  Async Tools and Dependency Injection
</h3>

<p>Real-world tools often need to call external services:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">WeatherTools</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">IWeatherService</span> <span class="n">_weatherService</span><span class="p">;</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">ILogger</span><span class="p">&lt;</span><span class="n">WeatherTools</span><span class="p">&gt;</span> <span class="n">_logger</span><span class="p">;</span>

    <span class="k">public</span> <span class="nf">WeatherTools</span><span class="p">(</span>
        <span class="n">IWeatherService</span> <span class="n">weatherService</span><span class="p">,</span> 
        <span class="n">ILogger</span><span class="p">&lt;</span><span class="n">WeatherTools</span><span class="p">&gt;</span> <span class="n">logger</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">_weatherService</span> <span class="p">=</span> <span class="n">weatherService</span><span class="p">;</span>
        <span class="n">_logger</span> <span class="p">=</span> <span class="n">logger</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="n">AgentTool</span><span class="p">]</span>
    <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"Gets the current weather for a city"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">WeatherInfo</span><span class="p">&gt;</span> <span class="nf">GetCurrentWeather</span><span class="p">(</span>
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"The city name, e.g., 'Seattle' or 'London, UK'"</span><span class="p">)]</span> 
        <span class="kt">string</span> <span class="n">city</span><span class="p">,</span>
        <span class="n">CancellationToken</span> <span class="n">cancellationToken</span> <span class="p">=</span> <span class="k">default</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">_logger</span><span class="p">.</span><span class="nf">LogInformation</span><span class="p">(</span><span class="s">"Fetching weather for {City}"</span><span class="p">,</span> <span class="n">city</span><span class="p">);</span>
        <span class="k">return</span> <span class="k">await</span> <span class="n">_weatherService</span><span class="p">.</span><span class="nf">GetCurrentAsync</span><span class="p">(</span><span class="n">city</span><span class="p">,</span> <span class="n">cancellationToken</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="p">[</span><span class="n">AgentTool</span><span class="p">]</span>
    <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"Gets the weather forecast for the next N days"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">IReadOnlyList</span><span class="p">&lt;</span><span class="n">WeatherForecast</span><span class="p">&gt;&gt;</span> <span class="nf">GetForecast</span><span class="p">(</span>
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"The city name"</span><span class="p">)]</span> <span class="kt">string</span> <span class="n">city</span><span class="p">,</span>
        <span class="p">[</span><span class="nf">Description</span><span class="p">(</span><span class="s">"Number of days to forecast (1-7)"</span><span class="p">)]</span> <span class="kt">int</span> <span class="n">days</span> <span class="p">=</span> <span class="m">3</span><span class="p">,</span>
        <span class="n">CancellationToken</span> <span class="n">cancellationToken</span> <span class="p">=</span> <span class="k">default</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">days</span> <span class="p">=</span> <span class="n">Math</span><span class="p">.</span><span class="nf">Clamp</span><span class="p">(</span><span class="n">days</span><span class="p">,</span> <span class="m">1</span><span class="p">,</span> <span class="m">7</span><span class="p">);</span>
        <span class="k">return</span> <span class="k">await</span> <span class="n">_weatherService</span><span class="p">.</span><span class="nf">GetForecastAsync</span><span class="p">(</span><span class="n">city</span><span class="p">,</span> <span class="n">days</span><span class="p">,</span> <span class="n">cancellationToken</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// With dependency injection</span>
<span class="kt">var</span> <span class="n">services</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ServiceCollection</span><span class="p">()</span>
    <span class="p">.</span><span class="n">AddSingleton</span><span class="p">&lt;</span><span class="n">IWeatherService</span><span class="p">,</span> <span class="n">OpenMeteoWeatherService</span><span class="p">&gt;()</span>
    <span class="p">.</span><span class="nf">AddLogging</span><span class="p">()</span>
    <span class="p">.</span><span class="nf">BuildServiceProvider</span><span class="p">();</span>

<span class="kt">var</span> <span class="n">weatherTools</span> <span class="p">=</span> <span class="n">ActivatorUtilities</span><span class="p">.</span><span class="n">CreateInstance</span><span class="p">&lt;</span><span class="n">WeatherTools</span><span class="p">&gt;(</span><span class="n">services</span><span class="p">);</span>
<span class="n">agent</span><span class="p">.</span><span class="nf">AddTools</span><span class="p">(</span><span class="n">weatherTools</span><span class="p">);</span>
</code></pre>

</div>






<h2>
  
  
  Conversation Management
</h2>

<p>Agents maintain conversation history automatically, but you have full control:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Create a conversation</span>
<span class="kt">var</span> <span class="n">conversation</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">AgentConversation</span><span class="p">();</span>

<span class="c1">// Add messages</span>
<span class="n">conversation</span><span class="p">.</span><span class="nf">AddUserMessage</span><span class="p">(</span><span class="s">"My name is Alex."</span><span class="p">);</span>
<span class="kt">var</span> <span class="n">response1</span> <span class="p">=</span> <span class="k">await</span> <span class="n">agent</span><span class="p">.</span><span class="nf">InvokeAsync</span><span class="p">(</span><span class="n">conversation</span><span class="p">);</span>
<span class="n">conversation</span><span class="p">.</span><span class="nf">AddAssistantMessage</span><span class="p">(</span><span class="n">response1</span><span class="p">);</span>

<span class="n">conversation</span><span class="p">.</span><span class="nf">AddUserMessage</span><span class="p">(</span><span class="s">"What's my name?"</span><span class="p">);</span>
<span class="kt">var</span> <span class="n">response2</span> <span class="p">=</span> <span class="k">await</span> <span class="n">agent</span><span class="p">.</span><span class="nf">InvokeAsync</span><span class="p">(</span><span class="n">conversation</span><span class="p">);</span>
<span class="c1">// The agent remembers: "Your name is Alex."</span>

<span class="c1">// Inspect the conversation</span>
<span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">message</span> <span class="k">in</span> <span class="n">conversation</span><span class="p">.</span><span class="n">Messages</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"[</span><span class="p">{</span><span class="n">message</span><span class="p">.</span><span class="n">Role</span><span class="p">}</span><span class="s">]: </span><span class="p">{</span><span class="n">message</span><span class="p">.</span><span class="n">Content</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
<span class="p">}</span>

<span class="c1">// Serialize for persistence</span>
<span class="kt">var</span> <span class="n">json</span> <span class="p">=</span> <span class="n">JsonSerializer</span><span class="p">.</span><span class="nf">Serialize</span><span class="p">(</span><span class="n">conversation</span><span class="p">);</span>

<span class="c1">// Clear or trim as needed</span>
<span class="n">conversation</span><span class="p">.</span><span class="nf">TrimToTokenLimit</span><span class="p">(</span><span class="n">maxTokens</span><span class="p">:</span> <span class="m">4000</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  System Messages and Context
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">agent</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatClientAgent</span><span class="p">(</span><span class="n">chatClient</span><span class="p">,</span> <span class="k">new</span> <span class="n">ChatClientAgentOptions</span>
<span class="p">{</span>
    <span class="n">Name</span> <span class="p">=</span> <span class="s">"CustomerSupport"</span><span class="p">,</span>
    <span class="n">Instructions</span> <span class="p">=</span> <span class="s">"""
</span>        <span class="n">You</span> <span class="n">are</span> <span class="n">a</span> <span class="n">customer</span> <span class="n">support</span> <span class="n">agent</span> <span class="k">for</span> <span class="n">TechCorp</span><span class="p">.</span>

        <span class="n">Company</span> <span class="n">policies</span><span class="p">:</span>
        <span class="p">-</span> <span class="n">Refunds</span> <span class="n">are</span> <span class="n">available</span> <span class="n">within</span> <span class="m">30</span> <span class="n">days</span>
        <span class="p">-</span> <span class="n">Premium</span> <span class="n">members</span> <span class="k">get</span> <span class="n">priority</span> <span class="n">support</span>
        <span class="p">-</span> <span class="n">Never</span> <span class="n">share</span> <span class="k">internal</span> <span class="n">pricing</span> <span class="n">formulas</span>

        <span class="n">Always</span> <span class="n">be</span> <span class="n">polite</span> <span class="k">and</span> <span class="n">professional</span><span class="p">.</span>
        <span class="s">""",
</span>
    <span class="c1">// Additional context injection</span>
    <span class="n">ContextProvider</span> <span class="p">=</span> <span class="k">async</span> <span class="p">(</span><span class="n">conversation</span><span class="p">,</span> <span class="n">ct</span><span class="p">)</span> <span class="p">=&gt;</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">userId</span> <span class="p">=</span> <span class="n">conversation</span><span class="p">.</span><span class="n">Metadata</span><span class="p">.</span><span class="nf">GetValueOrDefault</span><span class="p">(</span><span class="s">"userId"</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">userId</span> <span class="k">is</span> <span class="kt">string</span> <span class="n">id</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="kt">var</span> <span class="n">user</span> <span class="p">=</span> <span class="k">await</span> <span class="n">userService</span><span class="p">.</span><span class="nf">GetAsync</span><span class="p">(</span><span class="n">id</span><span class="p">,</span> <span class="n">ct</span><span class="p">);</span>
            <span class="k">return</span> <span class="s">$"Current user: </span><span class="p">{</span><span class="n">user</span><span class="p">.</span><span class="n">Name</span><span class="p">}</span><span class="s">, Tier: </span><span class="p">{</span><span class="n">user</span><span class="p">.</span><span class="n">Tier</span><span class="p">}</span><span class="s">, Member since: </span><span class="p">{</span><span class="n">user</span><span class="p">.</span><span class="n">CreatedAt</span><span class="p">:</span><span class="n">yyyy</span><span class="p">}</span><span class="s">"</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="k">null</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Migration from Semantic Kernel
</h2>

<p>If you're coming from Semantic Kernel, the migration is straightforward. Most patterns transfer directly:</p>

<h3>
  
  
  Before (Semantic Kernel)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Semantic Kernel approach</span>
<span class="kt">var</span> <span class="n">kernel</span> <span class="p">=</span> <span class="n">Kernel</span><span class="p">.</span><span class="nf">CreateBuilder</span><span class="p">()</span>
    <span class="p">.</span><span class="nf">AddAzureOpenAIChatCompletion</span><span class="p">(</span>
        <span class="n">deploymentName</span><span class="p">:</span> <span class="s">"gpt-4o"</span><span class="p">,</span>
        <span class="n">endpoint</span><span class="p">:</span> <span class="n">config</span><span class="p">[</span><span class="s">"AzureOpenAI:Endpoint"</span><span class="p">]!,</span>
        <span class="n">credentials</span><span class="p">:</span> <span class="k">new</span> <span class="nf">DefaultAzureCredential</span><span class="p">())</span>
    <span class="p">.</span><span class="nf">Build</span><span class="p">();</span>

<span class="n">kernel</span><span class="p">.</span><span class="n">Plugins</span><span class="p">.</span><span class="n">AddFromType</span><span class="p">&lt;</span><span class="n">MyPlugin</span><span class="p">&gt;();</span>

<span class="kt">var</span> <span class="n">agent</span> <span class="p">=</span> <span class="k">new</span> <span class="n">ChatCompletionAgent</span>
<span class="p">{</span>
    <span class="n">Kernel</span> <span class="p">=</span> <span class="n">kernel</span><span class="p">,</span>
    <span class="n">Name</span> <span class="p">=</span> <span class="s">"Assistant"</span><span class="p">,</span>
    <span class="n">Instructions</span> <span class="p">=</span> <span class="s">"You are helpful."</span>
<span class="p">};</span>

<span class="kt">var</span> <span class="n">history</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatHistory</span><span class="p">();</span>
<span class="n">history</span><span class="p">.</span><span class="nf">AddUserMessage</span><span class="p">(</span><span class="s">"Hello!"</span><span class="p">);</span>
<span class="k">await</span> <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">content</span> <span class="k">in</span> <span class="n">agent</span><span class="p">.</span><span class="nf">InvokeStreamingAsync</span><span class="p">(</span><span class="n">history</span><span class="p">))</span>
<span class="p">{</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">Write</span><span class="p">(</span><span class="n">content</span><span class="p">.</span><span class="n">Content</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  After (Agent Framework)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Agent Framework approach</span>
<span class="n">IChatClient</span> <span class="n">chatClient</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatCompletionsClient</span><span class="p">(</span>
    <span class="k">new</span> <span class="nf">Uri</span><span class="p">(</span><span class="n">config</span><span class="p">[</span><span class="s">"AzureOpenAI:Endpoint"</span><span class="p">]!),</span>
    <span class="k">new</span> <span class="nf">DefaultAzureCredential</span><span class="p">()</span>
<span class="p">).</span><span class="nf">AsChatClient</span><span class="p">(</span><span class="s">"gpt-4o"</span><span class="p">);</span>

<span class="kt">var</span> <span class="n">agent</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatClientAgent</span><span class="p">(</span><span class="n">chatClient</span><span class="p">,</span> <span class="k">new</span> <span class="n">ChatClientAgentOptions</span>
<span class="p">{</span>
    <span class="n">Name</span> <span class="p">=</span> <span class="s">"Assistant"</span><span class="p">,</span>
    <span class="n">Instructions</span> <span class="p">=</span> <span class="s">"You are helpful."</span>
<span class="p">});</span>

<span class="n">agent</span><span class="p">.</span><span class="n">AddTools</span><span class="p">&lt;</span><span class="n">MyPlugin</span><span class="p">&gt;();</span> <span class="c1">// Same plugin, renamed attribute</span>

<span class="kt">var</span> <span class="n">conversation</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">AgentConversation</span><span class="p">();</span>
<span class="n">conversation</span><span class="p">.</span><span class="nf">AddUserMessage</span><span class="p">(</span><span class="s">"Hello!"</span><span class="p">);</span>
<span class="k">await</span> <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">chunk</span> <span class="k">in</span> <span class="n">agent</span><span class="p">.</span><span class="nf">InvokeStreamingAsync</span><span class="p">(</span><span class="n">conversation</span><span class="p">))</span>
<span class="p">{</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">Write</span><span class="p">(</span><span class="n">chunk</span><span class="p">.</span><span class="n">Content</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Key Differences
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Semantic Kernel</th>
<th>Agent Framework</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>Kernel</code></td>
<td><code>IChatClient</code></td>
<td>Uses M.E.AI abstractions</td>
</tr>
<tr>
<td><code>KernelPlugin</code></td>
<td><code>AgentTool</code></td>
<td>Attribute renamed</td>
</tr>
<tr>
<td><code>ChatHistory</code></td>
<td><code>AgentConversation</code></td>
<td>Similar API, more features</td>
</tr>
<tr>
<td><code>ChatCompletionAgent</code></td>
<td><code>ChatClientAgent</code></td>
<td>Simplified construction</td>
</tr>
<tr>
<td><code>kernel.InvokeAsync&lt;T&gt;</code></td>
<td><code>agent.InvokeAsync</code></td>
<td>More consistent API</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  What Still Works
</h3>

<ul>
<li>Your existing plugins work with minimal changes (rename <code>KernelFunction</code> to <code>AgentTool</code>)</li>
<li>Dependency injection patterns are identical</li>
<li>Memory connectors are available through adapters</li>
<li>OpenTelemetry integration is enhanced, not replaced</li>
</ul>




<h2>
  
  
  Migration from AutoGen
</h2>

<p>AutoGen developers will find familiar multi-agent patterns:</p>

<h3>
  
  
  Before (AutoGen)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># AutoGen (Python)
</span><span class="n">assistant</span> <span class="o">=</span> <span class="nc">AssistantAgent</span><span class="p">(</span><span class="sh">"</span><span class="s">assistant</span><span class="sh">"</span><span class="p">,</span> <span class="n">llm_config</span><span class="o">=</span><span class="n">llm_config</span><span class="p">)</span>
<span class="n">user_proxy</span> <span class="o">=</span> <span class="nc">UserProxyAgent</span><span class="p">(</span><span class="sh">"</span><span class="s">user_proxy</span><span class="sh">"</span><span class="p">)</span>

<span class="n">user_proxy</span><span class="p">.</span><span class="nf">initiate_chat</span><span class="p">(</span><span class="n">assistant</span><span class="p">,</span> <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">Hello!</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  After (Agent Framework)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Agent Framework (C#)</span>
<span class="kt">var</span> <span class="n">assistant</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">ChatClientAgent</span><span class="p">(</span><span class="n">chatClient</span><span class="p">,</span> <span class="k">new</span> <span class="n">ChatClientAgentOptions</span>
<span class="p">{</span>
    <span class="n">Name</span> <span class="p">=</span> <span class="s">"Assistant"</span><span class="p">,</span>
    <span class="n">Instructions</span> <span class="p">=</span> <span class="s">"You are a helpful assistant."</span>
<span class="p">});</span>

<span class="kt">var</span> <span class="n">userProxy</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">UserProxyAgent</span><span class="p">(</span><span class="k">new</span> <span class="n">UserProxyAgentOptions</span>
<span class="p">{</span>
    <span class="n">Name</span> <span class="p">=</span> <span class="s">"User"</span><span class="p">,</span>
    <span class="n">HumanInputMode</span> <span class="p">=</span> <span class="n">HumanInputMode</span><span class="p">.</span><span class="n">Always</span>
<span class="p">});</span>

<span class="kt">var</span> <span class="n">chat</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">RoundRobinGroupChat</span><span class="p">(</span><span class="k">new</span><span class="p">[]</span> <span class="p">{</span> <span class="n">userProxy</span><span class="p">,</span> <span class="n">assistant</span> <span class="p">});</span>
<span class="k">await</span> <span class="n">chat</span><span class="p">.</span><span class="nf">RunAsync</span><span class="p">(</span><span class="s">"Hello!"</span><span class="p">);</span>
</code></pre>

</div>



<p>AutoGen's orchestration patterns are now first-class citizens in Agent Framework, with added type safety and production-ready durability.</p>




<h2>
  
  
  What's Next in This Series
</h2>

<p>In the upcoming articles, we'll dive deeper:</p>

<p><strong>Part 2: Workflows</strong> — Learn how to orchestrate complex multi-step tasks with explicit workflows, conditional branching, and checkpointing.</p>

<p><strong>Part 3: MCP Integration</strong> — Build and consume Model Context Protocol tools for cross-platform interoperability.</p>

<p><strong>Part 4: Production Deployment</strong> — Deploy to Azure AI Foundry with observability, scaling, and security best practices.</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li><a href="https://learn.microsoft.com/en-us/semantic-kernel/agents/" rel="noopener noreferrer">Microsoft Agent Framework Documentation</a></li>
<li><a href="https://github.com/microsoft/semantic-kernel" rel="noopener noreferrer">GitHub Repository</a></li>
<li><a href="https://devblogs.microsoft.com/dotnet/introducing-microsoft-extensions-ai/" rel="noopener noreferrer">Microsoft.Extensions.AI</a></li>
<li><a href="https://learn.microsoft.com/en-us/semantic-kernel/agents/migration" rel="noopener noreferrer">Migration Guide from Semantic Kernel</a></li>
</ul>




<h2>
  
  
  Conclusion
</h2>

<p>Microsoft Agent Framework represents a maturation of .NET's AI capabilities. By unifying Semantic Kernel's production-ready foundations with AutoGen's research-driven patterns, we finally have a comprehensive toolkit for building everything from simple chatbots to complex multi-agent systems.</p>

<p>The migration path is smooth, the patterns are familiar, and the future is unified. Whether you're starting fresh or migrating existing projects, now is the perfect time to embrace Agent Framework.</p>

<p><em>In Part 2, we'll explore Workflows—the powerful orchestration engine that enables complex, durable, multi-step agent tasks.</em></p>

