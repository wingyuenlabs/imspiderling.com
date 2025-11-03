---
Title: Building PRRover: A FastAPI GitHub PR Reviewer with Telex A2A Integration
Description: 
Author: Samuel Oshin
Date: 2025-11-03T21:44:32.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhref9ie945btcgcq2aj8.png" alt=" " width="800" height="800">
</h2>

<p>I recently built <strong>PRRover</strong>, an AI-powered GitHub Pull Request reviewer that integrates seamlessly with Telex, an AI agent platform like Zapier. It uses the cutting-edge <strong>A2A (Agent-to-Agent)</strong> protocol to perform automated code reviews—analyzing security vulnerabilities, performance issues, and adherence to best practices—and delivers rich, actionable reports directly in your chat interface. This project is a perfect showcase of marrying Python's power, FastAPI's speed, and the A2A standard.</p>

<p>You can chat with the live agent and try it yourself on Telex: <a href="https://telex.im" rel="noopener noreferrer">PRRover: The GitHub Code Review Bot</a>. This agent monitors and analyzes GitHub Pull Requests for security, performance, and best practice adherence, delivering comprehensive, actionable reports to the user.</p>

<h2>
  
  
  In this post, I'll walk you through:
</h2>

<ul>
<li>What the A2A protocol is and how it works</li>
<li>Building an A2A-compliant agent with FastAPI</li>
<li>Implementing blocking vs non-blocking (webhook) flows</li>
<li>Using LLMs for deep code analysis</li>
<li>Testing and troubleshooting the integration</li>
</ul>

<p>Whether you're building chat-integrated agents or exploring webhook-driven architectures, the patterns here will help you ship a production-ready A2A integration.</p>




<h2>
  
  
  🎯 What We Built
</h2>

<p><strong>PRRover</strong> is a FastAPI service that:</p>

<ul>
<li>✅ Accepts A2A JSON-RPC calls from Telex at <code>/a2a/tasks</code>
</li>
<li>✅ Supports blocking and non-blocking modes

<ul>
<li>
<strong>Blocking:</strong> Returns completed analysis immediately (for tasks under 10 seconds)</li>
<li>
<strong>Non-blocking:</strong> Returns "accepted" instantly, then pushes results via webhook (for complex, 30-40 second analyses)</li>
</ul>


</li>

<li>✅ Uses hybrid analysis pipeline:

<ul>
<li>Fast rule-based security/performance checks</li>
<li>Deep LLM-powered analysis (Google Gemini 2.0)</li>
</ul>


</li>

<li>✅ Delivers results as rich text artifacts with CWE references, severity levels, and actionable recommendations</li>

</ul>

<p><strong>Live Demo:</strong> <a href="https://code-reviewer-agent-a2a.onrender.com" rel="noopener noreferrer">https://code-reviewer-agent-a2a.onrender.com</a></p>

<p><strong>Repository:</strong> <a href="https://github.com/SamuelOshin/code_reviewer_agent_a2a" rel="noopener noreferrer">https://github.com/SamuelOshin/code_reviewer_agent_a2a</a></p>




<h2>
  
  
  🐍 The Core Integration: Python, FastAPI, and A2A
</h2>

<p>The entire PRRover service is built on <strong>Python</strong>, leveraging the high-performance, asynchronous web framework <strong>FastAPI</strong>. FastAPI's native support for asynchronous operations (<code>async</code>/<code>await</code>) was crucial for implementing the non-blocking A2A flow, ensuring a snappy user experience even for lengthy tasks.</p>

<h3>
  
  
  Handling the A2A Protocol in FastAPI
</h3>

<p>The <strong>Agent-to-Agent (A2A)</strong> protocol mandates communication via JSON-RPC 2.0 over a single endpoint, <code>/a2a/tasks</code>. My FastAPI implementation handles this integration by:</p>

<ol>
<li><p><strong>Request Parsing:</strong> The main <code>@router.post("/a2a/tasks")</code> endpoint receives the incoming raw JSON payload. Since A2A is JSON-RPC, the function extracts crucial fields like <code>method</code> (e.g., <code>message/send</code>), <code>id</code>, and <code>params</code>.</p></li>
<li><p><strong>Asynchronous Dispatch (The Key to UX):</strong> When a lengthy task is detected (by checking the <code>configuration.blocking</code> flag and finding it set to <code>false</code>), FastAPI immediately returns an HTTP 200 response with the <code>accepted</code> status. Crucially, we use Python's <code>asyncio.create_task</code> to spawn a non-blocking background process. This frees up the HTTP worker, preventing timeouts while the actual, compute-heavy work proceeds in parallel.</p></li>
<li><p><strong>Webhook Fulfillment:</strong> Once the Python analysis pipeline is complete, the background task uses an asynchronous HTTP client to push the final <code>completed</code> task result back to the unique Telex webhook URL provided in the initial A2A request. This pattern is essential for any long-running agent built with FastAPI.</p></li>
</ol>




