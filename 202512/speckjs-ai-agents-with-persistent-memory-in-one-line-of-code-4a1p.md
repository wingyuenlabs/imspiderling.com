---
Title: Speck.js: AI Agents with Persistent Memory in One Line of Code
Description: 
Author: Peter Pagano@SpeckOs
Date: 2025-12-15T21:12:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>Adding AI to web apps typically requires 50+ lines of boilerplate. State for messages, loading states, error handling, API calls, stream management, history tracking — the list goes on.</p>

<p>Speck.js eliminates all of that.</p>

<h2>
  
  
  One Line. Full AI Chat.
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;Agent.Chat</span> <span class="na">purpose=</span><span class="s">"You are a helpful assistant"</span> <span class="nt">/&gt;</span>
</code></pre>

</div>



<p>That's a complete chat interface with streaming responses, conversation history, and error handling. No state management. No useEffect. No boilerplate.</p>

<h2>
  
  
  Persistent Memory — A Framework First
</h2>

<p>Speck.js v0.4.1 introduces something no other framework offers: built-in persistent memory.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;Agent.Chat</span> 
  <span class="na">id=</span><span class="s">"support-bot"</span>
  <span class="na">purpose=</span><span class="s">"You are a helpful assistant"</span>
  <span class="na">memory=</span><span class="s">{true}</span>
<span class="nt">/&gt;</span>
</code></pre>

</div>



<p>With <code>memory={true}</code>, agents remember conversations across browser sessions. Page refreshes, browser restarts, even system reboots — the memory persists. No external database configuration required.</p>

<h3>
  
  
  How Memory Works
</h3>

<ul>
<li>Session IDs stored in localStorage</li>
<li>Conversations saved to local SQLite via libSQL</li>
<li>Last 100 interactions automatically loaded into context</li>
<li>Each agent maintains isolated memory via the <code>id</code> prop</li>
</ul>

<h2>
  
  
  Zero Imports
</h2>

<p>Speck.js components auto-discover each other:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;App&gt;</span>
  <span class="nt">&lt;Header</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;ChatBot</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;Footer</span> <span class="nt">/&gt;</span>
<span class="nt">&lt;/App&gt;</span>
</code></pre>

</div>



<p>No import statements. The compiler resolves dependencies automatically.</p>

<h2>
  
  
  Getting Started
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm create speck-app@latest my-app
<span class="nb">cd </span>my-app
npm run dev
</code></pre>

</div>



<p>Add an Anthropic API key to <code>.env</code> and start building AI-native applications immediately.</p>

<h2>
  
  
  Developer Tools
</h2>

<p>Speck.js includes full tooling support:</p>

<ul>
<li>🧩 <strong>VS Code Extension</strong> — Syntax highlighting and IntelliSense for <code>.speck</code> files</li>
<li>⚡ <strong>Hot Reload</strong> — Instant compilation on save</li>
<li>🛠️ <strong>Built-in Dev Server</strong> — API proxy included for seamless Claude integration</li>
</ul>

<h2>
  
  
  Links
</h2>

<ul>
<li>🌐 <strong>Website:</strong> <a href="https://speckjs.dev" rel="noopener noreferrer">speckjs.dev</a>
</li>
<li>📦 <strong>npm:</strong> <a href="https://www.npmjs.com/package/create-speck-app" rel="noopener noreferrer">create-speck-app</a>
</li>
<li>📖 <strong>Documentation:</strong> <a href="https://speckjs.dev/docs.html" rel="noopener noreferrer">speckjs.dev/docs.html</a>
</li>
</ul>

<h2>
  
  
  On the Roadmap
</h2>

<ul>
<li>Semantic memory recall via vector search</li>
<li>Multi-agent coordination</li>
<li>Additional agent types beyond chat</li>
</ul>




<p>Feedback welcome. What features would be most valuable?<a href="https://dev.tourl"></a></p>

