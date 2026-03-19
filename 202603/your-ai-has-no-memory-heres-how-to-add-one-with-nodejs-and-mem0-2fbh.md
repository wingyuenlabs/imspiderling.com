---
Title: Your AI has no memory. Here's How to Add One with Node.js and Mem0
Description: 
Author: Ayomide olofinsawe
Date: 2026-03-19T22:01:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every chat interface you've ever used feels like the AI remembers you. It doesn't. There's no memory, no session, no awareness of previous conversations. What you're seeing is your application feeding the entire conversation history back into every request. The model just processes whatever is in front of it and forgets everything the moment it responds.</p>

<p>That approach works, but it has real ceilings. The message array grows with every exchange, so longer conversations mean larger payloads and higher token costs. You'll hit context window limits eventually. And across sessions, everything is lost start a new conversation, and the user has to repeat themselves from scratch. There's also no intelligence to it: a throwaway message like "ok thanks" carries the same weight as "I'm building a fintech app in Node.js, and I hate ORMs."</p>

<p>What you actually want is a system that extracts the facts that matter, stores them, and retrieves only the relevant ones when needed without stuffing the entire conversation history into every request.</p>

<p>That's what Mem0 does. It sits between your application and your LLM as a dedicated memory layer: automatically pulling meaningful facts out of conversations, persisting them across sessions, and injecting the right context into future requests.</p>

<p>In this tutorial, we'll build a memory-aware REST API from scratch using:</p>

<ul>
<li>
<strong>Express</strong> and <strong>TypeScript</strong> for the API layer</li>
<li>
<strong>Groq</strong> as the LLM provider (fast inference, generous free tier)</li>
<li>
<strong>Mem0</strong> as the memory layer that ties it all together
By the end, you'll have a working API where a user can send a message, have relevant facts automatically extracted and stored, and receive responses informed by everything the system has learned about them across previous conversations.</li>
</ul>

<p>Let's get into it.</p>

<h2>
  
  
  What is Mem0 and How Does It Work
</h2>

<p>Mem0 is a memory layer for AI applications. The core idea is simple: instead of your application managing conversation history and replaying it on every request, Mem0 handles the memory side of things for you extracting what matters, storing it, and making it available when it's relevant.</p>

<h3>
  
  
  How it differs from storing chat history
</h3>

<p>When you store chat history, you're storing everything every message, in order, as it happened. Nothing is filtered, nothing is prioritized, and the whole thing gets sent back to the model on the next request regardless of whether any of it is actually useful.<br>
Mem0 works differently. After each exchange, it processes the conversation and pulls out meaningful facts things like user preferences, stated goals, technical context, or any detail worth remembering long term. Those facts are stored as discrete memory objects, not as raw messages. When a new request comes in, Mem0 retrieves only the memories that are relevant to it and surfaces them to your LLM as context.<br>
The practical difference is significant. Instead of a growing message array that the model has to wade through, the model gets a compact, relevant summary of what it actually needs to know about the user.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvihnwcka0x6vbkd0z4xh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvihnwcka0x6vbkd0z4xh.png" alt="Tradioton llm approach vs mem0 approach" width="800" height="533"></a></p>
<h3>
  
  
  Async processing and automatic fact extraction
</h3>

<p>Memory extraction in Mem0 happens asynchronously  it doesn't block your API response. After your LLM replies, Mem0 processes the conversation in the background, extracts facts, and updates the memory store. It also handles deduplication, so if a user mentions the same detail across multiple conversations, Mem0 won't create redundant memory entries  it updates the existing one instead.<br>
This means your API stays fast, and the memory store stays clean over time without any extra work on your end.</p>
<h2>
  
  
  Project Setup
</h2>

<p>In this section, we'll get the project initialized, install everything we need, and configure our environment variables. By the end of this section you should have a working foundation to start building on.</p>
<h3>
  
  
  Prerequisites
</h3>

<p>Before getting started, make sure you have the following:</p>

<ul>
<li>Node.js v18 or higher</li>
<li>A <a href="https://console.groq.com" rel="noopener noreferrer">Groq</a> account for the LLM API key</li>
<li>A <a href="https://app.mem0.ai" rel="noopener noreferrer">Mem0</a> account  for the memory layer API key</li>
</ul>
<h3>
  
  
  Installing dependencies
