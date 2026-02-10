---
Title: Build Your First AI Agent with RapidKit in 10 Minutes
Description: 
Author: RapidKit
Date: 2026-02-10T21:37:22.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  The Problem with AI Integration
</h3>

<p>You want to add AI to your FastAPI backend.</p>

<p>What usually happens:</p>

<ul>
<li>Install 5 different packages (openai, anthropic, langchain, etc.)</li>
<li>Write wrapper code for each provider</li>
<li>Handle streaming, caching, retries manually</li>
<li>Debug configuration hell</li>
<li>Spend 2 days integrating what should take 10 minutes</li>
</ul>

<p><strong>There's a better way.</strong></p>




<h2>
  
  
  What We're Building
</h2>

<p>A production-ready <strong>AI Assistant Service</strong> with:</p>

<ul>
<li>✅ Multiple AI providers (Echo, Template, OpenAI-ready)</li>
<li>✅ Chat completions API</li>
<li>✅ Streaming responses</li>
<li>✅ Conversation history</li>
<li>✅ Response caching</li>
<li>✅ Health monitoring</li>
<li>✅ Provider switching</li>
</ul>

<p><strong>Time to build:</strong> 10 minutes<br><br>
<strong>Lines of custom code:</strong> ~50</p>

<p>Let's start.</p>


<h2>
  
  
  Prerequisites
</h2>

<p>You need:</p>

<ul>
<li>Node.js 20+ (<code>node --version</code>)</li>
<li>Python 3.10+ (<code>python3 --version</code>)</li>
<li>A terminal</li>
<li>(Optional) OpenAI API key for real AI</li>
</ul>

<p><strong>Time:</strong> 1 minute to verify</p>


<h2>
  
  
  Minute 0-2: Create Workspace and Project
</h2>

<p><strong>Step 1:</strong> Create workspace (handles Python/Poetry setup automatically)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx rapidkit my-ai-workspace
</code></pre>

</div>



<p><strong>Prompts you'll see:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>? Author name: developer
? Select Python version: 3.10
? Install method: 🎯 Poetry (Recommended)
</code></pre>

</div>



<p><strong>What happens:</strong></p>

<ul>
<li>Creates workspace directory</li>
<li>Installs Poetry if needed</li>
<li>Creates virtualenv in workspace root</li>
<li>Installs rapidkit-core</li>
</ul>

<p><strong>Step 2:</strong> Create FastAPI project<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>my-ai-workspace
rapidkit create project fastapi.standard ai-agent
</code></pre>

</div>



<p><strong>Prompts:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Project name: ai-agent
Install essential modules? Y
</code></pre>

</div>



<p><strong>Step 3:</strong> Initialize project<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>ai-agent
rapidkit init
</code></pre>

</div>



<p><strong>Your structure:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>my-ai-workspace/
├── .venv/                  # Workspace virtualenv
├── pyproject.toml
└── ai-agent/
    ├── .venv/             # Project virtualenv
    ├── src/
    │   ├── main.py
    │   └── modules/
    ├── tests/
    └── pyproject.toml
</code></pre>

</div>






<h2>
  
  
  Minute 2-3: Add AI Assistant Module
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>rapidkit add module ai_assistant
</code></pre>

</div>



<p><strong>What gets installed:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>src/
├── modules/
│   └── free/
│       └── ai/
│           └── ai_assistant/
│               ├── ai_assistant.py          # Runtime engine
│               ├── ai_assistant_types.py    # Type definitions
│               └── routers/
│                   └── ai/
│                       └── ai_assistant.py  # FastAPI routes
└── health/
    └── ai_assistant.py                      # Health endpoint

tests/
└── modules/
    └── free/
        └── integration/
            └── ai/
                └── ai_assistant/
                    └── test_ai_assistant_integration.py
</code></pre>

</div>



<p><strong>Key Components:</strong></p>

<ol>
<li>
<p><strong>Runtime (<code>ai_assistant.py</code>):</strong></p>

<ul>
<li>Provider management</li>
<li>Chat completion logic</li>
<li>Streaming support</li>
<li>Conversation history</li>
<li>Response caching</li>
</ul>
</li>
<li>
<p><strong>Routes (<code>routers/ai/ai_assistant.py</code>):</strong></p>

<ul>
<li>
<code>POST /ai/assistant/completions</code> — Get completions</li>
<li>
<code>POST /ai/assistant/stream</code> — Stream responses</li>
<li>
<code>GET /ai/assistant/providers</code> — List providers</li>
<li>
<code>POST /ai/assistant/cache/clear</code> — Clear cache</li>
<li>
<code>GET /ai/assistant/health</code> — Health check</li>
</ul>
</li>
<li>
<p><strong>Types (<code>ai_assistant_types.py</code>):</strong></p>

<ul>
<li>
<code>AssistantMessage</code> — Message structure</li>
<li>
<code>AssistantResponse</code> — Response structure</li>
<li>
<code>ProviderConfig</code> — Provider configuration</li>
<li>
<code>AiAssistantConfig</code> — Runtime config</li>
</ul>
</li>
</ol>




<h2>
  
  
  Minute 3-4: Configure Providers
</h2>