<h2>
  
  
  📡 Understanding the A2A Protocol
</h2>

<p>A2A uses <strong>JSON-RPC 2.0</strong> for agent-to-agent communication. Key concepts:</p>

<h3>
  
  
  Incoming Request from Telex
</h3>

<p>When a user sends a message to your agent through Telex:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"jsonrpc"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"6d484504a27146998768f486aa4a8b41"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"method"</span><span class="p">:</span><span class="w"> </span><span class="s2">"message/send"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"params"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"message"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"message"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"role"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"parts"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">{</span><span class="w">
          </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"text"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"text"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Review https://github.com/owner/repo/pull/123"</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="nl">"messageId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"c4bdc8b7c96f475ba92919b80ead7729"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"configuration"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"blocking"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
      </span><span class="nl">"pushNotificationConfig"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"https://ping.telex.im/v1/a2a/webhooks/..."</span><span class="p">,</span><span class="w">
        </span><span class="nl">"token"</span><span class="p">:</span><span class="w"> </span><span class="s2">"eyJhbGc..."</span><span class="p">,</span><span class="w">
        </span><span class="nl">"authentication"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"schemes"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Bearer"</span><span class="p">]</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Your Agent's Response
</h3>

<p><strong>For blocking requests,</strong> return a completed task immediately:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"jsonrpc"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"6d484504a27146998768f486aa4a8b41"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"result"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-uuid"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"contextId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"context-uuid"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"state"</span><span class="p">:</span><span class="w"> </span><span class="s2">"completed"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"timestamp"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025-11-03T17:35:23.727191Z"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"message"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"messageId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"msg-uuid"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"role"</span><span class="p">:</span><span class="w"> </span><span class="s2">"agent"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"parts"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="w">
          </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"text"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"text"</span><span class="p">:</span><span class="w"> </span><span class="s2">"✅ Analysis complete! Found 6 security issues..."</span><span class="w">
        </span><span class="p">}],</span><span class="w">
        </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"message"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"taskId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-uuid"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"timestamp"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025-11-03T17:35:23.727162Z"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"progress"</span><span class="p">:</span><span class="w"> </span><span class="mf">1.0</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"artifacts"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"artifactId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"artifact-uuid"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PR #123 Analysis"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"parts"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="w">
          </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"text"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"text"</span><span class="p">:</span><span class="w"> </span><span class="s2">"# Full Analysis Report</span><span class="se">\n\n</span><span class="s2">..."</span><span class="w">
        </span><span class="p">}]</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">],</span><span class="w">
    </span><span class="nl">"history"</span><span class="p">:</span><span class="w"> </span><span class="p">[],</span><span class="w">
    </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>For non-blocking requests,</strong> immediately return "accepted":<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"jsonrpc"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"6d484504a27146998768f486aa4a8b41"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"result"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-uuid"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"contextId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"context-uuid"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"state"</span><span class="p">:</span><span class="w"> </span><span class="s2">"accepted"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"message"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"parts"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="w">
          </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"text"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"text"</span><span class="p">:</span><span class="w"> </span><span class="s2">"🔄 Analyzing PR now! I'll send results shortly..."</span><span class="w">
        </span><span class="p">}],</span><span class="w">
        </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"message"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"taskId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-uuid"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"progress"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.1</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"artifacts"</span><span class="p">:</span><span class="w"> </span><span class="p">[],</span><span class="w">
    </span><span class="nl">"history"</span><span class="p">:</span><span class="w"> </span><span class="p">[],</span><span class="w">
    </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Then, after analysis completes, <strong>POST to the webhook:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"jsonrpc"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2.0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"webhook-rpc-id"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"result"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-uuid"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"contextId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"context-uuid"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"state"</span><span class="p">:</span><span class="w"> </span><span class="s2">"completed"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"artifacts"</span><span class="p">:</span><span class="w"> </span><span class="p">[],</span><span class="w">
    </span><span class="nl">"history"</span><span class="p">:</span><span class="w"> </span><span class="p">[],</span><span class="w">
    </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<blockquote>
<p><strong>Critical:</strong> Webhooks expect a JSON-RPC <strong>response</strong> (with <code>result</code>), not a request (with <code>method</code>/<code>params</code>). This tripped me up initially!</p>
</blockquote>




<h2>
  
  
  🏗️ Architecture Overview
</h2>

