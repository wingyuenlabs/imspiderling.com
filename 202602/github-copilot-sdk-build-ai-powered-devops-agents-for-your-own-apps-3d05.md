---
Title: GitHub Copilot SDK - Build AI-Powered DevOps Agents for Your Own Apps
Description: 
Author: Marcel.L
Date: 2026-02-17T22:01:27.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  GitHub Copilot SDK: Build AI-Powered DevOps Agents for Your Own Apps
</h2>

<p>GitHub Copilot has steadily evolved from an in-editor autocomplete tool into a full-blown agentic platform. The latest step in that journey is the <strong>GitHub Copilot SDK</strong>, now available in <strong>Technical Preview</strong>. It lets you embed the same agent runtime that powers Copilot CLI directly into your own applications, scripts, and services using <strong>Python</strong>, <strong>TypeScript/Node.js</strong>, <strong>Go</strong>, or <strong>.NET</strong>.</p>

<p>For DevOps engineers, this opens up a powerful new pattern: instead of relying solely on Copilot inside your IDE or through GitHub Issues, you can now build <strong>custom AI agents</strong> that plug into your existing operational tooling. Think incident response bots, infrastructure validators, deployment assistants, and compliance checkers, all powered by the Copilot engine and your own custom tools.</p>

<p>In this post we will cover what the SDK is, how it works, how to get started, and walk through six practical DevOps use cases, several drawn from real open-source projects you can explore today.</p>




<h2>
  
  
  What Is the GitHub Copilot SDK?
</h2>

<p>The <a href="https://github.com/github/copilot-sdk" rel="noopener noreferrer">GitHub Copilot SDK</a> is a set of multi-platform libraries that expose the <strong>Copilot CLI agent runtime</strong> as a programmable interface. Rather than building your own LLM orchestration layer, you get the same production-tested engine that handles planning, tool invocation, file edits, and multi-turn conversations.</p>

<h3>
  
  
  Key Facts
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Detail</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Repository</strong></td>
<td><a href="https://github.com/github/copilot-sdk" rel="noopener noreferrer">github/copilot-sdk</a></td>
</tr>
<tr>
<td><strong>Status</strong></td>
<td>Technical Preview</td>
</tr>
<tr>
<td><strong>Languages</strong></td>
<td>TypeScript/Node.js, Python, Go, .NET</td>
</tr>
<tr>
<td><strong>Licence</strong></td>
<td>MIT</td>
</tr>
<tr>
<td><strong>Auth</strong></td>
<td>GitHub OAuth, environment variables (<code>GITHUB_TOKEN</code>), or BYOK</td>
</tr>
<tr>
<td><strong>Billing</strong></td>
<td>Counts against your Copilot premium request quota</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Architecture
</h3>

<p>All four SDKs share the same communication model. Your application talks to the SDK client, which communicates with the Copilot CLI running in server mode over <strong>JSON-RPC</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Your Application
       ↓
  SDK Client
       ↓  JSON-RPC
  Copilot CLI (server mode)
</code></pre>

</div>


<p>The SDK <strong>manages the CLI process lifecycle automatically</strong> by default. You can also run the CLI externally in headless server mode and point multiple SDK clients at it, which is useful for shared development environments or debugging.</p>
<h3>
  
  
  What You Can Do
</h3>

<ul>
<li>
<strong>Send prompts</strong> and receive responses (streaming or complete).</li>
<li>
<strong>Define custom tools</strong> that the agent can call, with typed parameters and handler functions you control.</li>
<li>
<strong>Connect to MCP servers</strong> (Model Context Protocol) to give the agent access to external services like the GitHub API, databases, or cloud provider tooling.</li>
<li>
<strong>Create custom agents</strong> with specialised personas, system messages, and tool sets.</li>
<li>
<strong>Use any supported model</strong>, including those available through Copilot or your own keys via BYOK (Bring Your Own Key) from providers like OpenAI, Azure AI Foundry, and Anthropic.</li>
</ul>


<h2>
  
  
  Getting Started
</h2>
<h3>
  
  
  Prerequisites
</h3>

<ol>
<li>
<strong>GitHub Copilot subscription</strong> (Free tier with limited usage, or Pro/Pro+/Business/Enterprise). If using BYOK, no GitHub auth is required.</li>
<li>
<strong>Copilot CLI</strong> installed and authenticated. Follow the <a href="https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli" rel="noopener noreferrer">Copilot CLI installation guide</a>.</li>
<li>Your preferred language runtime: <strong>Node.js 18+</strong>, <strong>Python 3.8+</strong>, <strong>Go 1.21+</strong>, or <strong>.NET 8.0+</strong>.</li>
</ol>

