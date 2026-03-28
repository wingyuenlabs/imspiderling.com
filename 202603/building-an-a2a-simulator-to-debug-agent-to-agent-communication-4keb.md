---
Title: Building an A2A Simulator to Debug Agent-to-Agent Communication
Description: 
Author: AgentDM
Date: 2026-03-28T22:00:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>We've been building <a href="https://agentdm.ai" rel="noopener noreferrer">AgentDM</a>, a platform where AI agents talk to each other using Google's <a href="https://github.com/a2aproject/A2A" rel="noopener noreferrer">A2A (Agent-to-Agent) protocol</a>. Early on we hit a wall: debugging what actually happens between two agents during a conversation was painful. We could read logs and stare at JSON, but we couldn't <em>see</em> the conversation unfold in real time or manually control one side of it.</p>

<p>The A2A project has an <a href="https://github.com/a2aproject/a2a-inspector" rel="noopener noreferrer">Inspector tool</a> that lets you connect to an agent and send messages. It's useful for quick smoke tests, but it only acts as a client. You can talk <em>to</em> an agent, but you can't simulate the other agent talking back. For debugging the full round trip, especially the <code>input-required</code> back-and-forth pattern, we needed something that could play both roles.</p>

<p>So we built the <a href="https://github.com/agentdmai/a2a-simulator" rel="noopener noreferrer">A2A Simulator</a>.</p>

<h2>
  
  
  What the A2A protocol actually does
</h2>

<p>Before diving into the tool, it helps to understand how A2A conversations work. The protocol defines a task lifecycle with specific states, and agents communicate by moving tasks through these states.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9pm7zv8735bryqvebah5.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9pm7zv8735bryqvebah5.jpg" alt=" " width="800" height="320"></a></p>

<p>When Agent Alpha sends a message to Agent Beta, it creates a <strong>task</strong>. That task starts in <code>submitted</code> state. Agent Beta receives the message and decides what to do with it. From there, the task can move through several states:</p>

<p><strong>Working</strong> means the agent is processing. Think of it like a "typing..." indicator, except the agent can send partial results while in this state. An agent might send three or four <code>working</code> updates before finishing, each with a progress message.</p>

<p><strong>Input Required</strong> is the interesting one. It means the agent has a question or needs more information before it can continue. The sending agent sees this and can reply with additional context. This creates a back-and-forth conversation within a single task, which is exactly the pattern that's hardest to debug.</p>

<p><strong>Completed</strong> and <strong>Failed</strong> are terminal states. Once a task reaches either of these, the conversation on that task is over.</p>

<p><strong>Canceled</strong> happens when the sending agent decides to abort a task mid-flight.</p>

<p>The protocol also supports <strong>artifacts</strong>, which are named attachments (files, data, structured output) that an agent can include with any response. And everything happens over streaming, so you see updates in real time rather than waiting for a final response.</p>

<h2>
  
  
  Why the Inspector wasn't enough
</h2>

<p>The Inspector connects to an agent and sends messages. That covers one direction. But when you're building an agent that needs to handle incoming messages, ask follow-up questions via <code>input-required</code>, and manage multi-turn conversations, you need to be the agent on the other side too.</p>

<p>We kept running into scenarios like: "Agent Beta asked for clarification, but did Agent Alpha actually receive the <code>input-required</code> status? Did the follow-up message land on the same task or create a new one? What did the raw JSON-RPC look like?"</p>

<p>Reading server logs worked, but it was slow and disconnected from the actual conversation flow. We wanted a chat UI where we could see both sides, control the responses manually, and inspect the wire protocol when something looked wrong.</p>

<h2>
  
  
  How the Simulator works
</h2>

<p>Each simulator instance runs as both an A2A server and client on a single port. It hosts its own agent card at <code>/.well-known/agent-card.json</code> and accepts incoming messages, while also being able to connect to and send messages to another agent.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9b9w142a2esyqwdx55rq.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9b9w142a2esyqwdx55rq.jpg" alt=" " width="800" height="400"></a></p>

<p>The UI is a chat interface where you act as the human behind the agent. When a remote agent sends you a message, it appears in the Incoming tab. You type a response, pick a state from a dropdown (<code>working</code>, <code>completed</code>, <code>input-required</code>, <code>failed</code>), and optionally attach artifacts. Your reply flows back through the A2A protocol to the sender.</p>

<p>Every message has a "View raw" link that opens a drawer showing the actual JSON-RPC request and response. This is where you catch the subtle bugs: wrong <code>contextId</code> values, missing fields, unexpected event ordering.</p>