</h3>

<p>First, create your project folder and initialize it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>mem0-express-api <span class="o">&amp;&amp;</span> <span class="nb">cd </span>mem0-express-api
npm init <span class="nt">-y</span>
</code></pre>

</div>



<p>Then install the runtime dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>express mem0ai openai dotenv
</code></pre>

</div>



<p>And the dev dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-D</span> typescript ts-node nodemon @types/express @types/node
</code></pre>

</div>



<p>Finally, initialize TypeScript:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx tsc <span class="nt">--init</span>
</code></pre>

</div>



<p>A quick note on the <code>openai</code> package: even though we're using Groq as our LLM provider, Groq's API is OpenAI-compatible, so the <code>openai</code> SDK works against it directly  you just point it at Groq's base URL.</p>

<h3>
  
  
  Add your scripts
</h3>

<p>Update your <code>package.json</code> scripts section to look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="nl">"dev"</span><span class="p">:</span><span class="w"> </span><span class="s2">"nodemon --exec ts-node src/index.ts"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"build"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tsc"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node dist/index.js"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Environment variables
</h3>

<p>Create a <code>.env</code> file at the root of your project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>GROQ_API_KEY=your_groq_api_key
MEM0_API_KEY=your_mem0_api_key
PORT=3000
</code></pre>

</div>



<p>You can grab your Groq API key from the <a href="https://console.groq.com" rel="noopener noreferrer">Groq console</a> and your Mem0 API key from the <a href="https://app.mem0.ai" rel="noopener noreferrer">Mem0 dashboard</a>. Never commit this file  add it to your <code>.gitignore</code>.</p>

<h2>
  
  
  Building the API
</h2>

<p>With the project set up, let's walk through the actual code. Here's how everything is structured:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mem0-express-api/
├── src/
│   ├── lib/
│   │   └── mem0.ts
│   ├── routes/
│   │   └── chat.ts
│   └── index.ts
├── .env
├── .gitignore
├── package.json
└── tsconfig.json
</code></pre>

</div>



<p>Go ahead and create the folders and files to match this structure before we start filling them in.</p>

<h3>
  
  
  Setting up the entry point
</h3>

<p><code>src/index.ts</code> is where the Express app is initialized. It loads environment variables, registers the chat router under the <code>/chat</code> path, and starts the server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="dl">"</span><span class="s2">dotenv/config</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">chatRouter</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./routes/chat</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">||</span> <span class="mi">3000</span><span class="p">;</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">json</span><span class="p">());</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="dl">"</span><span class="s2">/chat</span><span class="dl">"</span><span class="p">,</span> <span class="nx">chatRouter</span><span class="p">);</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Server running on http://localhost:</span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Initializing the Mem0 client
</h3>

<p><code>src/lib/mem0.ts</code> initializes the Mem0 client once and exports it for use across the app:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">MemoryClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">mem0ai</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">MemoryClient</span><span class="p">({</span>
  <span class="na">apiKey</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">MEM0_API_KEY</span><span class="o">!</span><span class="p">,</span>
<span class="p">});</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">client</span><span class="p">;</span>
</code></pre>

</div>



<p>Keeping this in its own file means you initialize the client once and import it wherever you need it, rather than recreating it on every request.</p>

<h3>
  
  
  The chat routes
</h3>

<p>All the route logic lives in <code>src/routes/chat.ts</code>. Here's the full file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Router</span><span class="p">,</span> <span class="nx">Request</span><span class="p">,</span> <span class="nx">Response</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">OpenAI</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">openai</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">mem0</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">../lib/mem0</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nc">Router</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">openai</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">OpenAI</span><span class="p">({</span>
  <span class="na">apiKey</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">GROQ_API_KEY</span><span class="o">!</span><span class="p">,</span>
  <span class="na">baseURL</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.groq.com/openai/v1</span><span class="dl">"</span><span class="p">,</span>
<span class="p">});</span>

