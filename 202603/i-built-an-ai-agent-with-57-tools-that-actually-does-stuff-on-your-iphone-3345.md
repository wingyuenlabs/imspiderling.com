---
Title: I built an AI agent with 57 tools that actually does stuff on your iPhone
Description: 
Author: Denis Babkevich
Date: 2026-03-05T21:46:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every AI app on the App Store right now is basically the same thing: a text box that talks to GPT. You type, it replies, you type again. That's a chatbot, not an agent.</p>

<p>I've been building <a href="https://spectrion.app" rel="noopener noreferrer">Spectrion</a> for the past year and the whole point was to make something that actually <em>does things</em>. Not just talks about doing things. You say "find me a good restaurant nearby, check the weather, and set a reminder for 7pm" and it figures out the steps, calls the right tools, handles the results, and tells you when it's done. No hand-holding.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjlz8fdg0escvr4sgx1m7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjlz8fdg0escvr4sgx1m7.png" alt="Spectrion chat interface" width="800" height="1739"></a></p>

<h2>
  
  
  The agent loop
</h2>

<p>The core idea is dead simple. Instead of one request -&gt; one response, you have a loop:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>messages = [userMessage]
loop {
    response = llm.generate(messages)
    if response.hasToolCalls {
        results = execute(response.toolCalls)
        messages.append(response)
        messages.append(results)
        continue
    } else {
        display(response.text)
        break
    }
}
</code></pre>

</div>



<p>The model calls tools, sees the results, decides what to do next. Maybe it takes 3 iterations, maybe 15. The loop runs until the model decides it's done (safety limit at 200, but I've never seen it go past ~30).</p>

<p>Here's what a real interaction looks like internally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User: "Find the best Italian place near me and remind me at 7pm"

Iteration 1: calls device_info(type: "location") -&gt; gets coordinates
Iteration 2: calls web_search("italian restaurant near 37.78,-122.41") -&gt; gets results
Iteration 3: calls web_fetch(url of top result) -&gt; checks reviews
Iteration 4: calls reminders(create, "Dinner at Trattoria Roma", 7pm) -&gt; done
Iteration 5: responds with summary
</code></pre>

</div>



<p>Four tool calls, zero user input after the first message. That's the difference between a chatbot and an agent.</p>

<h2>
  
  
  Compound requests and the todo system
</h2>

<p>Simple 5-step tasks are easy. The hard part is compound requests -- "research three competitors, compare pricing, build a tracker, and write a summary." That's where most agent implementations fall apart. The model does step 1, maybe step 2, then returns a half-baked answer.</p>

<p>We solved this with a <strong>todo system</strong>. When the agent gets a compound request, it creates a todo list with all subtasks, works through them one by one, checks items off as it goes. If the model stops responding but there are still pending items, the runtime nudges it: "you've got unfinished items, keep going." Up to 20 auto-continues before giving up.</p>

<p>There's a hierarchy of nudges that keep the agent on track:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Nudge</th>
<th>When it fires</th>
<th>What it does</th>
</tr>
</thead>
<tbody>
<tr>
<td>Todo-aware</td>
<td>Model stopped, pending items exist</td>
<td>"Uncompleted items remain, continue"</td>
</tr>
<tr>
<td>Todo nag</td>
<td>3 iterations without updating todo</td>
<td>Reminder to update the list</td>
</tr>
<tr>
<td>Action-promise</td>
<td>Model says "let me try" without calling tools</td>
<td>Deletes the empty promise, re-nudges</td>
</tr>
<tr>
<td>Auto-continue</td>
<td>Empty response after tool execution</td>
<td>Injects "[continue]"</td>
</tr>
</tbody>
</table></div>

<p>The result: you can throw a 7-step request at it and walk away. It grinds through all of them.</p>

<h2>
  
  
  Watchdogs and self-healing
</h2>

<p>An agent that hangs is worse than one that fails fast. So there's a watchdog system:</p>

<ul>
<li>
<strong>Iteration watchdog</strong> -- 600-second timeout per iteration. If the model is stuck, it aborts and retries (up to 3 attempts with backoff).</li>
<li>
<strong>Repetition detector</strong> -- catches when the model outputs 600+ characters of repeated text. Truncates and retries.</li>
<li>
<strong>Context overflow recovery</strong> -- when the conversation gets too long and the API returns 400/413, the runtime runs a 4-step recovery: truncate oversized tool results, compact old messages, emergency trim, retry. All automatic.</li>
<li>
<strong>Consecutive error tracking</strong> -- if 5 tool calls in a row return errors, the loop aborts instead of burning tokens in circles.</li>
</ul>