<p>The following diagram illustrates the non-blocking flow where the FastAPI application delegates the long-running analysis to a background task and fulfills the result via the Telex webhook.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────────┐
│                       Telex Platform                        │
│                     (User sends PR URL)                     │
└────────────────────────┬────────────────────────────────────┘
                         │ POST /a2a/tasks (JSON-RPC)
                         │ blocking: false + webhook config
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                 FastAPI Application (Python)                │
│ ┌──────────────────────────────────────────────────────┐  │
│ │   app/routes/a2a_tasks.py                            │  │
│ │   • Receives JSON-RPC requests                       │  │
│ │   • Routes to message handler                        │  │
│ └─────────────────────┬────────────────────────────────┘  │
│                       │                                   │
│ ┌─────────────────────▼────────────────────────────────┐  │
│ │   app/services/message_handler.py                    │  │
│ │   • Parse message &amp; configuration                    │  │
│ │   • Blocking: analyze synchronously                  │  │
│ │   • Non-blocking: return "accepted" + spawn bg task  │  │
│ └─────────────────────┬────────────────────────────────┘  │
│                       │                                   │
│ ┌─────────────────────▼────────────────────────────────┐  │
│ │   app/services/code_analyzer.py                      │  │
│ │   1. Fetch PR from GitHub (via MCP)                  │  │
│ │   2. Rule-based checks (security, performance)       │  │
│ │   3. LLM analysis (Gemini 2.0)                       │  │
│ │     • analyze_security()                             │  │
│ │     • analyze_performance()                          │  │
│ │     • analyze_best_practices()                       │  │
│ │   4. Generate summary                                │  │
│ └─────────────────────┬────────────────────────────────┘  │
│                       │                                   │
│ ┌─────────────────────▼────────────────────────────────┐  │
│ │   app/utils/formatters.py                            │  │
│ │   • Format for Telex UI (emoji, markdown)            │  │
│ │   • Create artifact text                             │  │
│ └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │ For non-blocking:
                         │ POST webhook with completed result
                         ▼
┌─────────────────────────────────────────────────────────────┐
│            Telex Webhook (ping.telex.im)                    │
│          Receives completed task, displays to user          │
└─────────────────────────────────────────────────────────────┘
</code></pre>

</div>






<h2>
  
  
  🚀 Implementation: FastAPI + Non-Blocking Flow
</h2>

<h3>
  
  
  1. FastAPI Route
</h3>

<p>The core of the FastAPI integration is setting up a single asynchronous route to receive all A2A traffic and handle the JSON-RPC parsing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># app/routes/a2a_tasks.py
</span><span class="kn">from</span> <span class="n">fastapi</span> <span class="kn">import</span> <span class="n">APIRouter</span><span class="p">,</span> <span class="n">Request</span>
<span class="kn">from</span> <span class="n">fastapi.responses</span> <span class="kn">import</span> <span class="n">JSONResponse</span>

<span class="n">router</span> <span class="o">=</span> <span class="nc">APIRouter</span><span class="p">()</span>

