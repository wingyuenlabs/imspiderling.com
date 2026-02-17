---
Title: Azure AI Agent Service: Your First Production-Ready AI Agent in C#
Description: 
Author: Brian Spann
Date: 2026-02-17T22:02:21.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Azure AI Agent Service: Your First Production-Ready AI Agent in C
</h1>

<p>Building AI agents from scratch is hard. You need to manage conversation state, handle tool calling, implement streaming, deal with rate limits, and somehow make it all production-ready. What if Microsoft had already solved most of these problems for you?</p>

<p>That's exactly what <strong>Azure AI Agent Service</strong> offers—a managed platform for building, deploying, and running AI agents in production. In this series, we'll take you from your first agent to a fully orchestrated multi-agent system deployed on Azure AI Foundry.</p>

<h2>
  
  
  What You'll Learn in This Series
</h2>

<ol>
<li>
<strong>Part 1 (this article)</strong>: Introduction to Azure AI Agent Service and your first agent</li>
<li>
<strong>Part 2</strong>: Extending agents with tools, function calling, and file search</li>
<li>
<strong>Part 3</strong>: Multi-agent orchestration with Semantic Kernel</li>
<li>
<strong>Part 4</strong>: Production patterns—state management, sessions, and observability</li>
<li>
<strong>Part 5</strong>: Deploying multi-agent systems to Azure AI Foundry</li>
</ol>

<p>Let's dive in.</p>




<h2>
  
  
  Why Azure AI Agent Service?
</h2>

<p>If you've built AI applications before, you've probably used the Azure OpenAI SDK or similar libraries to make direct completions calls. That works great for simple Q&amp;A scenarios, but agents are different. They need:</p>

<ul>
<li>
<strong>Persistent conversation state</strong> across multiple turns</li>
<li>
<strong>Tool execution</strong> with the ability to call external functions</li>
<li>
<strong>Streaming responses</strong> for better user experience</li>
<li>
<strong>Automatic context management</strong> to handle long conversations</li>
<li>
<strong>Built-in capabilities</strong> like code interpretation and file search</li>
</ul>

<p>Building all of this yourself is a significant undertaking. Azure AI Agent Service provides these capabilities out of the box, letting you focus on what makes your agent unique.</p>

<h3>
  
  
  Azure AI Agent Service vs. Building from Scratch
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concern</th>
<th>From Scratch</th>
<th>Azure AI Agent Service</th>
</tr>
</thead>
<tbody>
<tr>
<td>Conversation state</td>
<td>You build it</td>
<td>Managed threads</td>
</tr>
<tr>
<td>Tool calling loop</td>
<td>You implement it</td>
<td>Automatic with runs</td>
</tr>
<tr>
<td>Code execution</td>
<td>You sandbox it</td>
<td>Code Interpreter built-in</td>
</tr>
<tr>
<td>File search/RAG</td>
<td>You build vectors</td>
<td>Vector stores included</td>
</tr>
<tr>
<td>Streaming</td>
<td>You handle chunks</td>
<td>Native support</td>
</tr>
<tr>
<td>Scaling</td>
<td>Your infrastructure</td>
<td>Azure-managed</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  The Core Architecture
</h2>

<p>Before writing code, let's understand the key concepts:</p>

<h3>
  
  
  Agents
</h3>

<p>An <strong>Agent</strong> is your AI entity with a specific persona and capabilities. It has:</p>

<ul>
<li>A <strong>model</strong> (like <code>gpt-4o</code>) that powers its reasoning</li>
<li>
<strong>Instructions</strong> that define its behavior and personality</li>
<li>
<strong>Tools</strong> it can use (functions, code interpreter, file search)</li>
</ul>

<p>Agents are persistent resources—you create them once and reuse them across many conversations.</p>

<h3>
  
  
  Threads
</h3>

<p>A <strong>Thread</strong> represents a conversation. It's the container for all messages between users and agents. Threads:</p>

<ul>
<li>Persist automatically (no database needed on your side)</li>
<li>Support multiple messages from both users and assistants</li>
<li>Can be shared across agent invocations</li>
</ul>