<p>Every retry uses exponential backoff. The model never sees that any of this happened -- it gets a clean context and continues.</p>

<h2>
  
  
  Parallel everything
</h2>

<p>When the model calls 3 tools in one turn, they run simultaneously via <code>TaskGroup</code>. Not sequentially -- in parallel. A web search, a calendar check, and a location lookup all fire at the same time. This cuts latency by 2-3x on multi-tool turns.</p>

<p>You can also run <strong>multiple conversations in parallel</strong>. The runtime uses per-conversation state isolation, so conversation A's agent loop doesn't interfere with conversation B. Three main execution lanes, three sub-agent lanes, separate lanes for cron and nested calls. Each conversation is pinned at entry -- no cross-chat message leaks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Execution Lanes:
  main:     [conv A] [conv B] [conv C]     &lt;- 3 parallel conversations
  subagent: [task 1] [task 2] [task 3]     &lt;- 3 parallel sub-agents
  cron:     [scheduled task]               &lt;- background jobs
  nested:   [nested call] [nested call]    &lt;- agent-in-agent
</code></pre>

</div>



<h2>
  
  
  57 tools across 8 categories
</h2>

<p>An agent without tools is just a chatbot with extra steps. Here's what Spectrion can actually do on your phone:</p>

<p><strong>Web &amp; Search</strong> -- search the web, fetch and parse pages, extract content</p>

<p><strong>Communication</strong> -- send iMessages/SMS, make phone calls, read/write contacts, send emails</p>

<p><strong>Calendar &amp; Reminders</strong> -- full CRUD on events and reminders, recurring tasks, cron-style scheduling</p>

<p><strong>Files &amp; Documents</strong> -- browse filesystem, cloud files, zip/unzip, parse XLSX/DOCX/CSV natively, local notes with tags</p>

<p><strong>Media &amp; Vision</strong> -- take photos, record audio, <strong>AI vision</strong> (describe/OCR images and video), <strong>AI image editing</strong> (background removal, object removal, style transfer), <strong>AI image generation</strong> (text-to-image), scan/generate barcodes and QR codes</p>

<p><strong>System &amp; Device</strong> -- device info, screen brightness, location, app launcher, Siri Shortcuts, Apple Health data (steps/heart rate/sleep/workouts), music playback, maps (directions/nearby/geocoding), clipboard, share sheet, passwords, text transforms (base64/hashing/regex/JSON), translation, calculator, timers</p>

<p><strong>AI &amp; Meta</strong> -- create tools at runtime, manage skills, long-term memory, delegate to sub-agents, render dynamic UI (Canvas)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F28xnktjl9vfrq5jlt52f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F28xnktjl9vfrq5jlt52f.png" alt="57 tools across 8 categories" width="800" height="1739"></a></p>

<p>Not all 57 tools are loaded into the prompt at once -- that would waste tokens. There's a <code>ToolCatalog</code> that keeps core tools always active and activates others on demand. The catalog matches keywords from your message in <strong>11 languages</strong> to figure out which tools you need. Say "photo" in English, "foto" in German, or "фото" in Russian -- the camera tool activates automatically.</p>

<h2>
  
  
  Canvas: the agent builds UI
</h2>

<p>The agent can render <strong>interactive UIs</strong> from JSON -- not just text responses. It's called A2UI (Agent-Driven UI).</p>

<p>The agent outputs a declarative JSON structure, and the runtime renders it as native SwiftUI:</p>

<ul>
<li>
<strong>Layout</strong>: VStack, HStack, ZStack, ScrollView, Grid</li>
<li>
<strong>Input</strong>: Buttons, TextFields, Toggles, Sliders, Pickers, Steppers</li>
<li>
<strong>Data</strong>: Lists, Tables, Charts</li>
<li>
<strong>Media</strong>: Maps, WebViews</li>
<li>
<strong>Composite</strong>: Cards, Alerts, Sheets, Forms, Progress indicators</li>
</ul>

<p>There's a state management system and event bus so the UI is actually interactive -- form submissions, button clicks, slider changes all feed back to the agent. Ask it to "build me a unit converter" and it renders a working app-within-the-app.</p>

<h2>
  
  
  Sub-agents
</h2>