<span class="nd">@router.post</span><span class="p">(</span><span class="sh">"</span><span class="s">/a2a/tasks</span><span class="sh">"</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">handle_a2a_task</span><span class="p">(</span><span class="n">request</span><span class="p">:</span> <span class="n">Request</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">A2A endpoint for Telex integration</span><span class="sh">"""</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">payload</span> <span class="o">=</span> <span class="k">await</span> <span class="n">request</span><span class="p">.</span><span class="nf">json</span><span class="p">()</span>

        <span class="c1"># Extract JSON-RPC fields
</span>        <span class="n">rpc_id</span> <span class="o">=</span> <span class="n">payload</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">method</span> <span class="o">=</span> <span class="n">payload</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">method</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">params</span> <span class="o">=</span> <span class="n">payload</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">params</span><span class="sh">"</span><span class="p">,</span> <span class="p">{})</span>

        <span class="k">if</span> <span class="n">method</span> <span class="o">==</span> <span class="sh">"</span><span class="s">message/send</span><span class="sh">"</span><span class="p">:</span>
            <span class="c1"># Delegate to message handler
</span>            <span class="n">result</span> <span class="o">=</span> <span class="k">await</span> <span class="n">message_handler</span><span class="p">.</span><span class="nf">handle_message_send</span><span class="p">(</span>
                <span class="n">message</span><span class="o">=</span><span class="n">params</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">),</span>
                <span class="n">configuration</span><span class="o">=</span><span class="n">params</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">configuration</span><span class="sh">"</span><span class="p">)</span>
            <span class="p">)</span>

            <span class="c1"># Return JSON-RPC response
</span>            <span class="k">return</span> <span class="nc">JSONResponse</span><span class="p">({</span>
                <span class="sh">"</span><span class="s">jsonrpc</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">2.0</span><span class="sh">"</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">rpc_id</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">result</span><span class="sh">"</span><span class="p">:</span> <span class="n">result</span>
            <span class="p">})</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Error handling request: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="nc">JSONResponse</span><span class="p">({</span>
            <span class="sh">"</span><span class="s">jsonrpc</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">2.0</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">rpc_id</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="sh">"</span><span class="s">code</span><span class="sh">"</span><span class="p">:</span> <span class="o">-</span><span class="mi">32603</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">)</span>
            <span class="p">}</span>
        <span class="p">},</span> <span class="n">status_code</span><span class="o">=</span><span class="mi">500</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  2. Message Handler (Non-Blocking Pattern)
</h3>

<p>Here's the key Python pattern for handling long-running tasks asynchronously using <code>asyncio</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># app/services/message_handler.py
</span><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">datetime</span> <span class="kn">import</span> <span class="n">datetime</span><span class="p">,</span> <span class="n">timezone</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">handle_message_send</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span> 
    <span class="n">message</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">],</span> 
    <span class="n">configuration</span><span class="p">:</span> <span class="n">Optional</span><span class="p">[</span><span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]]</span> <span class="o">=</span> <span class="bp">None</span>
<span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="sh">"""</span><span class="s">Handle message/send JSON-RPC method</span><span class="sh">"""</span>

    <span class="c1"># Parse configuration
</span>    <span class="n">is_blocking</span> <span class="o">=</span> <span class="n">configuration</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">blocking</span><span class="sh">"</span><span class="p">,</span> <span class="bp">True</span><span class="p">)</span> <span class="k">if</span> <span class="n">configuration</span> <span class="k">else</span> <span class="bp">True</span>
    <span class="n">push_config</span> <span class="o">=</span> <span class="n">configuration</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">pushNotificationConfig</span><span class="sh">"</span><span class="p">)</span> <span class="k">if</span> <span class="n">configuration</span> <span class="k">else</span> <span class="bp">None</span>

    <span class="c1"># Extract PR URL from message
</span>    <span class="n">pr_url</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_extract_pr_url</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>

    <span class="c1"># Non-blocking mode: return accepted, analyze in background
</span>    <span class="k">if</span> <span class="ow">not</span> <span class="n">is_blocking</span> <span class="ow">and</span> <span class="n">push_config</span><span class="p">:</span>
        <span class="n">task_id</span> <span class="o">=</span> <span class="nf">str</span><span class="p">(</span><span class="n">uuid</span><span class="p">.</span><span class="nf">uuid4</span><span class="p">())</span>
        <span class="n">context_id</span> <span class="o">=</span> <span class="n">message</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">contextId</span><span class="sh">"</span><span class="p">,</span> <span class="nf">str</span><span class="p">(</span><span class="n">uuid</span><span class="p">.</span><span class="nf">uuid4</span><span class="p">()))</span>

        <span class="c1"># Start background task using Python's asyncio
</span>        <span class="n">asyncio</span><span class="p">.</span><span class="nf">create_task</span><span class="p">(</span>
            <span class="n">self</span><span class="p">.</span><span class="nf">_process_and_push_safe</span><span class="p">(</span>
                <span class="n">pr_url</span><span class="o">=</span><span class="n">pr_url</span><span class="p">,</span>
                <span class="n">task_id</span><span class="o">=</span><span class="n">task_id</span><span class="p">,</span>
                <span class="n">message_id</span><span class="o">=</span><span class="n">message</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">messageId</span><span class="sh">"</span><span class="p">),</span>
                <span class="n">message</span><span class="o">=</span><span class="n">message</span><span class="p">,</span>
                <span class="n">push_config</span><span class="o">=</span><span class="n">push_config</span>
            <span class="p">)</span>
        <span class="p">)</span>

        <span class="c1"># Return "accepted" immediately
</span>        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="n">task_id</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">contextId</span><span class="sh">"</span><span class="p">:</span> <span class="n">context_id</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="sh">"</span><span class="s">state</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">accepted</span><span class="sh">"</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">timestamp</span><span class="sh">"</span><span class="p">:</span> <span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">(</span><span class="n">timezone</span><span class="p">.</span><span class="n">utc</span><span class="p">).</span><span class="nf">isoformat</span><span class="p">(),</span>
                <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
                    <span class="sh">"</span><span class="s">messageId</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">uuid</span><span class="p">.</span><span class="nf">uuid4</span><span class="p">()),</span>
                    <span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">agent</span><span class="sh">"</span><span class="p">,</span>
                    <span class="sh">"</span><span class="s">parts</span><span class="sh">"</span><span class="p">:</span> <span class="p">[{</span>
                        <span class="sh">"</span><span class="s">kind</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">,</span>
                        <span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">🔄 Analyzing PR! Results coming shortly...</span><span class="se">\n\n</span><span class="s">📎 </span><span class="si">{</span><span class="n">pr_url</span><span class="si">}</span><span class="sh">"</span>
                    <span class="p">}],</span>
                    <span class="sh">"</span><span class="s">kind</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">,</span>
                    <span class="sh">"</span><span class="s">taskId</span><span class="sh">"</span><span class="p">:</span> <span class="n">task_id</span>
                <span class="p">},</span>
                <span class="sh">"</span><span class="s">progress</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.1</span>
            <span class="p">},</span>
            <span class="sh">"</span><span class="s">artifacts</span><span class="sh">"</span><span class="p">:</span> <span class="p">[],</span>
            <span class="sh">"</span><span class="s">history</span><span class="sh">"</span><span class="p">:</span> <span class="p">[],</span>
            <span class="sh">"</span><span class="s">kind</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">task</span><span class="sh">"</span>
        <span class="p">}</span>

    <span class="c1"># Blocking mode: analyze and return completed task
</span>    <span class="k">return</span> <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="nf">_analyze_and_return</span><span class="p">(</span><span class="n">pr_url</span><span class="p">,</span> <span class="n">task_id</span><span class="p">,</span> <span class="n">message_id</span><span class="p">,</span> <span class="n">message</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  3. Background Task with Webhook Push
</h3>

<p>The final stage of the non-blocking A2A flow is pushing the result back to Telex via the provided webhook URL, using an asynchronous client like <code>httpx</code> within the Python background task:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">async</span> <span class="k">def</span> <span class="nf">_process_and_push_safe</span><span class="p">(</span>
    <span class="n">self</span><span class="p">,</span>
    <span class="n">pr_url</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">task_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">message_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">message</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">],</span>
    <span class="n">push_config</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]</span>
<span class="p">):</span>
    <span class="sh">"""</span><span class="s">Analyze PR and push result to Telex webhook</span><span class="sh">"""</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="c1"># Run analysis (30-40 seconds)
</span>        <span class="n">result</span> <span class="o">=</span> <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="nf">_analyze_and_return</span><span class="p">(</span><span class="n">pr_url</span><span class="p">,</span> <span class="n">task_id</span><span class="p">,</span> <span class="n">message_id</span><span class="p">,</span> <span class="n">message</span><span class="p">)</span>

        <span class="c1"># Build webhook payload (JSON-RPC response format!)
</span>        <span class="n">webhook_payload</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">jsonrpc</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">2.0</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">uuid</span><span class="p">.</span><span class="nf">uuid4</span><span class="p">()),</span>
            <span class="sh">"</span><span class="s">result</span><span class="sh">"</span><span class="p">:</span> <span class="n">result</span>  <span class="c1"># The completed task
</span>        <span class="p">}</span>

        <span class="c1"># Push to Telex webhook
</span>        <span class="n">webhook_url</span> <span class="o">=</span> <span class="n">push_config</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">url</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">token</span> <span class="o">=</span> <span class="n">push_config</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">token</span><span class="sh">"</span><span class="p">)</span>

        <span class="n">headers</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">Content-Type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">application/json</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">Authorization</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Bearer </span><span class="si">{</span><span class="n">token</span><span class="si">}</span><span class="sh">"</span>
        <span class="p">}</span>

        <span class="k">async</span> <span class="k">with</span> <span class="n">httpx</span><span class="p">.</span><span class="nc">AsyncClient</span><span class="p">(</span><span class="n">timeout</span><span class="o">=</span><span class="mf">60.0</span><span class="p">)</span> <span class="k">as</span> <span class="n">client</span><span class="p">:</span>
            <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="n">client</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span>
                <span class="n">webhook_url</span><span class="p">,</span>
                <span class="n">json</span><span class="o">=</span><span class="n">webhook_payload</span><span class="p">,</span>
                <span class="n">headers</span><span class="o">=</span><span class="n">headers</span>
            <span class="p">)</span>
            <span class="n">response</span><span class="p">.</span><span class="nf">raise_for_status</span><span class="p">()</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">✅ Webhook push successful: </span><span class="si">{</span><span class="n">response</span><span class="p">.</span><span class="n">status_code</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">❌ Webhook push failed: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="c1"># Optionally send error to webhook
</span>        <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="nf">_push_error_to_telex</span><span class="p">(</span><span class="n">task_id</span><span class="p">,</span> <span class="n">message</span><span class="p">,</span> <span class="n">push_config</span><span class="p">,</span> <span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">))</span>
</code></pre>

</div>






<h2>
  
  
  🤖 LLM-Powered Analysis
</h2>

<p>The heart of PRRover is its multi-stage analysis pipeline. This function showcases the high-level steps, integrating both fast, rule-based checks and the deeper, slower LLM analysis.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># app/services/code_analyzer.py
</span><span class="k">async</span> <span class="k">def</span> <span class="nf">analyze_pr</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">pr_url</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">CodeAnalysisResult</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Analyze a GitHub Pull Request</span><span class="sh">"""</span>

    <span class="c1"># Step 1: Fetch PR data from GitHub (via MCP)
</span>    <span class="n">pr_data</span><span class="p">,</span> <span class="n">diff_content</span><span class="p">,</span> <span class="n">files</span> <span class="o">=</span> <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="nf">_fetch_pr_data</span><span class="p">(</span><span class="n">pr_url</span><span class="p">)</span>

    <span class="c1"># Step 2: Parse diff content
</span>    <span class="n">parsed_diffs</span> <span class="o">=</span> <span class="n">DiffParser</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="n">diff_content</span><span class="p">)</span>

    <span class="c1"># Step 3: Rule-based checks (fast, synchronous)