<p>Create <code>config/ai_assistant.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># AI Assistant Configuration</span>
<span class="na">providers</span><span class="pi">:</span>
  <span class="c1"># Echo Provider (for testing)</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">echo</span>
    <span class="na">provider_type</span><span class="pi">:</span> <span class="s">echo</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
    <span class="na">options</span><span class="pi">:</span>
      <span class="na">prefix</span><span class="pi">:</span> <span class="s2">"</span><span class="s">[AI]</span><span class="nv"> </span><span class="s">"</span>
      <span class="na">suffix</span><span class="pi">:</span> <span class="s2">"</span><span class="nv"> </span><span class="s">(powered</span><span class="nv"> </span><span class="s">by</span><span class="nv"> </span><span class="s">Echo)"</span>
      <span class="na">mirror_context</span><span class="pi">:</span> <span class="kc">true</span>

  <span class="c1"># Template Provider (pre-defined responses)</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">support</span>
    <span class="na">provider_type</span><span class="pi">:</span> <span class="s">template</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
    <span class="na">options</span><span class="pi">:</span>
      <span class="na">responses</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s2">"</span><span class="s">Thank</span><span class="nv"> </span><span class="s">you</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">contacting</span><span class="nv"> </span><span class="s">support.</span><span class="nv"> </span><span class="s">{prompt}"</span>
        <span class="pi">-</span> <span class="s2">"</span><span class="s">I</span><span class="nv"> </span><span class="s">understand</span><span class="nv"> </span><span class="s">your</span><span class="nv"> </span><span class="s">concern</span><span class="nv"> </span><span class="s">about</span><span class="nv"> </span><span class="s">{prompt}.</span><span class="nv"> </span><span class="s">Let</span><span class="nv"> </span><span class="s">me</span><span class="nv"> </span><span class="s">help."</span>
        <span class="pi">-</span> <span class="s2">"</span><span class="s">Based</span><span class="nv"> </span><span class="s">on</span><span class="nv"> </span><span class="s">'{context}',</span><span class="nv"> </span><span class="s">regarding</span><span class="nv"> </span><span class="s">{prompt},</span><span class="nv"> </span><span class="s">here's</span><span class="nv"> </span><span class="s">what</span><span class="nv"> </span><span class="s">I</span><span class="nv"> </span><span class="s">suggest..."</span>

  <span class="c1"># OpenAI Provider (ready for real AI)</span>
  <span class="c1"># - name: openai</span>
  <span class="c1">#   provider_type: openai</span>
  <span class="c1">#   enabled: false</span>
  <span class="c1">#   options:</span>
  <span class="c1">#     api_key: ${OPENAI_API_KEY}</span>
  <span class="c1">#     model: gpt-4</span>
  <span class="c1">#     temperature: 0.7</span>

<span class="na">default_provider</span><span class="pi">:</span> <span class="s">echo</span>
<span class="na">conversation_window</span><span class="pi">:</span> <span class="m">20</span>
<span class="na">cache_enabled</span><span class="pi">:</span> <span class="kc">true</span>
<span class="na">request_timeout_seconds</span><span class="pi">:</span> <span class="m">30</span>
</code></pre>

</div>



<p><strong>Provider Types Explained:</strong></p>

<h3>
  
  
  Echo Provider
</h3>

<p>Perfect for testing. Echoes back your prompt with optional prefix/suffix.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Example response:
# Input: "Hello"
# Output: "[AI] Hello (powered by Echo)"
</span></code></pre>

</div>



<h3>
  
  
  Template Provider
</h3>

<p>Returns pre-configured responses in rotation. Great for FAQ bots.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># First request: "How do I reset password?"
# Response: "Thank you for contacting support. How do I reset password?"
# 
# Second request: "Where is my order?"
# Response: "I understand your concern about Where is my order?. Let me help."
</span></code></pre>

</div>



<h3>
  
  
  OpenAI Provider (Real AI)
</h3>

<p>Integrates with OpenAI API. You'll implement this in advanced section.</p>




<h2>
  
  
  Minute 4-5: Integrate into FastAPI
</h2>

<p>Update <code>src/main.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">fastapi</span> <span class="kn">import</span> <span class="n">FastAPI</span>
<span class="kn">from</span> <span class="n">src.modules.free.ai.ai_assistant.routers.ai.ai_assistant</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">register_ai_assistant</span><span class="p">,</span>
    <span class="n">build_config</span><span class="p">,</span>
<span class="p">)</span>

<span class="c1"># Load config from YAML
</span><span class="kn">import</span> <span class="n">yaml</span>
<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">config/ai_assistant.yaml</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="n">config_data</span> <span class="o">=</span> <span class="n">yaml</span><span class="p">.</span><span class="nf">safe_load</span><span class="p">(</span><span class="n">f</span><span class="p">)</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">FastAPI</span><span class="p">(</span>
    <span class="n">title</span><span class="o">=</span><span class="sh">"</span><span class="s">AI Agent API</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">version</span><span class="o">=</span><span class="sh">"</span><span class="s">1.0.0</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Production AI Assistant powered by RapidKit</span><span class="sh">"</span>
<span class="p">)</span>