<h3>
  
  
  Messages
</h3>

<p><strong>Messages</strong> are the individual turns in a conversation. Each message has:</p>

<ul>
<li>A <strong>role</strong> (user, assistant, or tool)</li>
<li>
<strong>Content</strong> (text, images, or file references)</li>
<li>
<strong>Metadata</strong> for your application needs</li>
</ul>

<h3>
  
  
  Runs
</h3>

<p>A <strong>Run</strong> is a single execution of an agent against a thread. When you create a run:</p>

<ol>
<li>The agent reads all messages in the thread</li>
<li>It generates a response (potentially calling tools)</li>
<li>The response is automatically added to the thread</li>
</ol>

<p>Runs can be synchronous, asynchronous, or streamed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────────┐
│                     Azure AI Foundry Project                │
│  ┌─────────────┐                                            │
│  │   Agent     │ ──── model: gpt-4o                         │
│  │             │ ──── instructions: "You are..."            │
│  │             │ ──── tools: [function, code_interpreter]   │
│  └─────────────┘                                            │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐     ┌──────────────────────┐               │
│  │   Thread    │────▶│     Messages         │               │
│  └─────────────┘     │ • User: "Help me..." │               │
│         │            │ • Assistant: "Sure!" │               │
│         ▼            │ • User: "Also..."    │               │
│  ┌─────────────┐     └──────────────────────┘               │
│  │    Run      │                                            │
│  │  (in_progress)                                           │
│  └─────────────┘                                            │
└─────────────────────────────────────────────────────────────┘
</code></pre>

</div>






<h2>
  
  
  Setting Up Your Environment
</h2>

<h3>
  
  
  Prerequisites
</h3>