<p>Verify the CLI is working:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>copilot <span class="nt">--version</span>
</code></pre>

</div>

<h3>
  
  
  Install the SDK
</h3>

<p>Pick your language:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Node.js / TypeScript</span>
npm <span class="nb">install</span> @github/copilot-sdk

<span class="c"># Python</span>
pip <span class="nb">install </span>github-copilot-sdk

<span class="c"># Go</span>
go get github.com/github/copilot-sdk/go

<span class="c"># .NET</span>
dotnet add package GitHub.Copilot.SDK
</code></pre>

</div>

<h3>
  
  
  Your First Five Lines
</h3>

<p>Here is the simplest possible example in TypeScript. It creates a client, opens a session, sends a prompt, and prints the response:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">CopilotClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@github/copilot-sdk</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CopilotClient</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">createSession</span><span class="p">({</span> <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gpt-4.1</span><span class="dl">'</span> <span class="p">});</span>

<span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">session</span><span class="p">.</span><span class="nf">sendAndWait</span><span class="p">({</span>
  <span class="na">prompt</span><span class="p">:</span> <span class="dl">'</span><span class="s1">What is 2 + 2?</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">response</span><span class="p">?.</span><span class="nx">data</span><span class="p">.</span><span class="nx">content</span><span class="p">);</span>

<span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">stop</span><span class="p">();</span>
<span class="nx">process</span><span class="p">.</span><span class="nf">exit</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
</code></pre>

</div>


<p>And the Python equivalent:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">github_copilot_sdk</span> <span class="kn">import</span> <span class="n">CopilotClient</span>

<span class="n">client</span> <span class="o">=</span> <span class="nc">CopilotClient</span><span class="p">()</span>
<span class="n">session</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="nf">create_session</span><span class="p">(</span><span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gpt-4.1</span><span class="sh">"</span><span class="p">)</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">session</span><span class="p">.</span><span class="nf">send_and_wait</span><span class="p">(</span><span class="n">prompt</span><span class="o">=</span><span class="sh">"</span><span class="s">What is 2 + 2?</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">data</span><span class="p">.</span><span class="n">content</span><span class="p">)</span>

<span class="n">client</span><span class="p">.</span><span class="nf">stop</span><span class="p">()</span>
</code></pre>

</div>


<p>That is it. Five lines to get a Copilot-powered response in your own application.</p>


<h2>
  
  
  Adding Streaming and Custom Tools
</h2>

<p>The real power of the SDK emerges when you add <strong>streaming responses</strong> and <strong>custom tools</strong>. Streaming gives your users real-time feedback while the agent works. Custom tools let the agent call functions you define, bridging the gap between the LLM and your operational systems.</p>
<h3>
  
  
  Streaming Example (TypeScript)
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">CopilotClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@github/copilot-sdk</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CopilotClient</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">createSession</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gpt-4.1</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">streaming</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
<span class="p">});</span>

<span class="nx">session</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">assistant.message_delta</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">process</span><span class="p">.</span><span class="nx">stdout</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">deltaContent</span><span class="p">);</span>
<span class="p">});</span>
<span class="nx">session</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">session.idle</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">();</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">session</span><span class="p">.</span><span class="nf">sendAndWait</span><span class="p">({</span>
  <span class="na">prompt</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Explain blue-green deployments in three sentences.</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">stop</span><span class="p">();</span>
<span class="nx">process</span><span class="p">.</span><span class="nf">exit</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
</code></pre>

</div>