<span class="c1"># Register AI Assistant
</span><span class="nf">register_ai_assistant</span><span class="p">(</span><span class="n">app</span><span class="p">,</span> <span class="n">config</span><span class="o">=</span><span class="n">config_data</span><span class="p">)</span>

<span class="nd">@app.get</span><span class="p">(</span><span class="sh">"</span><span class="s">/</span><span class="sh">"</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">root</span><span class="p">():</span>
    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">AI Agent API is running</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">endpoints</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">completions</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">/ai/assistant/completions</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">stream</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">/ai/assistant/stream</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">providers</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">/ai/assistant/providers</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">health</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">/ai/assistant/health</span><span class="sh">"</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre>

</div>



<p><strong>Install PyYAML:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>poetry add pyyaml
</code></pre>

</div>






<h2>
  
  
  Minute 5-6: Test Basic Functionality
</h2>

<p><strong>Start the server:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>rapidkit dev
</code></pre>

</div>



<p><strong>Server output:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>INFO:     Uvicorn running on http://127.0.0.1:8000
INFO:     Application startup complete.
</code></pre>

</div>



<p><strong>Open browser:</strong> <a href="http://localhost:8000/docs" rel="noopener noreferrer">http://localhost:8000/docs</a></p>

<p>You'll see Swagger UI with AI endpoints!</p>




<h2>
  
  
  Minute 6-7: Test Echo Provider
</h2>

<p><strong>List available providers:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:8000/ai/assistant/providers
</code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">[</span><span class="s2">"echo"</span><span class="p">,</span><span class="w"> </span><span class="s2">"support"</span><span class="p">]</span><span class="w">
</span></code></pre>

</div>



<p><strong>Test echo completion:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "prompt": "What is RapidKit?",
    "provider": "echo"
  }'</span>
</code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"provider"</span><span class="p">:</span><span class="w"> </span><span class="s2">"echo"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"content"</span><span class="p">:</span><span class="w"> </span><span class="s2">"[AI] What is RapidKit? (powered by Echo)"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.234</span><span class="p">,</span><span class="w">
  </span><span class="nl">"cached"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"usage"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"prompt_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
    </span><span class="nl">"completion_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">8</span><span class="p">,</span><span class="w">
    </span><span class="nl">"total_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">11</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"metadata"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  Minute 7-8: Test Template Provider
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "prompt": "How do I reset my password?",
    "provider": "support"
  }'</span>
</code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"provider"</span><span class="p">:</span><span class="w"> </span><span class="s2">"support"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"content"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Thank you for contacting support. How do I reset my password?"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.156</span><span class="p">,</span><span class="w">
  </span><span class="nl">"cached"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"usage"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"prompt_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">6</span><span class="p">,</span><span class="w">
    </span><span class="nl">"completion_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">8</span><span class="p">,</span><span class="w">
    </span><span class="nl">"total_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">14</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Test with conversation context:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "prompt": "Follow up question",
    "provider": "support",
    "context": [
      {"role": "user", "content": "I forgot my password"},
      {"role": "assistant", "content": "I can help reset your password"}
    ]
  }'</span>
</code></pre>

</div>



<p><strong>Response includes context:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"provider"</span><span class="p">:</span><span class="w"> </span><span class="s2">"support"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"content"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Based on 'I forgot my password | I can help reset your password', regarding Follow up question, here's what I suggest..."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.189</span><span class="p">,</span><span class="w">
  </span><span class="nl">"cached"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"usage"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="err">...</span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  Minute 8-9: Test Streaming
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/stream <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "prompt": "Tell me about AI agents",
    "provider": "echo"
  }'</span>
</code></pre>

</div>



<p><strong>Response streams word by word:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[AI] Tell me about AI agents (powered by Echo)
</code></pre>

</div>



<p><strong>With Python client:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span>
    <span class="sh">"</span><span class="s">http://localhost:8000/ai/assistant/stream</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">json</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">prompt</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Explain RapidKit</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">provider</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">echo</span><span class="sh">"</span><span class="p">},</span>
    <span class="n">stream</span><span class="o">=</span><span class="bp">True</span>
<span class="p">)</span>

<span class="k">for</span> <span class="n">chunk</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="nf">iter_content</span><span class="p">(</span><span class="n">chunk_size</span><span class="o">=</span><span class="bp">None</span><span class="p">,</span> <span class="n">decode_unicode</span><span class="o">=</span><span class="bp">True</span><span class="p">):</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">chunk</span><span class="p">,</span> <span class="n">end</span><span class="o">=</span><span class="sh">""</span><span class="p">,</span> <span class="n">flush</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
</code></pre>

</div>






<h2>
  
  
  Minute 9-10: Test Caching
</h2>

<p><strong>First request (not cached):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "prompt": "What is caching?",
    "provider": "echo"
  }'</span>
</code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"cached"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.245</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Second identical request (cached):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Same request again</span>
curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "prompt": "What is caching?",
    "provider": "echo"
  }'</span>
</code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"cached"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.245</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Notice:</strong> <code>cached: true</code> and same latency (from cache, not recomputed)</p>

<p><strong>Clear cache:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/cache/clear
</code></pre>

</div>






<h2>
  
  
  Health Monitoring
</h2>