<ol>
<li>
<strong>Azure subscription</strong> with access to Azure AI services</li>
<li>
<strong>.NET 8 SDK</strong> or later</li>
<li>
<strong>Azure CLI</strong> installed and logged in</li>
<li>An <strong>Azure AI Foundry project</strong> (we'll create this)</li>
</ol>

<h3>
  
  
  Creating an Azure AI Foundry Project
</h3>

<p>Azure AI Foundry is the umbrella platform for Azure AI services. Your agents live inside a Foundry project.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Login to Azure</span>
az login

<span class="c"># Create a resource group</span>
az group create <span class="nt">--name</span> rg-ai-agents <span class="nt">--location</span> eastus2

<span class="c"># Create an Azure AI Foundry hub</span>
az ml workspace create <span class="se">\</span>
  <span class="nt">--name</span> hub-ai-agents <span class="se">\</span>
  <span class="nt">--resource-group</span> rg-ai-agents <span class="se">\</span>
  <span class="nt">--kind</span> hub

<span class="c"># Create a project within the hub</span>
az ml workspace create <span class="se">\</span>
  <span class="nt">--name</span> project-customer-support <span class="se">\</span>
  <span class="nt">--resource-group</span> rg-ai-agents <span class="se">\</span>
  <span class="nt">--kind</span> project <span class="se">\</span>
  <span class="nt">--hub-id</span> /subscriptions/<span class="o">{</span>sub-id<span class="o">}</span>/resourceGroups/rg-ai-agents/providers/Microsoft.MachineLearningServices/workspaces/hub-ai-agents
</code></pre>

</div>



<p>Once created, get your project endpoint from the Azure Portal (it looks like <code>https://{region}.api.azureml.ms/agents/v1.0/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.MachineLearningServices/workspaces/{project}</code>).</p>

<h3>
  
  
  Installing the SDK
</h3>

<p>Create a new .NET project and add the Azure.AI.Projects package:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>dotnet new console <span class="nt">-n</span> AgentDemo
<span class="nb">cd </span>AgentDemo
dotnet add package Azure.AI.Projects <span class="nt">--prerelease</span>
dotnet add package Azure.Identity
</code></pre>

</div>



<blockquote>
<p><strong>Note</strong>: As of early 2025, the Azure.AI.Projects package may still be in preview. Check NuGet for the latest stable version.</p>
</blockquote>




<h2>
  
  
  Your First Agent: A Customer Support Bot
</h2>

<p>Let's build a customer support agent that can answer questions about orders. We'll start simple and add capabilities throughout this series.</p>

<h3>
  
  
  Step 1: Initialize the Project Client
</h3>

<p>The <code>AIProjectClient</code> is your gateway to Azure AI Agent Service:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">Azure</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Azure.AI.Projects</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Azure.Identity</span><span class="p">;</span>

<span class="c1">// Store your endpoint in environment variables (never hardcode!)</span>
<span class="kt">var</span> <span class="n">endpoint</span> <span class="p">=</span> <span class="n">Environment</span><span class="p">.</span><span class="nf">GetEnvironmentVariable</span><span class="p">(</span><span class="s">"AZURE_AI_FOUNDRY_PROJECT_ENDPOINT"</span><span class="p">)</span>
    <span class="p">??</span> <span class="k">throw</span> <span class="k">new</span> <span class="nf">InvalidOperationException</span><span class="p">(</span><span class="s">"Set AZURE_AI_FOUNDRY_PROJECT_ENDPOINT"</span><span class="p">);</span>

<span class="c1">// Use DefaultAzureCredential for seamless local dev + production auth</span>
<span class="kt">var</span> <span class="n">credential</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">DefaultAzureCredential</span><span class="p">();</span>

<span class="kt">var</span> <span class="n">projectClient</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">AIProjectClient</span><span class="p">(</span><span class="k">new</span> <span class="nf">Uri</span><span class="p">(</span><span class="n">endpoint</span><span class="p">),</span> <span class="n">credential</span><span class="p">);</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">"✅ Connected to Azure AI Foundry project"</span><span class="p">);</span>
</code></pre>

</div>



<p><code>DefaultAzureCredential</code> automatically handles authentication:</p>

<ul>
<li>
<strong>Locally</strong>: Uses your Azure CLI login, Visual Studio credentials, or environment variables</li>
<li>
<strong>In Azure</strong>: Uses Managed Identity (no secrets to manage!)</li>
</ul>

<h3>
  
  
  Step 2: Create an Agent
</h3>

<p>Now let's create our customer support agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Define the agent's persona and capabilities</span>
<span class="kt">var</span> <span class="n">agentOptions</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">CreateAgentOptions</span><span class="p">(</span>
    <span class="n">model</span><span class="p">:</span> <span class="s">"gpt-4o"</span><span class="p">,</span>  <span class="c1">// Or your deployed model name</span>
    <span class="n">name</span><span class="p">:</span> <span class="s">"CustomerSupportAgent"</span><span class="p">,</span>
    <span class="n">instructions</span><span class="p">:</span> <span class="s">"""
</span>        <span class="n">You</span> <span class="n">are</span> <span class="n">a</span> <span class="n">helpful</span> <span class="n">customer</span> <span class="n">support</span> <span class="n">agent</span> <span class="k">for</span> <span class="n">Contoso</span> <span class="n">Electronics</span><span class="p">.</span>

        <span class="n">Your</span> <span class="n">responsibilities</span><span class="p">:</span>
        <span class="p">-</span> <span class="n">Answer</span> <span class="n">questions</span> <span class="n">about</span> <span class="n">orders</span><span class="p">,</span> <span class="n">shipping</span><span class="p">,</span> <span class="k">and</span> <span class="n">returns</span>
        <span class="p">-</span> <span class="n">Provide</span> <span class="n">product</span> <span class="n">information</span> <span class="k">and</span> <span class="n">recommendations</span>
        <span class="p">-</span> <span class="n">Escalate</span> <span class="n">complex</span> <span class="n">issues</span> <span class="n">to</span> <span class="n">human</span> <span class="n">support</span> <span class="k">when</span> <span class="n">needed</span>

        <span class="n">Guidelines</span><span class="p">:</span>
        <span class="p">-</span> <span class="n">Be</span> <span class="n">friendly</span><span class="p">,</span> <span class="n">professional</span><span class="p">,</span> <span class="k">and</span> <span class="n">concise</span>
        <span class="p">-</span> <span class="n">If</span> <span class="n">you</span> <span class="n">don</span><span class="err">'</span><span class="n">t</span> <span class="n">know</span> <span class="n">something</span><span class="p">,</span> <span class="n">say</span> <span class="n">so</span> <span class="n">honestly</span>
        <span class="p">-</span> <span class="n">Always</span> <span class="n">confirm</span> <span class="n">order</span> <span class="n">numbers</span> <span class="n">before</span> <span class="n">providing</span> <span class="n">order</span><span class="p">-</span><span class="n">specific</span> <span class="n">info</span>
        <span class="p">-</span> <span class="n">For</span> <span class="n">refund</span> <span class="n">requests</span> <span class="n">over</span> <span class="err">$</span><span class="m">500</span><span class="p">,</span> <span class="n">recommend</span> <span class="n">speaking</span> <span class="n">to</span> <span class="n">a</span> <span class="n">supervisor</span>

        <span class="n">Important</span><span class="p">:</span> <span class="n">Never</span> <span class="n">share</span> <span class="k">internal</span> <span class="n">policies</span> <span class="k">or</span> <span class="n">speculate</span> <span class="n">about</span> <span class="n">inventory</span><span class="p">.</span>
        <span class="s">"""
</span><span class="p">);</span>