</span>    <span class="n">rule_security_issues</span> <span class="o">=</span> <span class="n">SecurityChecker</span><span class="p">.</span><span class="nf">check</span><span class="p">(</span><span class="n">file_changes</span><span class="p">)</span>
    <span class="n">rule_performance_issues</span> <span class="o">=</span> <span class="n">PerformanceChecker</span><span class="p">.</span><span class="nf">check</span><span class="p">(</span><span class="n">file_changes</span><span class="p">)</span>

    <span class="c1"># Step 4: LLM-powered analysis (comprehensive, slower)
</span>    <span class="n">llm_security_issues</span> <span class="o">=</span> <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="n">llm_service</span><span class="p">.</span><span class="nf">analyze_security</span><span class="p">(</span>
        <span class="n">diff_content</span><span class="p">,</span> <span class="n">files</span>
    <span class="p">)</span>
    <span class="n">llm_performance_issues</span> <span class="o">=</span> <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="n">llm_service</span><span class="p">.</span><span class="nf">analyze_performance</span><span class="p">(</span>
        <span class="n">diff_content</span><span class="p">,</span> <span class="n">files</span>
    <span class="p">)</span>
    <span class="n">llm_best_practice_issues</span> <span class="o">=</span> <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="n">llm_service</span><span class="p">.</span><span class="nf">analyze_best_practices</span><span class="p">(</span>
        <span class="n">diff_content</span><span class="p">,</span> <span class="n">files</span>
    <span class="p">)</span>

    <span class="c1"># Step 5: Generate executive summary