<p>Check AI assistant health:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl http://localhost:8000/ai/assistant/health
</code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"vendor"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"module"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ai_assistant"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ok"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"0.1.7"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"runtime"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"module"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ai_assistant"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ok"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"providers"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"echo"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ok"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.245</span><span class="p">,</span><span class="w">
        </span><span class="nl">"details"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"prefix"</span><span class="p">:</span><span class="w"> </span><span class="s2">"[AI] "</span><span class="p">,</span><span class="w">
          </span><span class="nl">"suffix"</span><span class="p">:</span><span class="w"> </span><span class="s2">" (powered by Echo)"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"mirror_context"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"support"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ok"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.189</span><span class="p">,</span><span class="w">
        </span><span class="nl">"details"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"responses"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">],</span><span class="w">
    </span><span class="nl">"cache_entries"</span><span class="p">:</span><span class="w"> </span><span class="mi">5</span><span class="p">,</span><span class="w">
    </span><span class="nl">"history_length"</span><span class="p">:</span><span class="w"> </span><span class="mi">12</span><span class="p">,</span><span class="w">
    </span><span class="nl">"timestamp"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-10T10:30:00"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ok"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Key metrics:</strong></p>

<ul>
<li>Provider status and latency</li>
<li>Cache hit count</li>
<li>Conversation history length</li>
<li>Overall health status</li>
</ul>




<h2>
  
  
  Advanced: Add OpenAI Provider
</h2>

<p><strong>Step 1:</strong> Install OpenAI SDK<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>poetry add openai
</code></pre>

</div>



<p><strong>Step 2:</strong> Create custom provider</p>

<p>Create <code>src/modules/free/ai/ai_assistant/providers/openai_provider.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="sh">"""</span><span class="s">OpenAI provider implementation.</span><span class="sh">"""</span>

<span class="kn">from</span> <span class="n">__future__</span> <span class="kn">import</span> <span class="n">annotations</span>
<span class="kn">import</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Any</span><span class="p">,</span> <span class="n">Iterable</span><span class="p">,</span> <span class="n">Mapping</span><span class="p">,</span> <span class="n">Sequence</span>

<span class="kn">from</span> <span class="n">openai</span> <span class="kn">import</span> <span class="n">OpenAI</span>
<span class="kn">from</span> <span class="n">src.modules.free.ai.ai_assistant.ai_assistant</span> <span class="kn">import</span> <span class="n">ChatProvider</span>
<span class="kn">from</span> <span class="n">src.modules.free.ai.ai_assistant.ai_assistant_types</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">AssistantMessage</span><span class="p">,</span>
    <span class="n">ProviderStatus</span><span class="p">,</span>
<span class="p">)</span>