<span class="kt">var</span> <span class="n">agent</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateAgentAsync</span><span class="p">(</span><span class="n">agentOptions</span><span class="p">);</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"✅ Created agent: </span><span class="p">{</span><span class="n">agent</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Name</span><span class="p">}</span><span class="s"> (ID: </span><span class="p">{</span><span class="n">agent</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">}</span><span class="s">)"</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>A Note on Instructions</strong>: Your instructions are crucial. They define:</p>

<ul>
<li>
<strong>Persona</strong>: Who is this agent?</li>
<li>
<strong>Scope</strong>: What can/can't it help with?</li>
<li>
<strong>Guidelines</strong>: How should it behave?</li>
<li>
<strong>Guardrails</strong>: What are the limits?</li>
</ul>

<p>Spend time crafting good instructions—they're the foundation of agent quality.</p>

<h3>
  
  
  Step 3: Create a Thread
</h3>

<p>Threads hold the conversation history. Create one per user session:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">thread</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateThreadAsync</span><span class="p">();</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"✅ Created thread: </span><span class="p">{</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
</code></pre>

</div>



<p>In a real application, you'd store the thread ID and reuse it when the same user returns. We'll cover session management patterns in Part 4.</p>

<h3>
  
  
  Step 4: Add a User Message
</h3>

<p>Let's ask our agent a question:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">userMessage</span> <span class="p">=</span> <span class="s">"Hi! I placed an order last week but haven't received any shipping updates. My order number is ORD-12345. Can you help?"</span><span class="p">;</span>

<span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateMessageAsync</span><span class="p">(</span>
    <span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span>
    <span class="n">MessageRole</span><span class="p">.</span><span class="n">User</span><span class="p">,</span>
    <span class="n">userMessage</span>
<span class="p">);</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"📤 User: </span><span class="p">{</span><span class="n">userMessage</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Step 5: Run the Agent
</h3>

<p>Now we execute the agent against the thread:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Create a run - this triggers the agent to process all messages and respond</span>
<span class="kt">var</span> <span class="n">run</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateRunAsync</span><span class="p">(</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> <span class="n">agent</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>

<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"🔄 Run started: </span><span class="p">{</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">}</span><span class="s"> (Status: </span><span class="p">{</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span><span class="p">}</span><span class="s">)"</span><span class="p">);</span>

<span class="c1">// Poll until the run completes</span>
<span class="k">while</span> <span class="p">(</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span> <span class="p">==</span> <span class="n">RunStatus</span><span class="p">.</span><span class="n">Queued</span> <span class="p">||</span> <span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span> <span class="p">==</span> <span class="n">RunStatus</span><span class="p">.</span><span class="n">InProgress</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">await</span> <span class="n">Task</span><span class="p">.</span><span class="nf">Delay</span><span class="p">(</span><span class="m">500</span><span class="p">);</span>  <span class="c1">// Don't hammer the API</span>
    <span class="n">run</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">GetRunAsync</span><span class="p">(</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> <span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"   Status: </span><span class="p">{</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">if</span> <span class="p">(</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span> <span class="p">==</span> <span class="n">RunStatus</span><span class="p">.</span><span class="n">Completed</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">"✅ Run completed!"</span><span class="p">);</span>
<span class="p">}</span>
<span class="k">else</span>
<span class="p">{</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"❌ Run failed with status: </span><span class="p">{</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"   Error: </span><span class="p">{</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">LastError</span><span class="p">?.</span><span class="n">Message</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Step 6: Get the Response
</h3>

<p>Retrieve the messages to see what the agent said:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">messages</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">GetMessagesAsync</span><span class="p">(</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>

<span class="c1">// Messages are returned in reverse chronological order</span>
<span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">message</span> <span class="k">in</span> <span class="n">messages</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Data</span><span class="p">.</span><span class="nf">OrderBy</span><span class="p">(</span><span class="n">m</span> <span class="p">=&gt;</span> <span class="n">m</span><span class="p">.</span><span class="n">CreatedAt</span><span class="p">))</span>
<span class="p">{</span>
    <span class="kt">var</span> <span class="n">role</span> <span class="p">=</span> <span class="n">message</span><span class="p">.</span><span class="n">Role</span> <span class="p">==</span> <span class="n">MessageRole</span><span class="p">.</span><span class="n">User</span> <span class="p">?</span> <span class="s">"👤 User"</span> <span class="p">:</span> <span class="s">"🤖 Agent"</span><span class="p">;</span>

    <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">content</span> <span class="k">in</span> <span class="n">message</span><span class="p">.</span><span class="n">ContentItems</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">content</span> <span class="k">is</span> <span class="n">MessageTextContent</span> <span class="n">textContent</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"</span><span class="p">{</span><span class="n">role</span><span class="p">}</span><span class="s">: </span><span class="p">{</span><span class="n">textContent</span><span class="p">.</span><span class="n">Text</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Putting It All Together
</h3>

<p>Here's the complete first agent implementation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">Azure</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Azure.AI.Projects</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Azure.Identity</span><span class="p">;</span>

<span class="c1">// Configuration</span>
<span class="kt">var</span> <span class="n">endpoint</span> <span class="p">=</span> <span class="n">Environment</span><span class="p">.</span><span class="nf">GetEnvironmentVariable</span><span class="p">(</span><span class="s">"AZURE_AI_FOUNDRY_PROJECT_ENDPOINT"</span><span class="p">)</span>
    <span class="p">??</span> <span class="k">throw</span> <span class="k">new</span> <span class="nf">InvalidOperationException</span><span class="p">(</span><span class="s">"Set AZURE_AI_FOUNDRY_PROJECT_ENDPOINT"</span><span class="p">);</span>

<span class="kt">var</span> <span class="n">projectClient</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">AIProjectClient</span><span class="p">(</span><span class="k">new</span> <span class="nf">Uri</span><span class="p">(</span><span class="n">endpoint</span><span class="p">),</span> <span class="k">new</span> <span class="nf">DefaultAzureCredential</span><span class="p">());</span>

<span class="k">try</span>
<span class="p">{</span>
    <span class="c1">// Create the agent</span>
    <span class="kt">var</span> <span class="n">agent</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateAgentAsync</span><span class="p">(</span><span class="k">new</span> <span class="nf">CreateAgentOptions</span><span class="p">(</span>
        <span class="n">model</span><span class="p">:</span> <span class="s">"gpt-4o"</span><span class="p">,</span>
        <span class="n">name</span><span class="p">:</span> <span class="s">"CustomerSupportAgent"</span><span class="p">,</span>
        <span class="n">instructions</span><span class="p">:</span> <span class="s">"""
</span>            <span class="n">You</span> <span class="n">are</span> <span class="n">a</span> <span class="n">helpful</span> <span class="n">customer</span> <span class="n">support</span> <span class="n">agent</span> <span class="k">for</span> <span class="n">Contoso</span> <span class="n">Electronics</span><span class="p">.</span>
            <span class="n">Be</span> <span class="n">friendly</span><span class="p">,</span> <span class="n">professional</span><span class="p">,</span> <span class="k">and</span> <span class="n">concise</span><span class="p">.</span>
            <span class="n">If</span> <span class="n">you</span> <span class="n">don</span><span class="err">'</span><span class="n">t</span> <span class="n">have</span> <span class="n">order</span> <span class="n">lookup</span> <span class="n">capabilities</span> <span class="n">yet</span><span class="p">,</span> <span class="k">let</span> <span class="n">the</span> <span class="n">user</span> <span class="n">know</span>
            <span class="k">and</span> <span class="n">offer</span> <span class="n">to</span> <span class="n">help</span> <span class="k">with</span> <span class="n">general</span> <span class="n">questions</span><span class="p">.</span>
            <span class="s">"""
</span>    <span class="p">));</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"✅ Agent created: </span><span class="p">{</span><span class="n">agent</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>

    <span class="c1">// Create a conversation thread</span>
    <span class="kt">var</span> <span class="n">thread</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateThreadAsync</span><span class="p">();</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"✅ Thread created: </span><span class="p">{</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>

    <span class="c1">// Add user message</span>
    <span class="kt">var</span> <span class="n">userInput</span> <span class="p">=</span> <span class="s">"Hi! I need help with my order ORD-12345"</span><span class="p">;</span>
    <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateMessageAsync</span><span class="p">(</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> <span class="n">MessageRole</span><span class="p">.</span><span class="n">User</span><span class="p">,</span> <span class="n">userInput</span><span class="p">);</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"📤 User: </span><span class="p">{</span><span class="n">userInput</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>

    <span class="c1">// Run the agent</span>
    <span class="kt">var</span> <span class="n">run</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateRunAsync</span><span class="p">(</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> <span class="n">agent</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>

    <span class="k">while</span> <span class="p">(</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span> <span class="p">==</span> <span class="n">RunStatus</span><span class="p">.</span><span class="n">Queued</span> <span class="p">||</span> <span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span> <span class="p">==</span> <span class="n">RunStatus</span><span class="p">.</span><span class="n">InProgress</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">await</span> <span class="n">Task</span><span class="p">.</span><span class="nf">Delay</span><span class="p">(</span><span class="m">500</span><span class="p">);</span>
        <span class="n">run</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">GetRunAsync</span><span class="p">(</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> <span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c1">// Get and display response</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">run</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span> <span class="p">==</span> <span class="n">RunStatus</span><span class="p">.</span><span class="n">Completed</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">messages</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">GetMessagesAsync</span><span class="p">(</span><span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>
        <span class="kt">var</span> <span class="n">assistantMessage</span> <span class="p">=</span> <span class="n">messages</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Data</span>
            <span class="p">.</span><span class="nf">Where</span><span class="p">(</span><span class="n">m</span> <span class="p">=&gt;</span> <span class="n">m</span><span class="p">.</span><span class="n">Role</span> <span class="p">==</span> <span class="n">MessageRole</span><span class="p">.</span><span class="n">Assistant</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">OrderByDescending</span><span class="p">(</span><span class="n">m</span> <span class="p">=&gt;</span> <span class="n">m</span><span class="p">.</span><span class="n">CreatedAt</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">First</span><span class="p">();</span>

        <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">content</span> <span class="k">in</span> <span class="n">assistantMessage</span><span class="p">.</span><span class="n">ContentItems</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">content</span> <span class="k">is</span> <span class="n">MessageTextContent</span> <span class="n">textContent</span><span class="p">)</span>
            <span class="p">{</span>
                <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"🤖 Agent: </span><span class="p">{</span><span class="n">textContent</span><span class="p">.</span><span class="n">Text</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="c1">// Cleanup: delete the agent when done (optional for demos)</span>
    <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">DeleteAgentAsync</span><span class="p">(</span><span class="n">agent</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">"🧹 Agent deleted"</span><span class="p">);</span>
<span class="p">}</span>
<span class="k">catch</span> <span class="p">(</span><span class="n">RequestFailedException</span> <span class="n">ex</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"Azure error: </span><span class="p">{</span><span class="n">ex</span><span class="p">.</span><span class="n">Message</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"Error code: </span><span class="p">{</span><span class="n">ex</span><span class="p">.</span><span class="n">ErrorCode</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Streaming Responses
</h2>

<p>Polling works, but for a better user experience, you'll want streaming. This shows words as they're generated:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Create a streaming run</span>
<span class="k">await</span> <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">update</span> <span class="k">in</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">CreateRunStreamingAsync</span><span class="p">(</span>
    <span class="n">thread</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> 
    <span class="n">agent</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Id</span><span class="p">))</span>
<span class="p">{</span>
    <span class="k">switch</span> <span class="p">(</span><span class="n">update</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">case</span> <span class="n">RunUpdate</span> <span class="n">runUpdate</span><span class="p">:</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"Run status: </span><span class="p">{</span><span class="n">runUpdate</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Status</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
            <span class="k">break</span><span class="p">;</span>

        <span class="k">case</span> <span class="n">MessageContentUpdate</span> <span class="n">contentUpdate</span><span class="p">:</span>
            <span class="c1">// This fires for each chunk of text</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">Write</span><span class="p">(</span><span class="n">contentUpdate</span><span class="p">.</span><span class="n">Text</span><span class="p">);</span>
            <span class="k">break</span><span class="p">;</span>

        <span class="k">case</span> <span class="n">RunStepUpdate</span> <span class="n">stepUpdate</span><span class="p">:</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"\nStep: </span><span class="p">{</span><span class="n">stepUpdate</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Type</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
            <span class="k">break</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">();</span>  <span class="c1">// Final newline</span>
</code></pre>

</div>



<p>Streaming provides a much more responsive feel—users see the agent "thinking" in real-time rather than waiting for a complete response.</p>




<h2>
  
  
  Agent Lifecycle Management
</h2>

<p>In production, you need to think about agent lifecycle:</p>

<h3>
  
  
  Creating Agents
</h3>

<ul>
<li>
<strong>Startup creation</strong>: Create agents when your service starts, reuse across requests</li>
<li>
<strong>On-demand creation</strong>: Create specialized agents per use case</li>
<li>
<strong>Hybrid</strong>: Create base agents at startup, clone and customize for specific needs
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">AgentFactory</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">AIProjectClient</span> <span class="n">_client</span><span class="p">;</span>
    <span class="k">private</span> <span class="n">Agent</span><span class="p">?</span> <span class="n">_supportAgent</span><span class="p">;</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">Agent</span><span class="p">&gt;</span> <span class="nf">GetSupportAgentAsync</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">_supportAgent</span> <span class="p">==</span> <span class="k">null</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="kt">var</span> <span class="n">response</span> <span class="p">=</span> <span class="k">await</span> <span class="n">_client</span><span class="p">.</span><span class="nf">CreateAgentAsync</span><span class="p">(</span><span class="k">new</span> <span class="nf">CreateAgentOptions</span><span class="p">(</span>
                <span class="n">model</span><span class="p">:</span> <span class="s">"gpt-4o"</span><span class="p">,</span>
                <span class="n">name</span><span class="p">:</span> <span class="s">$"SupportAgent-</span><span class="p">{</span><span class="n">Guid</span><span class="p">.</span><span class="nf">NewGuid</span><span class="p">():</span><span class="n">N</span><span class="p">}</span><span class="s">"</span><span class="p">,</span>
                <span class="n">instructions</span><span class="p">:</span> <span class="s">"...support agent instructions..."</span>
            <span class="p">));</span>
            <span class="n">_supportAgent</span> <span class="p">=</span> <span class="n">response</span><span class="p">.</span><span class="n">Value</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="n">_supportAgent</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Cleaning Up
</h3>

<p>Agents persist until deleted. In development, always clean up:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Delete a specific agent</span>
<span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">DeleteAgentAsync</span><span class="p">(</span><span class="n">agentId</span><span class="p">);</span>

<span class="c1">// List and delete all your agents (careful in production!)</span>
<span class="kt">var</span> <span class="n">agents</span> <span class="p">=</span> <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">GetAgentsAsync</span><span class="p">();</span>
<span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">agent</span> <span class="k">in</span> <span class="n">agents</span><span class="p">.</span><span class="n">Value</span><span class="p">.</span><span class="n">Data</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"Deleting: </span><span class="p">{</span><span class="n">agent</span><span class="p">.</span><span class="n">Name</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
    <span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">DeleteAgentAsync</span><span class="p">(</span><span class="n">agent</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Thread Management
</h3>

<p>Threads also persist. For long-running applications:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Delete threads when conversations end</span>
<span class="k">await</span> <span class="n">projectClient</span><span class="p">.</span><span class="nf">DeleteThreadAsync</span><span class="p">(</span><span class="n">threadId</span><span class="p">);</span>

<span class="c1">// Or set up automatic cleanup for old threads</span>
<span class="c1">// (You'll need to track thread ages yourself)</span>
</code></pre>

</div>






<h2>
  
  
  When to Use Azure AI Agent Service
</h2>

<p>Azure AI Agent Service shines when you need:</p>

<p>✅ <strong>Multi-turn conversations</strong> with automatic state management<br><br>
✅ <strong>Tool execution</strong> (function calling, code interpreter, file search)<br><br>
✅ <strong>Quick prototyping</strong> of agent experiences<br><br>
✅ <strong>Azure-native deployment</strong> with managed scaling<br><br>
✅ <strong>Enterprise features</strong> (authentication, audit logs, compliance)  </p>

<h3>
  
  
  When to Consider Alternatives
</h3>

<p>❌ <strong>Simple completion requests</strong> — Use Azure OpenAI SDK directly<br><br>
❌ <strong>Non-Azure infrastructure</strong> — Consider OpenAI's Assistants API or build custom<br><br>
❌ <strong>Maximum control needed</strong> — Build on Microsoft Agent Framework or Semantic Kernel directly<br><br>
❌ <strong>Latency-critical applications</strong> — Direct model calls have less overhead  </p>




<h2>
  
  
  What's Next
</h2>

<p>We've built a basic agent, but it can't actually look up orders—it just pretends! In <strong>Part 2</strong>, we'll fix that by adding:</p>

<ul>
<li>
<strong>Function calling</strong> to connect to your order database</li>
<li>
<strong>Code Interpreter</strong> for data analysis tasks</li>
<li>
<strong>File Search</strong> to answer questions from documentation</li>
</ul>

<p>You'll learn how to give your agent real capabilities that make it genuinely useful.</p>




<h2>
  
  
  Summary
</h2>

<p>In this article, we:</p>

<ul>
<li>Explored <strong>why Azure AI Agent Service</strong> exists and when to use it</li>
<li>Understood the <strong>core architecture</strong>: Agents, Threads, Messages, and Runs</li>
<li>
<strong>Set up an Azure AI Foundry project</strong> and installed the SDK</li>
<li>Built a <strong>complete customer support agent</strong> from scratch</li>
<li>Implemented both <strong>polling and streaming</strong> response patterns</li>
<li>Discussed <strong>lifecycle management</strong> for production scenarios</li>
</ul>

<p>The foundation is laid. Your first agent is running. Now let's make it powerful.</p>




<p><strong>Resources:</strong></p>

<ul>
<li><a href="https://learn.microsoft.com/azure/ai-services/agents/" rel="noopener noreferrer">Azure AI Agent Service documentation</a></li>
<li><a href="https://learn.microsoft.com/dotnet/api/azure.ai.projects" rel="noopener noreferrer">Azure.AI.Projects SDK reference</a></li>
<li><a href="https://ai.azure.com" rel="noopener noreferrer">Azure AI Foundry portal</a></li>
</ul>

<p><em>Next up: Part 2 — Power Your Azure AI Agents: Function Calling, Code Interpreter, and File Search</em></p>