<p>Some tasks need more than one brain. The main agent can spin up sub-agents -- each gets its own tool executor, its own message history, its own loop (up to 50 iterations).</p>

<p>Four built-in personas:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Agent</th>
<th>What it does</th>
</tr>
</thead>
<tbody>
<tr>
<td>Researcher</td>
<td>Deep web research across multiple sources</td>
</tr>
<tr>
<td>Coder</td>
<td>Writes and debugs code</td>
</tr>
<tr>
<td>Writer</td>
<td>Content, creative writing, editing</td>
</tr>
<tr>
<td>Tool Builder</td>
<td>Creates new JS tools from scratch</td>
</tr>
</tbody>
</table></div>

<p>You can dispatch up to 5 tasks in parallel. Sub-agents can message each other across sessions -- one agent's output feeds into another's next run via <code>SessionMessenger</code>.</p>

<h2>
  
  
  Runtime tool creation
</h2>

<p>The agent can build its own tools. Not in theory -- it literally writes JavaScript, tests it, fixes bugs, and registers a working tool. All at runtime.</p>

<p>The JS sandbox has three API tiers:</p>

<ul>
<li>
<strong>Core</strong>: HTTP requests, key-value storage, crypto, tool chaining (<code>call_tool()</code>)</li>
<li>
<strong>Extended</strong>: CommonJS modules, filesystem, HTML parsing, polling</li>
<li>
<strong>Advanced</strong>: SQLite database, device queries, image processing</li>
</ul>