<span class="k">class</span> <span class="nc">OpenAIProvider</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">OpenAI chat completion provider.</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span>
        <span class="n">self</span><span class="p">,</span>
        <span class="o">*</span><span class="p">,</span>
        <span class="n">name</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="sh">"</span><span class="s">openai</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">api_key</span><span class="p">:</span> <span class="nb">str</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>
        <span class="n">model</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="sh">"</span><span class="s">gpt-3.5-turbo</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">temperature</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">0.7</span><span class="p">,</span>
    <span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="n">self</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="n">name</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_model</span> <span class="o">=</span> <span class="n">model</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_temperature</span> <span class="o">=</span> <span class="n">temperature</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_client</span> <span class="o">=</span> <span class="nc">OpenAI</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="n">api_key</span> <span class="ow">or</span> <span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">OPENAI_API_KEY</span><span class="sh">"</span><span class="p">))</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_last_latency_ms</span><span class="p">:</span> <span class="nb">float</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span>

    <span class="k">def</span> <span class="nf">generate</span><span class="p">(</span>
        <span class="n">self</span><span class="p">,</span>
        <span class="n">prompt</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
        <span class="o">*</span><span class="p">,</span>
        <span class="n">conversation</span><span class="p">:</span> <span class="n">Sequence</span><span class="p">[</span><span class="n">AssistantMessage</span><span class="p">],</span>
        <span class="n">settings</span><span class="p">:</span> <span class="n">Mapping</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>
    <span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Generate completion using OpenAI API.</span><span class="sh">"""</span>
        <span class="n">messages</span> <span class="o">=</span> <span class="p">[</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="n">msg</span><span class="p">.</span><span class="n">role</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">msg</span><span class="p">.</span><span class="n">content</span><span class="p">}</span>
            <span class="k">for</span> <span class="n">msg</span> <span class="ow">in</span> <span class="n">conversation</span>
        <span class="p">]</span>
        <span class="n">messages</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">prompt</span><span class="p">})</span>

        <span class="n">response</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">_client</span><span class="p">.</span><span class="n">chat</span><span class="p">.</span><span class="n">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
            <span class="n">model</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">model</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">_model</span><span class="p">)</span> <span class="k">if</span> <span class="n">settings</span> <span class="k">else</span> <span class="n">self</span><span class="p">.</span><span class="n">_model</span><span class="p">,</span>
            <span class="n">messages</span><span class="o">=</span><span class="n">messages</span><span class="p">,</span>
            <span class="n">temperature</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">temperature</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">_temperature</span><span class="p">)</span> <span class="k">if</span> <span class="n">settings</span> <span class="k">else</span> <span class="n">self</span><span class="p">.</span><span class="n">_temperature</span><span class="p">,</span>
        <span class="p">)</span>

        <span class="k">return</span> <span class="n">response</span><span class="p">.</span><span class="n">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">message</span><span class="p">.</span><span class="n">content</span>

    <span class="k">def</span> <span class="nf">stream</span><span class="p">(</span>
        <span class="n">self</span><span class="p">,</span>
        <span class="n">prompt</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
        <span class="o">*</span><span class="p">,</span>
        <span class="n">conversation</span><span class="p">:</span> <span class="n">Sequence</span><span class="p">[</span><span class="n">AssistantMessage</span><span class="p">],</span>
        <span class="n">settings</span><span class="p">:</span> <span class="n">Mapping</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>
    <span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Iterable</span><span class="p">[</span><span class="nb">str</span><span class="p">]:</span>
        <span class="sh">"""</span><span class="s">Stream completion tokens from OpenAI.</span><span class="sh">"""</span>
        <span class="n">messages</span> <span class="o">=</span> <span class="p">[</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="n">msg</span><span class="p">.</span><span class="n">role</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">msg</span><span class="p">.</span><span class="n">content</span><span class="p">}</span>
            <span class="k">for</span> <span class="n">msg</span> <span class="ow">in</span> <span class="n">conversation</span>
        <span class="p">]</span>
        <span class="n">messages</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="n">prompt</span><span class="p">})</span>

        <span class="n">stream</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">_client</span><span class="p">.</span><span class="n">chat</span><span class="p">.</span><span class="n">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
            <span class="n">model</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">model</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">_model</span><span class="p">)</span> <span class="k">if</span> <span class="n">settings</span> <span class="k">else</span> <span class="n">self</span><span class="p">.</span><span class="n">_model</span><span class="p">,</span>
            <span class="n">messages</span><span class="o">=</span><span class="n">messages</span><span class="p">,</span>
            <span class="n">temperature</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">temperature</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">_temperature</span><span class="p">)</span> <span class="k">if</span> <span class="n">settings</span> <span class="k">else</span> <span class="n">self</span><span class="p">.</span><span class="n">_temperature</span><span class="p">,</span>
            <span class="n">stream</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
        <span class="p">)</span>

        <span class="k">for</span> <span class="n">chunk</span> <span class="ow">in</span> <span class="n">stream</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">chunk</span><span class="p">.</span><span class="n">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">delta</span><span class="p">.</span><span class="n">content</span><span class="p">:</span>
                <span class="k">yield</span> <span class="n">chunk</span><span class="p">.</span><span class="n">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">delta</span><span class="p">.</span><span class="n">content</span>

    <span class="k">def</span> <span class="nf">note_latency</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">latency_ms</span><span class="p">:</span> <span class="nb">float</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Record latency for health metrics.</span><span class="sh">"""</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_last_latency_ms</span> <span class="o">=</span> <span class="n">latency_ms</span>

    <span class="k">def</span> <span class="nf">health</span><span class="p">(</span><span class="n">self</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">ProviderStatus</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Return provider health status.</span><span class="sh">"""</span>
        <span class="k">return</span> <span class="nc">ProviderStatus</span><span class="p">(</span>
            <span class="n">name</span><span class="o">=</span><span class="n">self</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
            <span class="n">status</span><span class="o">=</span><span class="sh">"</span><span class="s">ok</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">latency_ms</span><span class="o">=</span><span class="n">self</span><span class="p">.</span><span class="n">_last_latency_ms</span><span class="p">,</span>
            <span class="n">details</span><span class="o">=</span><span class="p">{</span>
                <span class="sh">"</span><span class="s">model</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">_model</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">temperature</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">_temperature</span><span class="p">,</span>
            <span class="p">},</span>
        <span class="p">)</span>
</code></pre>

</div>



<p><strong>Step 3:</strong> Register OpenAI provider</p>

<p>Update <code>src/main.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">src.modules.free.ai.ai_assistant.routers.ai.ai_assistant</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">register_ai_assistant</span><span class="p">,</span>
<span class="p">)</span>
<span class="kn">from</span> <span class="n">src.modules.free.ai.ai_assistant.providers.openai_provider</span> <span class="kn">import</span> <span class="n">OpenAIProvider</span>

<span class="kn">import</span> <span class="n">yaml</span>
<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">config/ai_assistant.yaml</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="n">config_data</span> <span class="o">=</span> <span class="n">yaml</span><span class="p">.</span><span class="nf">safe_load</span><span class="p">(</span><span class="n">f</span><span class="p">)</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">FastAPI</span><span class="p">(</span><span class="n">title</span><span class="o">=</span><span class="sh">"</span><span class="s">AI Agent API</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Register assistant
</span><span class="n">assistant</span> <span class="o">=</span> <span class="nf">register_ai_assistant</span><span class="p">(</span><span class="n">app</span><span class="p">,</span> <span class="n">config</span><span class="o">=</span><span class="n">config_data</span><span class="p">)</span>

<span class="c1"># Add OpenAI provider manually
</span><span class="n">openai_provider</span> <span class="o">=</span> <span class="nc">OpenAIProvider</span><span class="p">(</span>
    <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">openai</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">api_key</span><span class="o">=</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">OPENAI_API_KEY</span><span class="sh">"</span><span class="p">),</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gpt-4</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">temperature</span><span class="o">=</span><span class="mf">0.7</span><span class="p">,</span>
<span class="p">)</span>
<span class="n">assistant</span><span class="p">.</span><span class="nf">register_provider</span><span class="p">(</span><span class="n">openai_provider</span><span class="p">)</span>
<span class="n">assistant</span><span class="p">.</span><span class="nf">set_default_provider</span><span class="p">(</span><span class="sh">"</span><span class="s">openai</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Step 4:</strong> Set API key<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">OPENAI_API_KEY</span><span class="o">=</span><span class="s2">"sk-..."</span>
</code></pre>

</div>



<p><strong>Step 5:</strong> Test real AI<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/ai/assistant/completions <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "prompt": "Explain RapidKit in one sentence",
    "provider": "openai"
  }'</span>
</code></pre>

</div>



<p><strong>Response (real GPT-4):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"provider"</span><span class="p">:</span><span class="w"> </span><span class="s2">"openai"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"content"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RapidKit is a developer toolkit that accelerates backend API development by providing pre-built modules, workspace management, and production-ready templates for FastAPI and NestJS frameworks."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">1247.5</span><span class="p">,</span><span class="w">
  </span><span class="nl">"cached"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"usage"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"prompt_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">7</span><span class="p">,</span><span class="w">
    </span><span class="nl">"completion_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">31</span><span class="p">,</span><span class="w">
    </span><span class="nl">"total_tokens"</span><span class="p">:</span><span class="w"> </span><span class="mi">38</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  Real-World Use Case: Customer Support Agent
</h2>

<p>Let's build a customer support AI agent:</p>

<p><strong>Create <code>src/agents/support_agent.py</code>:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="sh">"""</span><span class="s">Customer support AI agent.</span><span class="sh">"""</span>

<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Sequence</span>
<span class="kn">from</span> <span class="n">src.modules.free.ai.ai_assistant.ai_assistant</span> <span class="kn">import</span> <span class="n">AiAssistant</span>
<span class="kn">from</span> <span class="n">src.modules.free.ai.ai_assistant.ai_assistant_types</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">AssistantMessage</span><span class="p">,</span>
    <span class="n">AssistantResponse</span><span class="p">,</span>
<span class="p">)</span>