<h3>
  
  
  Custom Tool Example (TypeScript)
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">CopilotClient</span><span class="p">,</span> <span class="nx">defineTool</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@github/copilot-sdk</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">checkPodStatus</span> <span class="o">=</span> <span class="nf">defineTool</span><span class="p">(</span><span class="dl">'</span><span class="s1">check_pod_status</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Check the status of Kubernetes pods in a namespace</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">parameters</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">object</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">properties</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">namespace</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">The Kubernetes namespace</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">},</span>
    <span class="na">required</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">namespace</span><span class="dl">'</span><span class="p">],</span>
  <span class="p">},</span>
  <span class="na">handler</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="na">args</span><span class="p">:</span> <span class="p">{</span> <span class="na">namespace</span><span class="p">:</span> <span class="kr">string</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// In a real scenario, call kubectl or the Kubernetes API here</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">namespace</span><span class="p">:</span> <span class="nx">args</span><span class="p">.</span><span class="k">namespace</span><span class="p">,</span>
      <span class="na">pods</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">api-server-1</span><span class="dl">'</span><span class="p">,</span> <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Running</span><span class="dl">'</span><span class="p">,</span> <span class="na">restarts</span><span class="p">:</span> <span class="mi">0</span> <span class="p">},</span>
        <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">api-server-2</span><span class="dl">'</span><span class="p">,</span> <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CrashLoopBackOff</span><span class="dl">'</span><span class="p">,</span> <span class="na">restarts</span><span class="p">:</span> <span class="mi">12</span> <span class="p">},</span>
        <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">worker-1</span><span class="dl">'</span><span class="p">,</span> <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Running</span><span class="dl">'</span><span class="p">,</span> <span class="na">restarts</span><span class="p">:</span> <span class="mi">0</span> <span class="p">},</span>
      <span class="p">],</span>
    <span class="p">};</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>


<p>When the agent receives a question like "Are there any unhealthy pods in the production namespace?", it decides to call your <code>check_pod_status</code> tool, receives the result, and incorporates it into a natural-language response.</p>


<h2>
  
  
  Connecting to MCP Servers
</h2>

<p>The SDK supports <a href="https://modelcontextprotocol.io/" rel="noopener noreferrer">Model Context Protocol (MCP)</a> servers out of the box. MCP provides a standardised way for AI agents to interact with external tools and data sources. This is especially relevant for DevOps because you can connect the agent to GitHub, cloud providers, databases, and monitoring APIs without writing custom tool handlers for each one.</p>

<p>For example, connecting to GitHub's MCP server gives the agent access to repositories, issues, pull requests, and more:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">createSession</span><span class="p">({</span>
  <span class="na">mcpServers</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">github</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">http</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://api.githubcopilot.com/mcp/</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>


<p>You can also connect to local MCP servers. For example, an Azure Bicep schema server:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"AzureBicep"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"local"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npx"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"args"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"-y"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"@azure/mcp@latest"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"server"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"start"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"--namespace"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"bicepschema"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"--read-only"</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>


<p>The <a href="https://github.com/modelcontextprotocol/servers" rel="noopener noreferrer">MCP Servers Directory</a> maintains a growing catalogue of community servers, including integrations for Terraform, Docker, Prometheus, and many other DevOps-adjacent tools.</p>


<h2>
  
  
  DevOps Use Cases
</h2>

<p>The SDK is still young, but the community is already building real tools with it. Here are six use cases, several drawn from actual open-source projects and the official cookbook, that show what is possible for DevOps teams.</p>
<h3>
  
  
  Use Case 1: Autonomous SRE Agent for GitHub Actions
</h3>

<p><strong>Real project</strong>: <a href="https://github.com/htekdev/github-sre-agent" rel="noopener noreferrer">htekdev/github-sre-agent</a></p>

<p>This open-source project is a fully autonomous SRE agent built with the Copilot SDK. It listens for GitHub Actions webhook events and, when a workflow fails, it:</p>

<ol>
<li>
<strong>Fetches and analyses logs</strong> via the GitHub MCP server.</li>
<li>
<strong>Checks GitHub system status</strong> to rule out platform outages.</li>
<li>
<strong>Searches the web</strong> for known fixes using the Exa AI MCP server.</li>
<li>
<strong>Makes an intelligent decision</strong>: retry a transient failure, create a detailed issue for a genuine bug, or skip if the failure is expected.</li>
<li>
<strong>Tracks resolution</strong>: when a previously failed workflow succeeds, it automatically closes the related issue.</li>
</ol>

<p>The architecture is clean and worth studying. It uses the Copilot SDK's <code>createSession</code> with two MCP servers (GitHub and Exa AI) plus custom tools for status checking, note-taking, and workflow tracking. Repository-level configuration lives in <code>.github/sre-agent.yml</code>:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">version</span><span class="pi">:</span> <span class="m">1</span>
<span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
<span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
  <span class="s">- This repo uses pnpm, not npm</span>
  <span class="s">- Always check if tests pass before suggesting retry</span>
  <span class="s">- Create issues with label "ci-failure" for tracking</span>
<span class="na">actions</span><span class="pi">:</span>
  <span class="na">retry</span><span class="pi">:</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
    <span class="na">maxAttempts</span><span class="pi">:</span> <span class="m">3</span>
  <span class="na">createIssue</span><span class="pi">:</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">sre-agent</span>
      <span class="pi">-</span> <span class="s">automated</span>
      <span class="pi">-</span> <span class="s">ci-failure</span>