<p>Each tool gets isolated storage (its own KV store + SQLite). Everything is versioned -- every edit saves the old version with full rollback. There are 20+ pre-built templates to start from.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// A tool the agent might create:</span>
<span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="nf">http</span><span class="p">({</span>
    <span class="na">url</span><span class="p">:</span> <span class="s2">`https://api.github.com/repos/</span><span class="p">${</span><span class="nx">owner</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">repo</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">Accept</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/vnd.github.v3+json</span><span class="dl">'</span> <span class="p">}</span>
<span class="p">});</span>
<span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">body</span><span class="p">);</span>
<span class="k">return</span> <span class="s2">`</span><span class="p">${</span><span class="nx">data</span><span class="p">.</span><span class="nx">full_name</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">data</span><span class="p">.</span><span class="nx">stargazers_count</span><span class="p">}</span><span class="s2"> stars`</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  Workflows: automation chains
</h2>

<p>Beyond single tool calls, you can build <strong>multi-step workflows</strong> that chain tools together:</p>

<ul>
<li>
<strong>Trigger nodes</strong>: manual, scheduled (cron), or event-driven</li>
<li>
<strong>Action nodes</strong>: any tool call with templated arguments (<code>${variable}</code>)</li>
<li>
<strong>Condition nodes</strong>: branching logic based on previous results</li>
<li>
<strong>LLM nodes</strong>: inject AI reasoning at any point</li>
<li>
<strong>HTTP / JavaScript nodes</strong>: call external APIs or run custom logic</li>
<li>
<strong>Loop and parallel branches</strong>: repeat or fan out</li>
</ul>

<p>The agent can build workflows for you or you can design them manually.</p>

<h2>
  
  
  Device Mesh: your devices become one agent
</h2>

<p>This is the big one. Spectrion runs on iPhone, iPad, and Mac -- and with Device Mesh, all your devices become a <strong>single unified agent</strong>.</p>

<p>Here's how it works: you pair two devices (QR code scan or pairing code). They perform an ECDH key exchange (Curve25519), derive a shared AES-256-GCM key, and establish an encrypted WebSocket tunnel through the server. From that point on, everything syncs in real time:</p>

<ul>
<li>
<strong>Conversations and messages</strong> -- start a chat on iPhone, continue on Mac</li>
<li>
<strong>Settings and persona</strong> -- change the agent's personality on one device, it updates everywhere</li>
<li>
<strong>Custom tools</strong> -- create a JS tool on Mac, it appears on iPhone</li>
<li>
<strong>Memory</strong> -- what the agent learned on one device is available on all</li>
<li>
<strong>Scheduled tasks</strong> -- cron jobs sync across devices</li>
<li>
<strong>Knowledge base</strong> -- documents indexed on one device are searchable from another</li>
</ul>

<p>But the killer feature is <strong>cross-device tool execution</strong>. When your iPhone is paired with your Mac, the agent on Mac can call <code>remote_camera</code> -- and it fires the actual camera on your iPhone. The agent on iPhone can call <code>remote_file_manager</code> -- and it reads files from your Mac's filesystem.</p>

<p>Every remote tool appears in the catalog with a <code>remote_</code> prefix. The LLM doesn't know or care that the tool runs on a different device -- it just calls it. Under the hood, it's an RPC call over the encrypted WebSocket:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Mac agent calls remote_camera(action: "capture")
  -&gt; MeshManager sends RPC request via encrypted WebSocket
  -&gt; iPhone receives, checks tool policy (allowList)
  -&gt; iPhone executes camera tool locally
  -&gt; Result (including image data) sent back encrypted
  -&gt; Mac agent receives result, continues loop
</code></pre>

</div>



<p>The sync engine uses a <strong>Hybrid Logical Clock (HLC)</strong> for conflict-free merging -- no "last write wins" disasters. Deltas are batched (200ms debounce) and sent as they happen. Offline changes queue up and sync when the device reconnects.</p>

<p>All of this is end-to-end encrypted. The server relays WebSocket messages but can't read them -- it never has the symmetric key.</p>

<h2>
  
  
  Remote CLI: deploy your agent to Linux servers
</h2>

<p>With an Ultra subscription, you can deploy Spectrion CLI instances as Docker containers on your own Linux servers. The <code>remote_cli</code> tool handles everything:</p>

<ul>
<li>
<strong>Deploy</strong>: spin up a new container on any server via SSH</li>
<li>
<strong>Manage</strong>: list, status, logs, restart, stop, remove</li>
<li>
<strong>Execute</strong>: run shell commands on the remote instance</li>
<li>
<strong>Update</strong>: pull latest container image</li>
</ul>

<p>This means your agent can run 24/7 on a VPS, executing long-running tasks, monitoring services, or doing heavy compute -- all controlled from your phone.</p>

<h2>
  
  
  Evolution: the agent improves itself
</h2>

<p>The Evolution Engine is an autonomous self-improvement system. Every 24 hours (configurable), it:</p>

<ol>
<li>
<strong>Collects metrics</strong> -- tool success rates, tools per conversation, common task patterns, user satisfaction signals</li>
<li>
<strong>Analyzes via LLM</strong> -- sends metrics to the model with a prompt asking for improvements</li>
<li>
<strong>Safety gate</strong> -- filters out any dangerous or nonsensical changes</li>
<li>
<strong>Applies improvements</strong> -- updates system prompt, persona config, temperature</li>
<li>
<strong>Auto-creates tools</strong> -- if it notices you frequently do something that could be automated, it builds a tool for it</li>
</ol>

<p>Every evolution cycle saves a versioned snapshot. Don't like what it did? Roll back to any previous version instantly. The entire evolution history syncs across devices via Mesh.</p>

<p>The agent literally gets better at helping you the more you use it. Not through fine-tuning -- through runtime self-modification of its own prompt and configuration.</p>

<h2>
  
  
  The proxy: bring your own model or use ours
</h2>

<p>Out of the box, Spectrion works through our own proxy infrastructure -- no API keys needed, just open the app and go. The proxy handles account management, rate limit avoidance, model routing, and token refresh transparently.</p>

<p>If you want full control, plug in your own providers:</p>

<ul>
<li>Anthropic / OpenAI / any OpenAI-compatible API key</li>
<li>Local models via Ollama (fully offline, nothing leaves your phone)</li>
<li>Apple Foundation Models (on-device, instant for simple tasks)</li>
</ul>

<p>The system auto-falls back between providers if one fails. Vision tasks route to vision-capable models. Sub-agents get cheaper models so you're not burning premium tokens on web lookups.</p>

<p>The backend maintains a pool of provider accounts with utilization-aware load balancing. If one account gets rate-limited, the system switches to another mid-conversation. A tier system (HIGH/MEDIUM/LOW) routes requests to the right model class. Token refresh runs every 5 minutes in the background. Two production servers with replica sync. The result: zero user-facing rate limit errors.</p>

<h2>
  
  
  Heartbeat: your agent works while you sleep
</h2>

<p>Schedule background tasks and the agent runs them autonomously:</p>

<ul>
<li>"Check my email summary at 8am daily"</li>
<li>"Monitor this GitHub repo for new issues every hour"</li>
<li>"Remind me about pending tasks when I open the app"</li>
</ul>

<p>Smart check-in logic -- only pings the model when there's actual work to do, only during active hours (default 8am-11pm). On app launch, it checks for unfinished tasks and picks up where it left off.</p>

<h2>
  
  
  Voice mode
</h2>

<p>A wake-word engine runs on-device ML, detects your trigger phrase, captures speech (15 seconds), runs it through the full agent loop, and reads back the response via TTS. Fully hands-free. Wake-word detection, speech recognition, and TTS all run on-device.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8fgvp1ri2obk0u5is4v5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8fgvp1ri2obk0u5is4v5.png" alt="Voice mode" width="800" height="1739"></a></p>

<h2>
  
  
  Memory, knowledge base, and everything else
</h2>

<p><strong>Long-term memory</strong> -- the agent decides what to remember through a dedicated tool. The model determines what's worth storing. Memories persist across conversations. Auto-compacts when too large. PII gets filtered.</p>

<p><strong>Knowledge base (RAG)</strong> -- upload PDFs, DOCX, images. The system chunks them, computes vector embeddings, and indexes everything. Hybrid search: semantic + keyword matching.</p>

<p><strong>MCP support</strong> -- connect any <a href="https://modelcontextprotocol.io/" rel="noopener noreferrer">Model Context Protocol</a> server and its tools appear in the catalog instantly. Streamable HTTP transport with SSE, automatic retries, protocol versions 2024-11-05 and 2025-03-26.</p>

<p><strong>Plugins</strong> -- a hot-reload plugin system. Discover, load, enable/disable, unload. Plugin tools show up in the same catalog as everything else.</p>

<p><strong>Channels</strong> -- connect a Telegram bot and the agent responds there too. Bidirectional -- full tool loop runs on incoming messages. Infrastructure supports SMS/WhatsApp gateways.</p>

<p><strong>Skills</strong> -- reusable instruction sets that inject into the system prompt when triggered. Built-in: <code>web_researcher</code>, <code>scheduler</code>. Create your own or download from the community store.</p>

<p><strong>iOS integration</strong> -- Siri App Intents, home screen widgets, deep links (<code>spectrion://new-chat</code>), Spotlight search.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F270d60xuscmpfrlkmexp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F270d60xuscmpfrlkmexp.png" alt="Extensions and connections" width="800" height="1739"></a></p>

<h2>
  
  
  The stack
</h2>

<p>For the curious:</p>

<ul>
<li>
<strong>App</strong>: SwiftUI, <code>@Observable</code> (no Combine), async/await, SwiftData, Keychain. Zero third-party dependencies -- no SPM, no CocoaPods. Everything is first-party Apple frameworks.</li>
<li>
<strong>Mesh</strong>: WebSocket transport, Curve25519 ECDH key agreement, AES-256-GCM encryption, Hybrid Logical Clock sync, RPC for remote tool execution.</li>
<li>
<strong>Concurrency</strong>: Tool executor is an actor. <code>Sendable</code> everywhere. <code>TaskGroup</code> for parallel tools. Execution lanes for resource control. Per-conversation state isolation.</li>
<li>
<strong>Backend</strong>: Node.js + Express + SQLite (WAL) + Redis. Account pooling, tier routing, replica sync between servers.</li>
<li>
<strong>Reliability</strong>: Watchdog timers, retry with backoff, context overflow recovery, repetition detection, todo-aware nudges. The agent self-heals.</li>
<li>
<strong>Privacy</strong>: Keys in Keychain only. Speech/wake-word on-device. E2E encryption for mesh. No conversation logging on server. No telemetry.</li>
<li>
<strong>Localization</strong>: 11 languages. Tool auto-activation works in all of them.</li>
</ul>

<h2>
  
  
  What's next
</h2>

<ul>
<li>Android and desktop clients (architecture separates platform code from agent logic)</li>
<li>Community marketplace for sharing custom tools and skills (infrastructure built, store is live)</li>
<li>More MCP integrations as the ecosystem grows</li>
<li>Deeper OS integration</li>
</ul>

<h2>
  
  
  Try it
</h2>

<p><a href="https://apps.apple.com/app/spectrion-agent-ai/id6759151825" rel="noopener noreferrer">App Store</a> | <a href="https://spectrion.app" rel="noopener noreferrer">spectrion.app</a></p>

<p>If you've been looking for an AI app that goes past the text box -- that actually takes action, chains tasks, builds its own tools, connects your devices into one brain, and evolves to serve you better -- give it a shot. Happy to answer architecture questions in the comments.</p>