<span class="k">class</span> <span class="nc">SupportAgent</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">AI-powered customer support agent.</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">assistant</span><span class="p">:</span> <span class="n">AiAssistant</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_assistant</span> <span class="o">=</span> <span class="n">assistant</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_system_prompt</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">You are a helpful customer support agent.
Be concise, friendly, and solution-oriented.
If you don</span><span class="sh">'</span><span class="s">t know the answer, escalate to human support.</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">handle_ticket</span><span class="p">(</span>
        <span class="n">self</span><span class="p">,</span>
        <span class="n">customer_message</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
        <span class="n">ticket_history</span><span class="p">:</span> <span class="n">Sequence</span><span class="p">[</span><span class="n">AssistantMessage</span><span class="p">]</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>
    <span class="p">)</span> <span class="o">-&gt;</span> <span class="n">AssistantResponse</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Process customer support ticket.</span><span class="sh">"""</span>
        <span class="n">context</span> <span class="o">=</span> <span class="p">[</span>
            <span class="nc">AssistantMessage</span><span class="p">(</span><span class="n">role</span><span class="o">=</span><span class="sh">"</span><span class="s">system</span><span class="sh">"</span><span class="p">,</span> <span class="n">content</span><span class="o">=</span><span class="n">self</span><span class="p">.</span><span class="n">_system_prompt</span><span class="p">)</span>
        <span class="p">]</span>
        <span class="k">if</span> <span class="n">ticket_history</span><span class="p">:</span>
            <span class="n">context</span><span class="p">.</span><span class="nf">extend</span><span class="p">(</span><span class="n">ticket_history</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">_assistant</span><span class="p">.</span><span class="nf">chat</span><span class="p">(</span>
            <span class="n">customer_message</span><span class="p">,</span>
            <span class="n">provider</span><span class="o">=</span><span class="sh">"</span><span class="s">openai</span><span class="sh">"</span><span class="p">,</span>  <span class="c1"># or "support" for template
</span>            <span class="n">context</span><span class="o">=</span><span class="n">context</span><span class="p">,</span>
        <span class="p">)</span>

    <span class="k">def</span> <span class="nf">classify_urgency</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">message</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Classify ticket urgency using AI.</span><span class="sh">"""</span>
        <span class="n">prompt</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Classify urgency (low/medium/high): </span><span class="si">{</span><span class="n">message</span><span class="si">}</span><span class="sh">"</span>
        <span class="n">response</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">_assistant</span><span class="p">.</span><span class="nf">chat</span><span class="p">(</span>
            <span class="n">prompt</span><span class="p">,</span>
            <span class="n">provider</span><span class="o">=</span><span class="sh">"</span><span class="s">openai</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">settings</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">temperature</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.1</span><span class="p">},</span>  <span class="c1"># More deterministic
