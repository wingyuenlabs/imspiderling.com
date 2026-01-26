---
Title: # MCP vs CAP: Why Your AI Agents Need Both Protocols
Description: 
Author: yaron torgeman
Date: 2026-01-26T20:58:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>The AI agent ecosystem is exploding with protocols. Anthropic released MCP (Model Context Protocol). Google announced A2A (Agent-to-Agent). Every week there's a new "standard" for agent communication.</p>

<p>But here's the thing most people miss: <strong>these protocols solve different problems at different layers.</strong> Using MCP for distributed agent orchestration is like using HTTP for job scheduling—wrong tool, wrong layer.</p>

<p>Let me break down the actual difference and why you probably need both.</p>




<h2>
  
  
  What MCP Actually Does
</h2>

<p>MCP (Model Context Protocol) is a <strong>tool-calling protocol for a single model</strong>. It standardizes how one LLM discovers and invokes external tools—databases, APIs, file systems, etc.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────┐
│            Your LLM                 │
│                                     │
│  "I need to query the database"     │
│              │                      │
│              ▼                      │
│     ┌─────────────┐                 │
│     │  MCP Client │                 │
│     └──────┬──────┘                 │
└────────────┼────────────────────────┘
             │
             ▼
     ┌───────────────┐
     │  MCP Server   │
     │  (tool host)  │
     └───────────────┘
             │
             ▼
        [Database]
</code></pre>

</div>



<p>MCP is great at this. It solves tool discovery, schema negotiation, and invocation for a <strong>single model context</strong>.</p>

<p><strong>What MCP doesn't cover:</strong></p>

<ul>
<li>How do you schedule work across multiple agents?</li>
<li>How do you track job state across a cluster?</li>
<li>How do you enforce safety policies before execution?</li>
<li>How do you handle agent liveness and capacity?</li>
<li>How do you fan out workflows with parent/child relationships?</li>
</ul>

<p>MCP was never designed for this. It's a tool protocol, not an orchestration protocol.</p>




<h2>
  
  
  Enter CAP: The Missing Layer
</h2>

<p>CAP (Cordum Agent Protocol) is a <strong>cluster-native job protocol for AI agents</strong>. It standardizes the control plane that MCP doesn't touch:</p>

<ul>
<li>
<strong>Job lifecycle</strong>: submit → schedule → dispatch → run → complete</li>
<li>
<strong>Distributed routing</strong>: pool-based dispatch with competing consumers</li>
<li>
<strong>Safety hooks</strong>: allow/deny/throttle decisions before any job runs</li>
<li>
<strong>Heartbeats</strong>: worker liveness, capacity, and pool membership</li>
<li>
<strong>Workflows</strong>: parent/child jobs with aggregation</li>
<li>
<strong>Pointer architecture</strong>: keeps payloads off the bus for security and performance
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────────┐
│                     CAP Control Plane                       │
│                                                             │
│  Client ──▶ Gateway ──▶ Scheduler ──▶ Safety ──▶ Workers   │
│                              │                      │       │
│                              ▼                      ▼       │
│                         [Job State]           [Results]     │
└─────────────────────────────────────────────────────────────┘
                                                      │
                                                      ▼
                                              ┌──────────────┐
                                              │ MCP (tools)  │
                                              └──────────────┘
</code></pre>

</div>



<p><strong>CAP handles:</strong></p>

<ul>
<li>
<code>BusPacket</code> envelopes for all messages</li>
<li>
<code>JobRequest</code> / <code>JobResult</code> with full state machine</li>
<li>
<code>context_ptr</code> / <code>result_ptr</code> to keep blobs off the wire</li>
<li>Heartbeats for worker pools</li>
<li>Safety Kernel integration (policy checks before dispatch)</li>
<li>Workflow orchestration with <code>workflow_id</code>, <code>parent_job_id</code>, <code>step_index</code>
</li>
</ul>




<h2>
  
  
  The Key Insight: Different Layers
</h2>