</span>    <span class="n">executive_summary</span> <span class="o">=</span> <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="n">llm_service</span><span class="p">.</span><span class="nf">generate_summary</span><span class="p">(</span>
        <span class="n">pr_title</span><span class="o">=</span><span class="n">pr_data</span><span class="p">.</span><span class="n">title</span><span class="p">,</span>
        <span class="n">pr_author</span><span class="o">=</span><span class="n">pr_data</span><span class="p">.</span><span class="n">user</span><span class="p">.</span><span class="n">login</span><span class="p">,</span>
        <span class="n">security_issues</span><span class="o">=</span><span class="n">all_security_issues</span><span class="p">,</span>
        <span class="n">performance_issues</span><span class="o">=</span><span class="n">all_performance_issues</span><span class="p">,</span>
        <span class="n">best_practice_issues</span><span class="o">=</span><span class="n">all_best_practice_issues</span>
    <span class="p">)</span>

    <span class="c1"># Step 6: Merge findings and calculate risk
</span>    <span class="n">risk_level</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_calculate_risk_level</span><span class="p">(</span>
        <span class="n">all_security_issues</span><span class="p">,</span> 
        <span class="n">all_performance_issues</span>
    <span class="p">)</span>

    <span class="k">return</span> <span class="nc">CodeAnalysisResult</span><span class="p">(...)</span>
</code></pre>

</div>



<h3>
  
  
  Why hybrid analysis?
</h3>

<ul>
<li>
<strong>Rule-based:</strong> Fast pattern matching for common issues (hardcoded secrets, SQL injection patterns).</li>
<li>
<strong>LLM-powered:</strong> Deep semantic understanding, context-aware recommendations, executive summaries.</li>
</ul>

<p><strong>LLM Model:</strong> Google Gemini 2.0 Flash Lite<br><br>
<strong>Typical latency:</strong> 30-40 seconds for a complete analysis</p>