<h2>
  
  
  Setting it up
</h2>

<p>Clone the repo and install dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/agentdmai/a2a-simulator.git
<span class="nb">cd </span>a2a-simulator
npm <span class="nb">install</span>
</code></pre>

</div>



<p>Open two terminal windows and start two instances:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Terminal 1</span>
npm run dev <span class="nt">--</span> <span class="nt">--port</span> 3000 <span class="nt">--name</span> <span class="s2">"Agent Alpha"</span>

<span class="c"># Terminal 2</span>
npm run dev <span class="nt">--</span> <span class="nt">--port</span> 3001 <span class="nt">--name</span> <span class="s2">"Agent Beta"</span>
</code></pre>

</div>



<p>Open your browser to <code>http://localhost:5173</code>. This is Agent Alpha's UI (Vite proxies to port 3000). In the connection panel on the left, type <code>http://localhost:3001</code> and click Connect. You should see Agent Beta's agent card appear, confirming the connection.</p>

<p>Now type a message and hit Send. Switch to Agent Beta's perspective (open another browser tab pointed at Beta's Vite dev server, or build and serve both). You'll see the message appear in Beta's Incoming tab. Pick a state, type a response, and send it back. Watch it appear on Alpha's side in real time.</p>

<h2>
  
  
  Simulating a multi-turn conversation
</h2>

<p>Here's where it gets interesting. Try this sequence:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqy5jscxsmoj2k89k0e7r.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqy5jscxsmoj2k89k0e7r.jpg" alt=" " width="700" height="520"></a></p>

<ol>
<li>From Alpha, send "What's the weather?"</li>
<li>On Beta, select <code>working</code> from the dropdown and reply "Checking forecast data..."</li>
<li>On Beta, select <code>working</code> again and reply "Found 3 matching stations"</li>
<li>On Beta, select <code>input-required</code> and reply "Which city did you mean? I found New York, New York City, and New York Mills."</li>
<li>Back on Alpha, you'll see the <code>input-required</code> status. Click the message to reply, then type "New York City" and send</li>
<li>On Beta, the follow-up arrives on the same task. Select <code>completed</code> and reply "72°F and sunny in New York City"</li>
</ol>

<p>This entire exchange happens on a single task, with the task status moving through <code>working</code> → <code>working</code> → <code>input-required</code> → <code>completed</code>. You can click "View raw" on any message to see exactly what went over the wire at each step.</p>

<h2>
  
  
  What we learned building this
</h2>

<p>The biggest surprise was how many edge cases exist around <code>contextId</code> handling. When Agent Alpha replies to an <code>input-required</code> task, the follow-up message needs to reference the original task's <code>contextId</code> or the SDK creates a new task instead of continuing the existing one. We found this bug <em>using</em> the simulator, which is exactly the kind of thing it was built for.</p>

<p>We also discovered that the <code>@a2a-js/sdk</code> streams multiple events for terminal states. A single <code>completed</code> reply from the server can generate both a status-update event and a task snapshot event, each carrying the same message. Without deduplication on the client side, the reply shows up twice. Again, visible immediately in the simulator, would have taken much longer to catch from logs alone.</p>

<h2>
  
  
  Other features worth mentioning
</h2>

<p><strong>Artifacts.</strong> Attach named artifacts with MIME types to any response. They show up inline in the conversation thread. Useful for testing agents that return structured data or files.</p>

<p><strong>Authentication.</strong> Configure bearer token authentication on your instance through the Agent Card editor. When connecting to a remote agent that requires auth, paste the token in the connection panel.</p>

<p><strong>Agent Card editing.</strong> Modify your agent's name, description, skills, and auth settings on the fly without restarting. Changes take effect immediately for the next incoming connection.</p>

<h2>
  
  
  Try it out
</h2>

<p>The simulator is open source under MIT license: <a href="https://github.com/agentdmai/a2a-simulator" rel="noopener noreferrer">github.com/agentdmai/a2a-simulator</a></p>

<p>If you're integrating the A2A protocol into your own agents, give it a spin. It's faster than reading logs and more flexible than the Inspector for debugging the full conversation lifecycle. We use it daily while building <a href="https://agentdm.ai" rel="noopener noreferrer">AgentDM</a>, and it's caught more bugs than we'd like to admit.</p>




<p><em>Built by <a href="https://agentdm.ai" rel="noopener noreferrer">AgentDM</a></em></p>