</span>        <span class="p">)</span>
        <span class="k">return</span> <span class="n">response</span><span class="p">.</span><span class="n">content</span><span class="p">.</span><span class="nf">strip</span><span class="p">().</span><span class="nf">lower</span><span class="p">()</span>
</code></pre>

</div>



<p><strong>Add route in <code>src/main.py</code>:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">fastapi</span> <span class="kn">import</span> <span class="n">Request</span>
<span class="kn">from</span> <span class="n">src.agents.support_agent</span> <span class="kn">import</span> <span class="n">SupportAgent</span>

<span class="nd">@app.post</span><span class="p">(</span><span class="sh">"</span><span class="s">/support/ticket</span><span class="sh">"</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">handle_support_ticket</span><span class="p">(</span>
    <span class="n">message</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">request</span><span class="p">:</span> <span class="n">Request</span><span class="p">,</span>
<span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
    <span class="n">assistant</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">app</span><span class="p">.</span><span class="n">state</span><span class="p">.</span><span class="n">ai_assistant</span>
    <span class="n">agent</span> <span class="o">=</span> <span class="nc">SupportAgent</span><span class="p">(</span><span class="n">assistant</span><span class="p">)</span>

    <span class="c1"># Classify urgency
</span>    <span class="n">urgency</span> <span class="o">=</span> <span class="n">agent</span><span class="p">.</span><span class="nf">classify_urgency</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>

    <span class="c1"># Generate response
</span>    <span class="n">response</span> <span class="o">=</span> <span class="n">agent</span><span class="p">.</span><span class="nf">handle_ticket</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>

    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">ticket_id</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">TKT-12345</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">urgency</span><span class="sh">"</span><span class="p">:</span> <span class="n">urgency</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">ai_response</span><span class="sh">"</span><span class="p">:</span> <span class="n">response</span><span class="p">.</span><span class="n">content</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">latency_ms</span><span class="sh">"</span><span class="p">:</span> <span class="n">response</span><span class="p">.</span><span class="n">latency_ms</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">next_action</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">escalate</span><span class="sh">"</span> <span class="k">if</span> <span class="n">urgency</span> <span class="o">==</span> <span class="sh">"</span><span class="s">high</span><span class="sh">"</span> <span class="k">else</span> <span class="sh">"</span><span class="s">monitor</span><span class="sh">"</span>
    <span class="p">}</span>
</code></pre>

</div>



<p><strong>Test support agent:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="s2">"http://localhost:8000/support/ticket?message=My%20payment%20failed"</span>
</code></pre>

</div>



<p><strong>Response:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"ticket_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TKT-12345"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"urgency"</span><span class="p">:</span><span class="w"> </span><span class="s2">"high"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"ai_response"</span><span class="p">:</span><span class="w"> </span><span class="s2">"I understand your payment failed. Let me help you resolve this. Can you provide your order ID and the error message you received?"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"latency_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">1156.3</span><span class="p">,</span><span class="w">
  </span><span class="nl">"next_action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"escalate"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  What You Built
</h2>

<p>A production-ready AI Agent API with:</p>

<p><strong>Core Features:</strong></p>

<ul>
<li>✅ Multi-provider AI assistant (Echo, Template, OpenAI)</li>
<li>✅ Chat completion endpoint</li>
<li>✅ Streaming responses</li>
<li>✅ Conversation context handling</li>
<li>✅ Response caching (instant cache hits)</li>
<li>✅ Health monitoring with metrics</li>
<li>✅ Provider switching at runtime</li>
</ul>

<p><strong>Advanced Features:</strong></p>

<ul>
<li>✅ Custom OpenAI integration</li>
<li>✅ Customer support agent</li>
<li>✅ Urgency classification</li>
<li>✅ Ticket routing logic</li>
</ul>

<p><strong>Production Ready:</strong></p>

<ul>
<li>✅ Type-safe with Pydantic models</li>
<li>✅ Health checks for monitoring</li>
<li>✅ Performance metrics (latency, cache hits)</li>
<li>✅ Error handling</li>
<li>✅ Async/streaming support</li>
<li>✅ Integration tests included</li>
</ul>

<p><strong>Total time:</strong> 10 minutes<br><br>
<strong>Custom code:</strong> ~50 lines<br><br>
<strong>Everything else:</strong> Generated by RapidKit</p>


<h2>
  
  
  Key Takeaways
</h2>
<h3>
  
  
  1. Provider-Agnostic Design
</h3>