<p>This is exactly why non-blocking mode matters—users get instant acknowledgement while the AI does its work.</p>


<h2>
  
  
  📸 PRRover in Action: Non-Blocking Success
</h2>

<p>The core benefit of the non-blocking A2A architecture is the user experience: giving instant acknowledgement while the heavy lifting (the 30-40 second LLM analysis) happens safely in the background.</p>

<p>In the example above, the agent immediately responds with an acknowledgement message: <em>"Analyzing the Pull Request for security vulnerabilities and performance issues! Please hold on a moment."</em></p>

<p>Once the full analysis of the <strong>PR #11:Feature/atomic property image upload</strong> is complete, the background task uses the webhook to push the comprehensive report directly into the Telex chat as a rich artifact. This provides a polished, non-disruptive, and high-fidelity result, ensuring the user is never left waiting for a timeout.</p>


<h2>
  
  
  🧪 Testing Your A2A Integration
</h2>

<p>I built a comprehensive test suite to validate A2A compliance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install dependencies</span>
pip <span class="nb">install </span>httpx

<span class="c"># Run tests</span>
python scripts/test_a2a_agent.py https://your-agent-url.com
</code></pre>

</div>



<p>The test suite validates:</p>

<ul>
<li>✅ <strong>Agent Card:</strong> Presence, format, required fields</li>
<li>✅ <strong>Endpoint Accessibility:</strong> <code>/a2a/tasks</code> accepts POST</li>
<li>✅ <strong>JSON-RPC Format:</strong> Valid <code>jsonrpc: "2.0"</code>, <code>result</code> or <code>error</code>
</li>
<li>✅ <strong>Task Structure:</strong> Correct <code>id</code>, <code>status</code>, <code>kind</code> fields</li>
<li>✅ <strong>Blocking Mode:</strong> Returns <code>completed</code> state</li>
<li>✅ <strong>Non-Blocking Mode:</strong> Returns <code>accepted</code> state</li>
<li>✅ <strong>Artifacts:</strong> Proper format with <code>kind: "text"</code>
</li>
</ul>

<h3>
  
  
  Example output:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>============================================================
📊 **GRADING REPORT**
============================================================
🎯 Score: 22/28 (78.6%)
✅ Passed: 11/14
❌ Failed: 3/14
⏱️  Execution Time: 10.23s
============================================================
</code></pre>

</div>



<p>PowerShell users can use the convenient runner:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="o">.</span><span class="n">\scripts\run_tests.ps1</span><span class="w"> </span><span class="nx">https://your-agent-url.com</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  🐛 Troubleshooting: Lessons Learned
</h2>

<h3>
  
  
  1. Webhook Format Confusion
</h3>

<p><strong>Problem:</strong> Webhook returns 400, Telex shows "could not find message"</p>