<p>Think of it like the network stack:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Protocol</th>
<th>What It Does</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Tool execution</strong></td>
<td>MCP</td>
<td>Model ↔ Tool communication</td>
</tr>
<tr>
<td><strong>Agent orchestration</strong></td>
<td>CAP</td>
<td>Job scheduling, routing, safety, state</td>
</tr>
<tr>
<td><strong>Transport</strong></td>
<td>NATS/Kafka</td>
<td>Message delivery</td>
</tr>
</tbody>
</table></div>

<p><strong>MCP is layer 7. CAP is layer 5-6.</strong></p>

<p>You wouldn't use HTTP to schedule Kubernetes jobs. Similarly, you shouldn't use MCP to orchestrate distributed agent workloads.</p>




<h2>
  
  
  How They Work Together
</h2>

<p>Here's the beautiful part: <strong>MCP and CAP complement each other perfectly.</strong></p>

<p>A CAP worker receives a job, executes it (potentially using MCP to call tools), and returns a result. MCP handles the tool-calling inside the worker. CAP handles everything outside.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────────────┐
│                         CAP Cluster                             │
│                                                                 │
│   ┌──────────┐    ┌───────────┐    ┌─────────────────────────┐ │
│   │  Client  │───▶│ Scheduler │───▶│      Worker Pool        │ │
│   └──────────┘    └───────────┘    │  ┌───────────────────┐  │ │
│                         │          │  │   CAP Worker      │  │ │
│                         ▼          │  │        │          │  │ │
│                   [Safety Kernel]  │  │        ▼          │  │ │
│                                    │  │   ┌─────────┐     │  │ │
│                                    │  │   │   MCP   │     │  │ │
│                                    │  │   │ Client  │     │  │ │
│                                    │  │   └────┬────┘     │  │ │
│                                    │  └────────┼──────────┘  │ │
│                                    └───────────┼─────────────┘ │
└────────────────────────────────────────────────┼───────────────┘
                                                 ▼
                                          [MCP Servers]
                                          (tools, DBs, APIs)
</code></pre>

</div>



<p><strong>Example flow:</strong></p>

<ol>
<li>Client submits job via CAP (<code>JobRequest</code> to <code>sys.job.submit</code>)</li>
<li>Scheduler checks Safety Kernel → approved</li>
<li>Job dispatched to worker pool via CAP</li>
<li>Worker uses MCP to call tools (query DB, fetch API, etc.)</li>
<li>Worker returns result via CAP (<code>JobResult</code> to <code>sys.job.result</code>)</li>
<li>Scheduler updates state, notifies client</li>
</ol>

<p>MCP never touches the bus. CAP never touches the tools. Clean separation.</p>




<h2>
  
  
  Why This Matters for Production
</h2>

<p>If you're building a toy demo, you don't need CAP. One model, a few tools, MCP is plenty.</p>

<p>But if you're building <strong>production multi-agent systems</strong>, you need:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Requirement</th>
<th>MCP</th>
<th>CAP</th>
</tr>
</thead>
<tbody>
<tr>
<td>Tool discovery &amp; invocation</td>
<td>✅</td>
<td>❌</td>
</tr>
<tr>
<td>Job scheduling</td>
<td>❌</td>
<td>✅</td>
</tr>
<tr>
<td>Distributed worker pools</td>
<td>❌</td>
<td>✅</td>
</tr>
<tr>
<td>Safety policies (allow/deny/throttle)</td>
<td>❌</td>
<td>✅</td>
</tr>
<tr>
<td>Job state machine</td>
<td>❌</td>
<td>✅</td>
</tr>
<tr>
<td>Worker heartbeats &amp; capacity</td>
<td>❌</td>
<td>✅</td>
</tr>
<tr>
<td>Workflow orchestration</td>
<td>❌</td>
<td>✅</td>
</tr>
<tr>
<td>Payload security (pointer refs)</td>
<td>❌</td>
<td>✅</td>
</tr>
</tbody>
</table></div>

<p><strong>CAP gives you the control plane. MCP gives you the tool plane.</strong></p>




<h2>
  
  
  Getting Started with CAP
</h2>