</code></pre>

</div>


<p>This is a strong example of how the SDK can replace manual on-call triage for CI/CD failures.</p>


<h3>
  
  
  Use Case 2: Repository Health Analysis (RepoCheckAI)
</h3>

<p><strong>Real project</strong>: <a href="https://github.com/glaucia86/repocheckai" rel="noopener noreferrer">glaucia86/repocheckai</a> (70+ stars)</p>

<p>RepoCheckAI (formerly Repo Doctor) is an agentic CLI tool built with the Copilot SDK that performs comprehensive health checks across six areas: <strong>documentation</strong>, <strong>developer experience</strong>, <strong>CI/CD</strong>, <strong>testing</strong>, <strong>governance</strong>, and <strong>security</strong>. It delivers a health score (0-100%), prioritised findings (P0/P1/P2), and actionable remediation steps with code snippets.</p>

<p>It offers two analysis modes:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Mode</th>
<th>How it works</th>
<th>Best for</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Quick Scan</strong></td>
<td>Analyses via GitHub API (up to 20 file reads)</td>
<td>Governance review, quick checks</td>
</tr>
<tr>
<td><strong>Deep Analysis</strong></td>
<td>Full source scan using Repomix</td>
<td>Code quality, architecture review</td>
</tr>
</tbody>
</table></div>

<p>The killer feature for DevOps teams is the <code>--issue</code> flag. After analysis, it automatically creates structured GitHub Issues for each problem found, complete with priority labels, impact assessments, and fix instructions:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>repocheck analyze your-org/your-repo <span class="nt">--issue</span>

<span class="c"># Creates:</span>
<span class="c"># 🔴 [RepoCheckAI] docs: Missing README</span>
<span class="c"># 🟠 [RepoCheckAI] ci: No CI/CD Pipeline</span>
<span class="c"># 🟡 [RepoCheckAI] dx: Code Quality Issues</span>
</code></pre>

</div>


<p>This pattern, using the SDK to audit repositories and create actionable issues, is directly applicable to platform engineering teams managing dozens of microservice repos.</p>


<h3>
  
  
  Use Case 3: Autonomous Coding Loops (Ralph Loop Pattern)
</h3>

<p><strong>Source</strong>: <a href="https://github.com/github/awesome-copilot/blob/main/cookbook/copilot-sdk/nodejs/ralph-loop.md" rel="noopener noreferrer">Official Copilot SDK Cookbook</a></p>