<p>Switch AI providers without changing your code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Use OpenAI
</span><span class="n">response</span> <span class="o">=</span> <span class="n">assistant</span><span class="p">.</span><span class="nf">chat</span><span class="p">(</span><span class="n">prompt</span><span class="p">,</span> <span class="n">provider</span><span class="o">=</span><span class="sh">"</span><span class="s">openai</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Switch to Anthropic (when you add it)
</span><span class="n">response</span> <span class="o">=</span> <span class="n">assistant</span><span class="p">.</span><span class="nf">chat</span><span class="p">(</span><span class="n">prompt</span><span class="p">,</span> <span class="n">provider</span><span class="o">=</span><span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Fallback to template
</span><span class="n">response</span> <span class="o">=</span> <span class="n">assistant</span><span class="p">.</span><span class="nf">chat</span><span class="p">(</span><span class="n">prompt</span><span class="p">,</span> <span class="n">provider</span><span class="o">=</span><span class="sh">"</span><span class="s">support</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  2. Built-in Production Features
</h3>

<p>No need to implement yourself:</p>

<ul>
<li>Conversation history tracking</li>
<li>Response caching (faster + cheaper)</li>
<li>Streaming support</li>
<li>Health monitoring</li>
<li>Performance metrics</li>
</ul>

<h3>
  
  
  3. Module Architecture Benefits
</h3>

<p>The <code>ai_assistant</code> module follows RapidKit's standard:</p>

<ul>
<li>
<code>module.yaml</code> — Version and metadata</li>
<li>
<code>templates/</code> — FastAPI and NestJS variants</li>
<li>
<code>docs/usage.md</code> — Usage guide</li>
<li>Health endpoints built-in</li>
<li>Testing included</li>
</ul>

<h3>
  
  
  4. Extensibility
</h3>

<p>Easy to add providers:</p>

<ol>
<li>Implement <code>ChatProvider</code> protocol</li>
<li>Register with <code>assistant.register_provider()</code>
</li>
<li>Done</li>
</ol>

<p><strong>Example providers you can add:</strong></p>

<ul>
<li>Anthropic Claude</li>
<li>Google PaLM</li>
<li>Azure OpenAI</li>
<li>Cohere</li>
<li>Local models (Ollama, LM Studio)</li>
</ul>




<h2>
  
  
  Next Steps
</h2>

<p><strong>Enhance your AI agent:</strong></p>

<ol>
<li>
<strong>Add authentication:</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   rapidkit add module auth_core
</code></pre>

</div>



<ol>
<li>
<strong>Add database for chat history:</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   rapidkit add module db_postgres
</code></pre>

</div>



<ol>
<li>
<strong>Add Redis for distributed caching:</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   rapidkit add module redis
</code></pre>

</div>



<ol>
<li>
<strong>Add observability:</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   rapidkit add module observability
</code></pre>

</div>



<ol>
<li>
<strong>Deploy:</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   docker build <span class="nt">-t</span> ai-agent <span class="nb">.</span>
   docker run <span class="nt">-p</span> 8000:8000 ai-agent
</code></pre>

</div>






<h2>
  
  
  Module Customization
</h2>

<p><strong>Override configuration via environment:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">RAPIDKIT_AI_ASSISTANT_LOG_LEVEL</span><span class="o">=</span>DEBUG
<span class="nb">export </span><span class="nv">RAPIDKIT_AI_ASSISTANT_TIMEOUT</span><span class="o">=</span>60
<span class="nb">export </span><span class="nv">RAPIDKIT_AI_ASSISTANT_FEATURE_FLAG</span><span class="o">=</span><span class="nb">true</span>
</code></pre>

</div>



<p><strong>Custom overrides in <code>overrides.py</code>:</strong></p>

<p>See <code>/src/modules/free/ai/ai_assistant/overrides.py</code> for runtime hooks.</p>




<h2>
  
  
  Testing
</h2>

<p><strong>Run integration tests:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>rapidkit <span class="nb">test</span>
</code></pre>

</div>



<p><strong>Test specific module:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>poetry run pytest tests/modules/free/integration/ai/ai_assistant/ <span class="nt">-v</span>
</code></pre>

</div>



<p><strong>Module verification:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>rapidkit modules verify ai_assistant
</code></pre>

</div>






<h2>
  
  
  Troubleshooting
</h2>

<h3>
  
  
  Provider Not Found
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ProviderNotFoundError: Provider 'openai' is not registered
</code></pre>

</div>



<p><strong>Solution:</strong> Check provider is registered in config or manually.</p>

<h3>
  
  
  Timeout Errors
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>TimeoutError: Request exceeded 30 seconds
</code></pre>

</div>



<p><strong>Solution:</strong> Increase timeout:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">request_timeout_seconds</span><span class="pi">:</span> <span class="m">60</span>
</code></pre>

</div>



<h3>
  
  
  OpenAI API Errors
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Authentication Error: Invalid API key
</code></pre>

</div>



<p><strong>Solution:</strong> Check environment variable:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo</span> <span class="nv">$OPENAI_API_KEY</span>
</code></pre>

</div>






<h2>
  
  
  Learn More
</h2>

<ul>
<li>🌐 <a href="https://www.getrapidkit.com" rel="noopener noreferrer">RapidKit Website</a>
</li>
<li>📦 <a href="https://www.npmjs.com/package/rapidkit" rel="noopener noreferrer">npm Package</a>
</li>
<li>🐍 <a href="https://pypi.org/project/rapidkit-core/" rel="noopener noreferrer">PyPI Package</a>
</li>
<li>🧩 <a href="https://marketplace.visualstudio.com/items?itemName=rapidkit.rapidkit-vscode" rel="noopener noreferrer">VS Code Extension</a>
</li>
<li>📚 <a href="https://docs.rapidkit.top/modules/ai_assistant" rel="noopener noreferrer">AI Assistant Module Docs</a>
</li>
</ul>




<p><strong>You just built a production AI agent in 10 minutes.</strong></p>

<p>Now scale it.</p>