<p>CAP is open source (Apache-2.0) with SDKs for Go, Python, Node/TS, and C++.</p>

<p><strong>Minimal Go worker (20 lines):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">nc</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">nats</span><span class="o">.</span><span class="n">Connect</span><span class="p">(</span><span class="s">"nats://127.0.0.1:4222"</span><span class="p">)</span>

<span class="n">nc</span><span class="o">.</span><span class="n">QueueSubscribe</span><span class="p">(</span><span class="s">"job.echo"</span><span class="p">,</span> <span class="s">"job.echo"</span><span class="p">,</span> <span class="k">func</span><span class="p">(</span><span class="n">msg</span> <span class="o">*</span><span class="n">nats</span><span class="o">.</span><span class="n">Msg</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">var</span> <span class="n">pkt</span> <span class="n">agentv1</span><span class="o">.</span><span class="n">BusPacket</span>
    <span class="n">proto</span><span class="o">.</span><span class="n">Unmarshal</span><span class="p">(</span><span class="n">msg</span><span class="o">.</span><span class="n">Data</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">pkt</span><span class="p">)</span>

    <span class="n">req</span> <span class="o">:=</span> <span class="n">pkt</span><span class="o">.</span><span class="n">GetJobRequest</span><span class="p">()</span>
    <span class="n">res</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">agentv1</span><span class="o">.</span><span class="n">JobResult</span><span class="p">{</span>
        <span class="n">JobId</span><span class="o">:</span>  <span class="n">req</span><span class="o">.</span><span class="n">GetJobId</span><span class="p">(),</span>
        <span class="n">Status</span><span class="o">:</span> <span class="n">agentv1</span><span class="o">.</span><span class="n">JobStatus_JOB_STATUS_SUCCEEDED</span><span class="p">,</span>
    <span class="p">}</span>

    <span class="n">out</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">proto</span><span class="o">.</span><span class="n">Marshal</span><span class="p">(</span><span class="o">&amp;</span><span class="n">agentv1</span><span class="o">.</span><span class="n">BusPacket</span><span class="p">{</span>
        <span class="n">Payload</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">agentv1</span><span class="o">.</span><span class="n">BusPacket_JobResult</span><span class="p">{</span><span class="n">JobResult</span><span class="o">:</span> <span class="n">res</span><span class="p">},</span>
    <span class="p">})</span>
    <span class="n">nc</span><span class="o">.</span><span class="n">Publish</span><span class="p">(</span><span class="s">"sys.job.result"</span><span class="p">,</span> <span class="n">out</span><span class="p">)</span>
<span class="p">})</span>
</code></pre>

</div>



<p><strong>Links:</strong></p>

<ul>
<li>GitHub: <a href="https://github.com/cordum-io/cap" rel="noopener noreferrer">github.com/cordum-io/cap</a>
</li>
<li>Spec: <a href="https://github.com/cordum-io/cap/tree/main/spec" rel="noopener noreferrer">github.com/cordum-io/cap/tree/main/spec</a>
</li>
<li>Reference implementation: <a href="https://cordum.io" rel="noopener noreferrer">Cordum</a>
</li>
</ul>




<h2>
  
  
  TL;DR
</h2>

<ul>
<li>
<strong>MCP</strong> = tool protocol for single-model contexts</li>
<li>
<strong>CAP</strong> = job protocol for distributed agent clusters</li>
<li><strong>They solve different problems at different layers</strong></li>
<li>
<strong>Use both</strong>: CAP for orchestration, MCP inside workers for tools</li>
<li>Stop using MCP for things it wasn't designed for</li>
</ul>

<p>The multi-agent future needs both protocols. Now you know which one to reach for.</p>




<p><em>CAP is developed by <a href="https://cordum.io" rel="noopener noreferrer">Cordum</a>, the AI Agent Governance Platform. Star the repo if this was useful: <a href="https://github.com/cordum-io/cap" rel="noopener noreferrer">github.com/cordum-io/cap</a></em></p>




<p><strong>Tags:</strong> #ai #agents #mcp #distributed-systems #orchestration #protocols</p>