<p>The Ralph Loop is an autonomous development pattern from the SDK cookbook that is particularly powerful for DevOps automation. The concept: an AI agent iterates through tasks in isolated context windows, with state persisted on disk between iterations. Each loop creates a <strong>fresh session</strong>, reads the current state, does one task, writes results back, and exits.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">readFile</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs/promises</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CopilotClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@github/copilot-sdk</span><span class="dl">'</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">ralphLoop</span><span class="p">(</span><span class="nx">promptFile</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">maxIterations</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">50</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CopilotClient</span><span class="p">();</span>
  <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">start</span><span class="p">();</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">prompt</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">readFile</span><span class="p">(</span><span class="nx">promptFile</span><span class="p">,</span> <span class="dl">'</span><span class="s1">utf-8</span><span class="dl">'</span><span class="p">);</span>

    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;=</span> <span class="nx">maxIterations</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`\n=== Iteration </span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">maxIterations</span><span class="p">}</span><span class="s2"> ===`</span><span class="p">);</span>

      <span class="c1">// Fresh session each iteration. Context isolation is the point</span>
      <span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">createSession</span><span class="p">({</span>
        <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gpt-4.1</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">workingDirectory</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nf">cwd</span><span class="p">(),</span>
        <span class="na">onPermissionRequest</span><span class="p">:</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="na">allow</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}),</span>
      <span class="p">});</span>

      <span class="k">try</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">session</span><span class="p">.</span><span class="nf">sendAndWait</span><span class="p">({</span> <span class="nx">prompt</span> <span class="p">},</span> <span class="mi">600</span><span class="nx">_000</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">session</span><span class="p">.</span><span class="nf">destroy</span><span class="p">();</span>
      <span class="p">}</span>

      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Iteration </span><span class="p">${</span><span class="nx">i</span><span class="p">}</span><span class="s2"> complete.`</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">stop</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nf">ralphLoop</span><span class="p">(</span><span class="dl">'</span><span class="s1">PROMPT.md</span><span class="dl">'</span><span class="p">,</span> <span class="mi">20</span><span class="p">);</span>
</code></pre>

</div>


<p>For DevOps, imagine pointing this at an <code>IMPLEMENTATION_PLAN.md</code> that lists infrastructure tasks: "add monitoring to service X", "update Terraform module Y to v2", "write integration tests for pipeline Z". The agent picks the next task, implements it, runs tests, commits, and moves on. The key principles:</p>

<ul>
<li>
<strong>Fresh context per iteration</strong> prevents context window degradation.</li>
<li>
<strong>Disk as shared state</strong> (<code>IMPLEMENTATION_PLAN.md</code>) coordinates between iterations.</li>
<li>
<strong>Backpressure</strong> (tests, builds, lints) ensures quality, the agent must pass them before committing.</li>
</ul>

<p>This is ideal for burning down infrastructure debt or implementing a batch of related IaC changes unattended.</p>


<h3>
  
  
  Use Case 4: Incident Response with PagerDuty and Datadog
</h3>

<p><strong>Source</strong>: <a href="https://github.com/microsoft/copilot-sdk-samples" rel="noopener noreferrer">microsoft/copilot-sdk-samples</a></p>

<p>Microsoft's official sample repository includes dedicated <strong>PagerDuty</strong> and <strong>Datadog</strong> connector samples that demonstrate how to build incident management and monitoring agents. All connectors support a <strong>mock-first</strong> design, so you can develop and test without live credentials.</p>

<p>Here is how you might wire up an incident response agent that combines both:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">CopilotClient</span><span class="p">,</span> <span class="nx">defineTool</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@github/copilot-sdk</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Tools backed by your PagerDuty and Datadog connectors</span>
<span class="kd">const</span> <span class="nx">getActiveIncidents</span> <span class="o">=</span> <span class="nf">defineTool</span><span class="p">(</span><span class="dl">'</span><span class="s1">get_active_incidents</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">List active PagerDuty incidents for a service</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">parameters</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">object</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">properties</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">service</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">,</span> <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Service name</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">},</span>
    <span class="na">required</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">service</span><span class="dl">'</span><span class="p">],</span>
  <span class="p">},</span>
  <span class="na">handler</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="na">args</span><span class="p">:</span> <span class="p">{</span> <span class="na">service</span><span class="p">:</span> <span class="kr">string</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// In production, call PagerDuty API</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">incidents</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">PD-4521</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">title</span><span class="p">:</span> <span class="dl">'</span><span class="s1">High error rate on payments-api</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">severity</span><span class="p">:</span> <span class="dl">'</span><span class="s1">P1</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">triggered</span><span class="p">:</span> <span class="dl">'</span><span class="s1">2026-02-15T14:32:00Z</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">assignee</span><span class="p">:</span> <span class="dl">'</span><span class="s1">on-call-team</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">],</span>
    <span class="p">};</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">queryMonitoring</span> <span class="o">=</span> <span class="nf">defineTool</span><span class="p">(</span><span class="dl">'</span><span class="s1">query_monitoring</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Query Datadog metrics and logs</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">parameters</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">object</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">properties</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">query</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">,</span> <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Datadog query string</span><span class="dl">'</span> <span class="p">},</span>
      <span class="na">timeRange</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">,</span> <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Time range (e.g. last_1h)</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">},</span>
    <span class="na">required</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">query</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">timeRange</span><span class="dl">'</span><span class="p">],</span>
  <span class="p">},</span>
  <span class="na">handler</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="na">args</span><span class="p">:</span> <span class="p">{</span> <span class="na">query</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">timeRange</span><span class="p">:</span> <span class="kr">string</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// In production, call Datadog API</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">metrics</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">errorRate</span><span class="p">:</span> <span class="dl">'</span><span class="s1">12.4%</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">p99Latency</span><span class="p">:</span> <span class="dl">'</span><span class="s1">2340ms</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">requestsPerSecond</span><span class="p">:</span> <span class="mi">890</span><span class="p">,</span>
      <span class="p">},</span>
      <span class="na">recentLogs</span><span class="p">:</span> <span class="p">[</span>
        <span class="dl">'</span><span class="s1">ERROR: Connection pool exhausted for database replica-02</span><span class="dl">'</span><span class="p">,</span>
        <span class="dl">'</span><span class="s1">WARN: Retry limit exceeded for downstream service auth-api</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">],</span>
    <span class="p">};</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CopilotClient</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">createSession</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gpt-4.1</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">streaming</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">tools</span><span class="p">:</span> <span class="p">[</span><span class="nx">getActiveIncidents</span><span class="p">,</span> <span class="nx">queryMonitoring</span><span class="p">],</span>
  <span class="na">systemMessage</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">content</span><span class="p">:</span>
      <span class="dl">'</span><span class="s1">You are an incident response assistant. When asked about an incident, </span><span class="dl">'</span> <span class="o">+</span>
      <span class="dl">'</span><span class="s1">gather data from PagerDuty and Datadog, then provide: </span><span class="dl">'</span> <span class="o">+</span>
      <span class="dl">'</span><span class="s1">1) Incident timeline, 2) Affected services and metrics, </span><span class="dl">'</span> <span class="o">+</span>
      <span class="dl">'</span><span class="s1">3) Likely root cause, 4) Recommended remediation steps.</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="nx">session</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">assistant.message_delta</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">process</span><span class="p">.</span><span class="nx">stdout</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">deltaContent</span><span class="p">);</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">session</span><span class="p">.</span><span class="nf">sendAndWait</span><span class="p">({</span>
  <span class="na">prompt</span><span class="p">:</span>
    <span class="dl">'</span><span class="s1">We have a P1 incident on payments-api. Pull the PagerDuty details and check Datadog for the last hour.</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">stop</span><span class="p">();</span>
<span class="nx">process</span><span class="p">.</span><span class="nf">exit</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
</code></pre>

</div>


<p>You could deploy this as a Slack bot, a Teams webhook, or a CLI tool for your on-call team. The agent correlates PagerDuty incident metadata with Datadog metrics and logs, then produces a structured summary with remediation steps.</p>


<h3>
  
  
  Use Case 5: PR Age Visualisation and Repository Insights
</h3>

<p><strong>Source</strong>: <a href="https://github.com/github/awesome-copilot/blob/main/cookbook/copilot-sdk/nodejs/pr-visualization.md" rel="noopener noreferrer">Official Copilot SDK Cookbook: PR Visualization</a></p>

<p>This cookbook recipe demonstrates a powerful pattern: using the SDK with <strong>zero custom tools</strong>. Instead, it relies entirely on the Copilot CLI's built-in capabilities, the GitHub MCP server for fetching PR data, file tools for saving charts, and code execution for generating visualisations.</p>

<p>The core setup is refreshingly simple. No <code>defineTool</code> calls, just a session with a system message and a prompt:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">CopilotClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@github/copilot-sdk</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CopilotClient</span><span class="p">({</span> <span class="na">logLevel</span><span class="p">:</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span> <span class="p">});</span>

<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">createSession</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gpt-4.1</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">systemMessage</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">content</span><span class="p">:</span> <span class="s2">`You are analyzing pull requests for the GitHub repository: </span><span class="p">${</span><span class="nx">owner</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">repo</span><span class="p">}</span><span class="s2">.
Use the GitHub MCP Server tools to fetch PR data.
Use your file and code execution tools to generate charts.
Save any generated images to the current working directory.`</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">session</span><span class="p">.</span><span class="nf">sendAndWait</span><span class="p">({</span>
  <span class="na">prompt</span><span class="p">:</span> <span class="s2">`Fetch the open pull requests for </span><span class="p">${</span><span class="nx">owner</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">repo</span><span class="p">}</span><span class="s2"> from the last week.
Calculate the age of each PR in days.
Generate a bar chart showing the distribution of PR ages.
Save the chart as "pr-age-chart.png".
Summarise the PR health - average age, oldest PR, and how many might be stale.`</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>


<p>The agent uses the GitHub MCP server to list PRs, then generates a chart using Python/matplotlib. The interactive session lets you ask follow-up questions like "expand to the last month" or "group by author instead of age".</p>

<p>For DevOps leads, this pattern is gold. Build a scheduled job that runs this weekly and posts the chart to a Slack channel. No API integration code to maintain, just prompts.</p>


<h3>
  
  
  Use Case 6: Infrastructure as Code Validation Agent
</h3>

<p><strong>The problem</strong>: Your team maintains dozens of Terraform modules. Reviewing them for best practices, security compliance, and naming conventions is time-consuming and inconsistent.</p>

<p><strong>The solution</strong>: Build a CLI tool that reads a Terraform directory and asks the Copilot agent to validate it against your organisation's policies.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">CopilotClient</span><span class="p">,</span> <span class="nx">defineTool</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@github/copilot-sdk</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">fs</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">path</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">readTerraformFiles</span> <span class="o">=</span> <span class="nf">defineTool</span><span class="p">(</span><span class="dl">'</span><span class="s1">read_terraform_files</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Read all .tf files from a directory</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">parameters</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">object</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">properties</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">directory</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Path to the Terraform module directory</span><span class="dl">'</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">},</span>
    <span class="na">required</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">directory</span><span class="dl">'</span><span class="p">],</span>
  <span class="p">},</span>
  <span class="na">handler</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="na">args</span><span class="p">:</span> <span class="p">{</span> <span class="na">directory</span><span class="p">:</span> <span class="kr">string</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">files</span> <span class="o">=</span> <span class="nx">fs</span>
      <span class="p">.</span><span class="nf">readdirSync</span><span class="p">(</span><span class="nx">args</span><span class="p">.</span><span class="nx">directory</span><span class="p">)</span>
      <span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="nx">f</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">f</span><span class="p">.</span><span class="nf">endsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">.tf</span><span class="dl">'</span><span class="p">));</span>
    <span class="kd">const</span> <span class="na">contents</span><span class="p">:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="kr">string</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{};</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">file</span> <span class="k">of</span> <span class="nx">files</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">contents</span><span class="p">[</span><span class="nx">file</span><span class="p">]</span> <span class="o">=</span> <span class="nx">fs</span><span class="p">.</span><span class="nf">readFileSync</span><span class="p">(</span>
        <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">args</span><span class="p">.</span><span class="nx">directory</span><span class="p">,</span> <span class="nx">file</span><span class="p">),</span>
        <span class="dl">'</span><span class="s1">utf-8</span><span class="dl">'</span>
      <span class="p">);</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="nx">contents</span><span class="p">;</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CopilotClient</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">createSession</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gpt-4.1</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">streaming</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">tools</span><span class="p">:</span> <span class="p">[</span><span class="nx">readTerraformFiles</span><span class="p">],</span>
  <span class="na">systemMessage</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">content</span><span class="p">:</span> <span class="s2">`You are an infrastructure validation agent. When given a Terraform directory, read the files and check for:
1. All resources must include tags: Environment, Owner, CostCentre
2. No hardcoded secrets or credentials
3. Backend configuration must use remote state (azurerm or s3)
4. All variables must have descriptions and type constraints
5. Module sources must use version pinning
Report findings as a structured checklist with pass/fail for each rule.`</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="nx">session</span><span class="p">.</span><span class="nf">on</span><span class="p">(</span><span class="dl">'</span><span class="s1">assistant.message_delta</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">process</span><span class="p">.</span><span class="nx">stdout</span><span class="p">.</span><span class="nf">write</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">deltaContent</span><span class="p">);</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">session</span><span class="p">.</span><span class="nf">sendAndWait</span><span class="p">({</span>
  <span class="na">prompt</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Please validate the Terraform module at ./modules/networking</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>

<span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">stop</span><span class="p">();</span>
</code></pre>

</div>


<p>You could run this as a pre-commit hook, a CI step, or a standalone CLI that your platform team uses during module reviews. Combine it with the Ralph Loop pattern from Use Case 3 to validate and fix an entire library of modules autonomously.</p>


<h2>
  
  
  Authentication Options
</h2>

<p>The SDK supports several authentication methods, giving you flexibility for different environments:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>GitHub signed-in user</strong></td>
<td>Local development; uses stored OAuth credentials from <code>copilot</code> CLI login</td>
</tr>
<tr>
<td><strong>OAuth GitHub App</strong></td>
<td>Web applications; pass user tokens from your GitHub OAuth app</td>
</tr>
<tr>
<td><strong>Environment variables</strong></td>
<td>CI/CD pipelines; set <code>COPILOT_GITHUB_TOKEN</code>, <code>GH_TOKEN</code>, or <code>GITHUB_TOKEN</code>
</td>
</tr>
<tr>
<td><strong>BYOK (Bring Your Own Key)</strong></td>
<td>Air-gapped or custom environments; use your own API keys from OpenAI, Azure AI Foundry, or Anthropic</td>
</tr>
</tbody>
</table></div>

<p>For CI/CD integration, environment variables are the most practical. For internal tools, the GitHub OAuth flow gives you per-user billing and audit trails.</p>

<blockquote>
<p><strong>Note</strong>: BYOK uses key-based authentication only. Microsoft Entra ID (Azure AD), managed identities, and third-party identity providers are not supported at this time.</p>
</blockquote>


<h2>
  
  
  Tips for DevOps Teams
</h2>

<p><strong>Start with a well-defined problem.</strong> The SDK is not a magic box. It works best when you give the agent a clear task, the right tools, and a focused system message. "Analyse these logs and find errors" will outperform "do DevOps stuff."</p>

<p><strong>Use system messages to encode your standards.</strong> The <code>systemMessage</code> field is your equivalent of a runbook. Tell the agent about your naming conventions, required tags, preferred tools, and escalation procedures.</p>

<p><strong>Combine custom tools with MCP servers.</strong> Use MCP for standard integrations (GitHub API, cloud providers) and custom tools for your organisation-specific logic (internal APIs, proprietary systems).</p>

<p><strong>Keep tools focused.</strong> Each tool should do one thing well. An agent with ten small, focused tools will perform better than one with three large, multi-purpose tools.</p>

<p><strong>Review and validate.</strong> The SDK is in Technical Preview. Always review agent outputs before acting on them in production, especially for infrastructure changes or incident response recommendations.</p>

<p><strong>Be mindful of billing.</strong> Each prompt counts against your Copilot premium request quota. For high-volume automation, consider batching requests and caching responses where appropriate.</p>


<h2>
  
  
  Conclusion
</h2>

<p>The <strong>GitHub Copilot SDK</strong> brings agentic AI out of the IDE and into your operational tooling. For DevOps engineers, this means you can build custom agents that understand your infrastructure, your workflows, and your conventions, and embed them wherever they are most useful: CLI tools, chatbots, CI pipelines, or internal platforms.</p>

<p>The community is already proving the concept. Projects like <a href="https://github.com/htekdev/github-sre-agent" rel="noopener noreferrer">github-sre-agent</a> (autonomous CI/CD failure triage), <a href="https://github.com/glaucia86/repocheckai" rel="noopener noreferrer">RepoCheckAI</a> (repository health analysis), and Microsoft's <a href="https://github.com/microsoft/copilot-sdk-samples" rel="noopener noreferrer">copilot-sdk-samples</a> (PagerDuty and Datadog integrations) show that this is not theoretical. The official <a href="https://github.com/github/awesome-copilot/blob/main/cookbook/copilot-sdk" rel="noopener noreferrer">cookbook</a> adds patterns like Ralph Loops for autonomous task iteration and PR visualisation using zero custom tools.</p>

<p>The SDK is currently in <strong>Technical Preview</strong> with support for <strong>Python, TypeScript, Go, and .NET</strong>, an MIT licence, and a growing ecosystem of MCP integrations. Whether you want to automate SRE workflows, audit repositories, triage incidents, or visualise PR health, the building blocks are ready.</p>

<p>Check out the <a href="https://github.com/github/copilot-sdk" rel="noopener noreferrer">GitHub Copilot SDK repository</a> and the <a href="https://github.com/github/copilot-sdk/blob/main/docs/getting-started.md" rel="noopener noreferrer">getting started guide</a> to begin experimenting.</p>
<h3>
  
  
  <em>Author</em>
</h3>


<div class="ltag__user ltag__user__id__620034">
    <a href="/pwd9000" class="ltag__user__link profile-image-link">
      <div class="ltag__user__pic">
        <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F620034%2F93be2c72-3a13-478e-8af1-a4bedc1b2331.jpeg" alt="pwd9000 image">
      </div>
    </a>
  <div class="ltag__user__content">
    <h2>
<a class="ltag__user__link" href="/pwd9000">Marcel.L</a>Follow
</h2>
    <div class="ltag__user__summary">
      <a class="ltag__user__link" href="/pwd9000">Microsoft MVP in DevTech - DevOps | DevOps Architect | Technical speaker focused on Microsoft technologies, Agentic AI, IaC &amp; automation in Azure. Find me on GitHub: https://github.com/Pwd9000-ML</a>
    </div>
  </div>
</div>



<p>Like, share, follow me on: 🐙 <a href="https://github.com/Pwd9000-ML" rel="noopener noreferrer">GitHub</a> | 🐧 <a href="https://x.com/pwd9000" rel="noopener noreferrer">X</a> | 👾 <a href="https://www.linkedin.com/in/marcel-pwd9000/" rel="noopener noreferrer">LinkedIn</a></p>

<p>Date: 17-02-2026</p>