<span class="c1">// POST /chat/:userId</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">"</span><span class="s2">/:userId</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">,</span> <span class="nx">res</span><span class="p">:</span> <span class="nx">Response</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">userId</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">message</span><span class="p">,</span> <span class="nx">sessionId</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">message</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Message is required</span><span class="dl">"</span> <span class="p">});</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">memories</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">mem0</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span><span class="nx">message</span><span class="p">,</span> <span class="p">{</span> <span class="na">user_id</span><span class="p">:</span> <span class="nx">userId</span> <span class="p">}</span> <span class="k">as</span> <span class="kr">any</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">memoryContext</span> <span class="o">=</span> <span class="nx">memories</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="na">m</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">m</span><span class="p">.</span><span class="nx">memory</span><span class="p">).</span><span class="nf">join</span><span class="p">(</span><span class="dl">"</span><span class="se">\n</span><span class="dl">"</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">systemPrompt</span> <span class="o">=</span> <span class="s2">`You are a helpful assistant with memory of past interactions.</span><span class="p">${</span>
      <span class="nx">memoryContext</span> <span class="p">?</span> <span class="s2">`\nWhat you remember about this user:\n</span><span class="p">${</span><span class="nx">memoryContext</span><span class="p">}</span><span class="s2">`</span> <span class="p">:</span> <span class="dl">""</span>
    <span class="p">}</span><span class="s2">`</span><span class="p">;</span>

    <span class="kd">const</span> <span class="nx">completion</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">openai</span><span class="p">.</span><span class="nx">chat</span><span class="p">.</span><span class="nx">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
      <span class="na">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">llama-3.3-70b-versatile</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">messages</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">system</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">systemPrompt</span> <span class="p">},</span>
        <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">message</span> <span class="p">},</span>
      <span class="p">],</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">reply</span> <span class="o">=</span> <span class="nx">completion</span><span class="p">.</span><span class="nx">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">message</span><span class="p">.</span><span class="nx">content</span><span class="p">;</span>

    <span class="kd">const</span> <span class="na">addOptions</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span> <span class="na">user_id</span><span class="p">:</span> <span class="nx">userId</span> <span class="p">};</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">sessionId</span><span class="p">)</span> <span class="nx">addOptions</span><span class="p">.</span><span class="nx">run_id</span> <span class="o">=</span> <span class="nx">sessionId</span><span class="p">;</span>

    <span class="k">await</span> <span class="nx">mem0</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span>
      <span class="p">[</span>
        <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">message</span> <span class="p">},</span>
        <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">assistant</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">reply</span><span class="o">!</span> <span class="p">},</span>
      <span class="p">],</span>
      <span class="nx">addOptions</span>
    <span class="p">);</span>

    <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="nx">reply</span><span class="p">,</span> <span class="nx">userId</span><span class="p">,</span> <span class="na">sessionId</span><span class="p">:</span> <span class="nx">sessionId</span> <span class="o">||</span> <span class="kc">null</span> <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="na">error</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="c1">// GET /chat/:userId/memories</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">/:userId/memories</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">,</span> <span class="nx">res</span><span class="p">:</span> <span class="nx">Response</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">userId</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">memories</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">mem0</span><span class="p">.</span><span class="nf">getAll</span><span class="p">({</span> <span class="na">user_id</span><span class="p">:</span> <span class="nx">userId</span> <span class="p">}</span> <span class="k">as</span> <span class="kr">any</span><span class="p">);</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="nx">userId</span><span class="p">,</span> <span class="nx">memories</span> <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="na">error</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="c1">// DELETE /chat/:userId/memories</span>
<span class="nx">router</span><span class="p">.</span><span class="k">delete</span><span class="p">(</span><span class="dl">"</span><span class="s2">/:userId/memories</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">,</span> <span class="nx">res</span><span class="p">:</span> <span class="nx">Response</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">userId</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">mem0</span><span class="p">.</span><span class="nf">deleteAll</span><span class="p">({</span> <span class="na">user_id</span><span class="p">:</span> <span class="nx">userId</span> <span class="p">}</span> <span class="k">as</span> <span class="kr">any</span><span class="p">);</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">message</span><span class="p">:</span> <span class="s2">`All memories cleared for user </span><span class="p">${</span><span class="nx">userId</span><span class="p">}</span><span class="s2">`</span> <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="na">error</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="c1">// POST /chat/:userId/no-memory</span>
<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">"</span><span class="s2">/:userId/no-memory</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">Request</span><span class="p">,</span> <span class="nx">res</span><span class="p">:</span> <span class="nx">Response</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">userId</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">message</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">message</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Message is required</span><span class="dl">"</span> <span class="p">});</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">completion</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">openai</span><span class="p">.</span><span class="nx">chat</span><span class="p">.</span><span class="nx">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
      <span class="na">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">llama-3.3-70b-versatile</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">messages</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">system</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="dl">"</span><span class="s2">You are a helpful assistant.</span><span class="dl">"</span> <span class="p">},</span>
        <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">message</span> <span class="p">},</span>
      <span class="p">],</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">reply</span> <span class="o">=</span> <span class="nx">completion</span><span class="p">.</span><span class="nx">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">message</span><span class="p">.</span><span class="nx">content</span><span class="p">;</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="nx">reply</span><span class="p">,</span> <span class="nx">userId</span><span class="p">,</span> <span class="na">memoryUsed</span><span class="p">:</span> <span class="kc">false</span> <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="na">error</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">router</span><span class="p">;</span>
</code></pre>

</div>



<p>Now let's break down what each route does.</p>

<p><strong>POST <code>/chat/:userId</code></strong> is the core route. When a message comes in, it first searches Mem0 for any memories relevant to that message and injects them into the system prompt. The LLM then responds with that context already in place. After the response is generated, the full exchange is passed to <code>mem0.add()</code> which processes it asynchronously and extracts facts worth remembering. The optional <code>sessionId</code> in the request body scopes the memory to a specific conversation thread via <code>run_id</code> useful if one user has multiple distinct conversation contexts.</p>

<p><strong>GET <code>/chat/:userId/memories</code></strong> returns everything Mem0 has stored for a given user. We'll use this in Section 6 to inspect what was actually extracted after a conversation.</p>

<p><strong>DELETE <code>/chat/:userId/memories</code></strong> wipes all stored memories for a user. Useful for resetting state during testing or giving users a clean slate.</p>

<p><strong>POST <code>/chat/:userId/no-memory</code></strong> is a control route that calls the same LLM with the same model but with no memory context at all just a plain system prompt. We'll use this in  the next section to show the difference between a memory-aware response and a stateless one side by side.</p>

<h2>
  
  
  How Mem0 Stores Memories
</h2>

<p>When you call <code>mem0.add()</code> after an exchange, Mem0 doesn't just save the raw messages. It processes the conversation, pulls out the facts that matter, and stores them as structured memory objects. Here's what that actually looks like.</p>

<p>After a few conversations as <code>user123</code>, calling <code>GET /chat/user123/memories</code> returns something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"userId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"memories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"a9a14f57-e0ae-4533-82a1-c5e93364fb7b"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"memory"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Ayomide loves table tennis and enjoys trying different foods"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"user_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"categories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"sports"</span><span class="p">,</span><span class="w"> </span><span class="s2">"food"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"created_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-19T05:38:37-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"updated_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-19T05:38:41.614883-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"structured_attributes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"day"</span><span class="p">:</span><span class="w"> </span><span class="mi">19</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hour"</span><span class="p">:</span><span class="w"> </span><span class="mi">12</span><span class="p">,</span><span class="w">
        </span><span class="nl">"year"</span><span class="p">:</span><span class="w"> </span><span class="mi">2026</span><span class="p">,</span><span class="w">
        </span><span class="nl">"month"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
        </span><span class="nl">"day_of_week"</span><span class="p">:</span><span class="w"> </span><span class="s2">"thursday"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"f0c175d5-1b9a-462a-bf17-88d287271df0"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"memory"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Ayomide is building an API platform called APIblok"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"user_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"categories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"professional_details"</span><span class="p">,</span><span class="w"> </span><span class="s2">"technology"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"created_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-18T11:19:19-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"updated_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-18T11:19:17.557120-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"structured_attributes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"day"</span><span class="p">:</span><span class="w"> </span><span class="mi">18</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hour"</span><span class="p">:</span><span class="w"> </span><span class="mi">18</span><span class="p">,</span><span class="w">
        </span><span class="nl">"year"</span><span class="p">:</span><span class="w"> </span><span class="mi">2026</span><span class="p">,</span><span class="w">
        </span><span class="nl">"month"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
        </span><span class="nl">"day_of_week"</span><span class="p">:</span><span class="w"> </span><span class="s2">"wednesday"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ea262dae-72af-48d9-a8e8-4b18cdefa9a5"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"memory"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Ayomide has 3 years of experience with Node.js and prefers using TypeScript"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"user_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"categories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"professional_details"</span><span class="p">,</span><span class="w"> </span><span class="s2">"technology"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"created_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-18T10:55:20-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"updated_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-18T10:56:45.972173-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"structured_attributes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"day"</span><span class="p">:</span><span class="w"> </span><span class="mi">18</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hour"</span><span class="p">:</span><span class="w"> </span><span class="mi">17</span><span class="p">,</span><span class="w">
        </span><span class="nl">"year"</span><span class="p">:</span><span class="w"> </span><span class="mi">2026</span><span class="p">,</span><span class="w">
        </span><span class="nl">"month"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
        </span><span class="nl">"day_of_week"</span><span class="p">:</span><span class="w"> </span><span class="s2">"wednesday"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>A few things worth noticing here.</p>

<p><strong>The <code>memory</code> field is a clean, distilled fact</strong> not a raw message. Mem0 didn't store "hey I love table tennis and I also like trying new foods", it stored "Ayomide loves table tennis and enjoys trying different foods". The extraction is deliberate and concise.</p>

<p><strong>Categories are automatically assigned.</strong> Each memory object comes with a <code>categories</code> array <code>["sports", "food"]</code>, <code>["professional_details", "technology"]</code> and so on. Mem0 figures these out on its own without any configuration from you. This is what makes retrieval intelligent — when a new message comes in, Mem0 knows which memories are actually relevant to surface.</p>

<p><strong>Deduplication happens automatically.</strong> If the user mentions the same detail across multiple conversations, Mem0 won't create a new memory entry every time it updates the existing one instead. You can see this in the <code>updated_at</code> field, which can differ from <code>created_at</code> when a memory has been refined over time.</p>

<p><strong><code>structured_attributes</code> adds temporal context</strong> — recording when the memory was created down to the hour and day of week. This gives Mem0 the ability to reason about recency when deciding what to retrieve.</p>

<p>You can also view and manage all of this visually from the Mem0 dashboard:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxci1kmloaxh42keanrrl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxci1kmloaxh42keanrrl.png" alt="Mem0 dashabord" width="800" height="395"></a></p>

<p>The dashboard gives you a real-time view of what's been extracted for each user, which is particularly useful when you're debugging or want to verify that the right facts are being stored.</p>

<h2>
  
  
  Testing the API
</h2>

<p>With the project set up, you'll need <strong>two terminal windows open simultaneously</strong> for this section. In the first terminal, start the dev server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm run dev
</code></pre>

</div>



<p>You should see <code>Server running on http://localhost:3000</code>. <strong>Leave this terminal running</strong> — the server needs to stay alive to handle requests. Open a second terminal window to send the curl commands. If you're on Windows, use <strong>Command Prompt (cmd)</strong> for both terminals rather than PowerShell. PowerShell maps <code>curl</code> to its own <code>Invoke-WebRequest</code> command, which won't work with these examples. If you prefer Git Bash, that works too — it ships with Git for Windows and handles curl correctly.</p>

<blockquote>
<p><strong>Windows cmd users:</strong> The multiline curl commands below use <code>\</code> for line continuation, which is a macOS/Linux convention. Windows cmd doesn't support it if you run them as-is, you'll get <code>curl: (3) URL rejected: Bad hostname</code> errors after each line. The request itself will still go through, but to avoid the noise, collapse each command onto a single line. For example:<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  curl <span class="nt">-X</span> POST http://localhost:3000/chat/user123 <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="nt">-d</span> <span class="s2">"{</span><span class="se">\"</span><span class="s2">message</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">Your message here</span><span class="se">\"</span><span class="s2">}"</span>
</code></pre>

</div>



<h3>
  
  
  Sending your first message
</h3>

<p>Start with a message that gives the model something worth remembering — some personal or technical context:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/chat/user123 <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s2">"{</span><span class="se">\"</span><span class="s2">message</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">Hey, I'm Ayomide. I have 3 years of experience with Node.js and I prefer TypeScript over plain JavaScript.</span><span class="se">\"</span><span class="s2">}"</span>
</code></pre>

</div>



<p>You should get back something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"reply"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Hi Ayomide! That's a solid background — 3 years with Node.js and a preference for TypeScript is a great combo. TypeScript's type safety really pays off in larger codebases. What are you working on?"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"userId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"sessionId"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The response itself is unremarkable the model is working with just what you sent it. What matters is what's happening in the background: Mem0 is processing this exchange and extracting facts from it asynchronously.<br>
Wait a few seconds, then send another message to give Mem0 time to finish processing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/chat/user123 <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s2">"{</span><span class="se">\"</span><span class="s2">message</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">I'm currently building an API platform called APIblok.</span><span class="se">\"</span><span class="s2">}"</span>
</code></pre>

</div>



<p>Now check what Mem0 has stored:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:3000/chat/user123/memories
</code></pre>

</div>



<p>You should see something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"userId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"memories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ec30b7df-38f5-411d-af13-4d42befe4a3e"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"memory"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Ayomide is currently building an API platform called APIblok."</span><span class="p">,</span><span class="w">
      </span><span class="nl">"user_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"categories"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"created_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-19T14:08:07-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"updated_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-19T14:08:07-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"structured_attributes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"day"</span><span class="p">:</span><span class="w"> </span><span class="mi">19</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hour"</span><span class="p">:</span><span class="w"> </span><span class="mi">21</span><span class="p">,</span><span class="w">
        </span><span class="nl">"year"</span><span class="p">:</span><span class="w"> </span><span class="mi">2026</span><span class="p">,</span><span class="w">
        </span><span class="nl">"month"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
        </span><span class="nl">"day_of_week"</span><span class="p">:</span><span class="w"> </span><span class="s2">"thursday"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"b0217d62-f54d-4b7e-bca1-e6cf528ee09a"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"memory"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Ayomide has 3 years of experience with Node.js and prefers TypeScript over plain JavaScript."</span><span class="p">,</span><span class="w">
      </span><span class="nl">"user_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"categories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"professional_details"</span><span class="p">,</span><span class="w"> </span><span class="s2">"technology"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"created_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-19T14:07:18-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"updated_at"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-19T14:08:26.222857-07:00"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"structured_attributes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"day"</span><span class="p">:</span><span class="w"> </span><span class="mi">19</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hour"</span><span class="p">:</span><span class="w"> </span><span class="mi">21</span><span class="p">,</span><span class="w">
        </span><span class="nl">"year"</span><span class="p">:</span><span class="w"> </span><span class="mi">2026</span><span class="p">,</span><span class="w">
        </span><span class="nl">"month"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
        </span><span class="nl">"day_of_week"</span><span class="p">:</span><span class="w"> </span><span class="s2">"thursday"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Mem0 didn't store your raw messages. It extracted the facts that actually matter your experience level, your tech preferences, what you're building and stored them as clean, discrete memory objects. That's the extraction step working exactly as intended.</p>

<h3>
  
  
  The before/after comparison
</h3>

<p>To make the difference concrete, the API has a <code>/no-memory</code> route that hits the same LLM with the same model but with zero memory context. Let's use it.</p>

<p>First, send a follow-up question through the memory-aware route:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/chat/user123 <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s2">"{</span><span class="se">\"</span><span class="s2">message</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">What framework would you recommend for my project?</span><span class="se">\"</span><span class="s2">}"</span>
</code></pre>

</div>



<p>Sample Response:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"reply"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Given that you're building APIblok an API platform  and you're comfortable with TypeScript, I'd recommend sticking with Express if you want something minimal and flexible, or looking at Fastify if performance is a priority. Both have strong TypeScript support. Hono is also worth a look if you're targeting edge runtimes."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"userId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The response is specific. It knows what you're building, knows you're a TypeScript person, and gives a recommendation that fits your actual context.</p>

<p>Now send the exact same message through the stateless route:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/chat/user123/no-memory <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s2">"{</span><span class="se">\"</span><span class="s2">message</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">What framework would you recommend for my project?</span><span class="se">\"</span><span class="s2">}"</span>
</code></pre>

</div>



<p>Sample Response:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"reply"</span><span class="p">:</span><span class="w"> </span><span class="s2">"To provide a framework recommendation that fits your needs, I'd love to know more about your project. Here are a few questions to get started:</span><span class="se">\n\n</span><span class="s2">1. What type of project is it?</span><span class="se">\n</span><span class="s2">2. What programming languages are you planning to use?</span><span class="se">\n</span><span class="s2">3. What are the main features and functionalities?</span><span class="se">\n</span><span class="s2">4. Do you have any specific requirements or constraints?</span><span class="se">\n</span><span class="s2">5. Are there any particular technologies you're interested in integrating with?"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"userId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"memoryUsed"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Same model, same question  and instead of an answer, you get an interrogation. The stateless route has no idea who you are, what you're building, or what language you use, so it fires back five clarifying questions asking you to repeat everything you've already told it. The memory-aware route answered directly because it already had the context it needed.<br>
This is the ceiling that plain conversation history hits across sessions. Memory context closes that gap without any extra work from the user.</p>
<h3>
  
  
  What the memory objects look like
</h3>

<p>If you've been following along, the memory objects from <code>GET /chat/:userId/memories</code> should look familiar — we walked through the full structure in the previous section. But now that you've seen them generated live from real conversations, the structure makes more sense in context.<br>
A few things worth reinforcing:<br>
<strong>The <code>memory</code> field is a distilled fact, not a raw message.</strong> Mem0 didn't store "hey I have 3 years with Node and I prefer TypeScript" it stored "Ayomide has 3 years of experience with Node.js and prefers using TypeScript". The extraction is deliberate and concise.</p>

<p><strong>Categories are assigned automatically.</strong> Mem0 decided <code>["professional_details", "technology"]</code> without any configuration from you. This is what makes retrieval intelligent when a new message comes in, Mem0 knows which memories are actually relevant to surface rather than dumping everything into the context.<br>
<strong>Deduplication happens in the background.</strong> Try sending the same kind of detail twice across two separate requests:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/chat/user123 <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s2">"{</span><span class="se">\"</span><span class="s2">message</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">Just a reminder — I work primarily in TypeScript.</span><span class="se">\"</span><span class="s2">}"</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"reply"</span><span class="p">:</span><span class="w"> </span><span class="s2">"I recall that you prefer working with TypeScript over plain JavaScript, and that you have around 3 years of experience with Node.js. You're also currently building an API platform called APIblok. How can I assist you with your TypeScript or APIblok project today?"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"userId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"sessionId"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The model already knows. Now check the memories endpoint you won't find a new entry for TypeScript. Mem0 recognized it as information it already had and updated the <code>updated_at</code> timestamp on the existing memory rather than creating a redundant one. The store stays clean automatically, with no deduplication logic on your end.<br>
To reset your state between test runs, hit the DELETE endpoint:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> DELETE http://localhost:3000/chat/user123/memories
</code></pre>

</div>



<p>This wipes all stored memories for <code>user123</code> and gives you a clean slate to test from scratch.</p>

<h2>
  
  
  Conclusion
</h2>

<p>Most AI applications treat memory as an afterthought  they replay conversation history on every request and call it context. That works until it doesn't: context windows fill up, costs climb, and the moment a user starts a new session, everything they've told the system is gone. The problem isn't the LLM. It's the architecture around it.</p>

<p>What we built here takes a different approach. By adding Mem0 as a dedicated memory layer, the API extracts what matters from each conversation, stores it as structured facts, and retrieves only what's relevant on the next request. The result is a system that gets more useful over time — not one that forgets everything the moment a session ends.</p>

<p>To recap what we covered: setting up an Express and TypeScript API with Groq as the LLM provider, initializing Mem0 and wiring it into the request lifecycle, building routes for memory-aware chat, memory retrieval, memory deletion, and a stateless control route for comparison, and testing the whole thing end to end to see extraction, retrieval, and deduplication working in practice.</p>

<p>From here, a few natural directions to explore: scope memories to specific conversation threads using <code>run_id</code>, swap the Mem0 cloud client for a self-hosted instance if you need full data control, or wire this API into a frontend to build a chat interface that actually remembers its users.<br>
The full source code for this tutorial is available on GitHub: [<a href="https://github.com/techsplot/mem0-express-api" rel="noopener noreferrer">https://github.com/techsplot/mem0-express-api</a>]</p>