<p><strong>Solution:</strong> Telex expects a JSON-RPC <strong>response</strong> format for webhooks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// ❌ WRONG (request format)</span>
<span class="p">{</span>
  <span class="dl">"</span><span class="s2">jsonrpc</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">2.0</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">method</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">message/send</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">params</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span> <span class="p">...</span> <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// ✅ CORRECT (response format)</span>
<span class="p">{</span>
  <span class="dl">"</span><span class="s2">jsonrpc</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">2.0</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">rpc-id</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">result</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span> <span class="cm">/* task */</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Blocking Request Timeouts
</h3>

<p><strong>Problem:</strong> Long LLM analyses (30s+) timeout in blocking mode</p>

<p><strong>Solution:</strong> Use non-blocking mode for analyses greater than 10 seconds. Return "accepted" immediately, then push results via webhook using Python's background task facilities.</p>

<h3>
  
  
  3. Webhook Authentication
</h3>

<p><strong>Problem:</strong> Webhook POST returns 401 Unauthorized</p>

<p><strong>Solution:</strong> Check <code>pushNotificationConfig.authentication.schemes</code> and set the appropriate header:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">auth_schemes</span> <span class="o">=</span> <span class="n">push_config</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">authentication</span><span class="sh">"</span><span class="p">,</span> <span class="p">{}).</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">schemes</span><span class="sh">"</span><span class="p">,</span> <span class="p">[])</span>
<span class="k">if</span> <span class="sh">"</span><span class="s">Bearer</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">auth_schemes</span><span class="p">:</span>
    <span class="n">headers</span><span class="p">[</span><span class="sh">"</span><span class="s">Authorization</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Bearer </span><span class="si">{</span><span class="n">token</span><span class="si">}</span><span class="sh">"</span>
</code></pre>

</div>



<h3>
  
  
  4. Debugging Webhook Payloads
</h3>

<p>Add clear markers to logs for easy extraction:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">WEBHOOK_PAYLOAD_JSON_START</span><span class="sh">"</span><span class="p">)</span>
<span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">webhook_payload</span><span class="p">,</span> <span class="n">indent</span><span class="o">=</span><span class="mi">2</span><span class="p">))</span>
<span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">WEBHOOK_PAYLOAD_JSON_END</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>Then you can copy/paste and test with curl:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="s2">"https://ping.telex.im/v1/a2a/webhooks/..."</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer YOUR_TOKEN"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> @webhook_payload.json
</code></pre>

</div>






<h2>
  
  
  📈 Results &amp; Impact
</h2>

<h3>
  
  
  What works:
</h3>

<ul>
<li>✅ Instant acknowledgement for long-running tasks (&lt; 1s response)</li>
<li>✅ Comprehensive LLM-powered analysis (security, performance, best practices)</li>
<li>✅ Rich artifacts with CWE references and actionable recommendations</li>
<li>✅ 78.6% test pass rate (11/14 tests passing)</li>
<li>✅ Production-ready on Render.com</li>
</ul>

<h3>
  
  
  Sample Analysis Output:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✅ Code Review Complete - PR #123

Feature/atomic property image upload by @SamuelOshin

🟢 Risk: LOW | ✅ Approve

📊 16 files changed (+194/-59) • 20 issues found

Issues Found:
🔒 6 Security issues
⚡ 7 Performance issues
📐 7 Best Practice violations

💡 Top Concerns:
1. 📐 BEST PRACTICES: Focus on code organization (3 issues)

[Full detailed artifact with line numbers, CWE IDs, and fix suggestions]
</code></pre>

</div>






<h2>
  
  
  🚀 Next Steps &amp; Scaling
</h2>

<h3>
  
  
  Immediate improvements:
</h3>

<ul>
<li>[ ] Move LLM calls to Celery/RQ worker queue for horizontal scaling</li>
<li>[ ] Add Redis caching for PR diffs and LLM responses</li>
<li>[ ] Implement webhook retry with exponential backoff</li>
<li>[ ] Add structured logging (JSON) for better observability</li>
<li>[ ] Support multiple LLM providers (OpenAI, Anthropic)</li>
</ul>

<h3>
  
  
  Feature ideas:
</h3>

<ul>
<li>[ ] Support review comments directly on GitHub</li>
<li>[ ] Diff-level suggestions (not just file-level)</li>
<li>[ ] Team-specific rule customization</li>
<li>[ ] Historical trend analysis ("security issues down 40% this month")</li>
</ul>




<h2>
  
  
  🎓 Key Takeaways
</h2>

<ol>
<li>
<strong>A2A uses JSON-RPC 2.0</strong> — understand the request/response format differences.</li>
<li>
<strong>Non-blocking + webhooks = better UX</strong> — users don't wait for slow operations, thanks to Python's asyncio capabilities.</li>
<li>
<strong>Webhook payloads are responses, not requests</strong> — this is the #1 gotcha.</li>
<li>
<strong>Test everything</strong> — build automated tests early to catch protocol issues.</li>
<li>
<strong>Log everything</strong> — especially webhook payloads for debugging.</li>
</ol>

<h3>
  
  
  Resources:
</h3>

<ul>
<li><a href="https://github.com/SamuelOshin/code_reviewer_agent_a2a" rel="noopener noreferrer">PRRover Repository</a></li>
<li><a href="https://a2a-protocol.org" rel="noopener noreferrer">A2A Protocol Spec</a></li>
<li><a href="https://docs.telex.im" rel="noopener noreferrer">Telex Documentation</a></li>
<li><a href="https://fastapi.tiangolo.com" rel="noopener noreferrer">FastAPI Documentation</a></li>
</ul>




<h2>
  
  
  💬 Let's Connect
</h2>

<p>Building agents? Hit me up! I'd love to hear about your A2A integrations or help debug webhook issues 😄</p>

<p>Questions? Drop them in the comments below! 👇</p>

<ul>
<li>
<strong>GitHub:</strong> <a href="https://github.com/SamuelOshin" rel="noopener noreferrer">@SamuelOshin</a>
</li>
<li>
<strong>Twitter:</strong> <a href="https://x.com/oshin_samuel" rel="noopener noreferrer">@samuelOshin</a>
</li>
<li>
<strong>LinkedIn:</strong> <a href="http://linkedin.com/in/samuel-oshin-2903611a5/" rel="noopener noreferrer">Lets Connect</a>
</li>
</ul>

<p>Happy coding! 🚀</p>

