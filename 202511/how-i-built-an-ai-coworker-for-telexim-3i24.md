---
Title: How I Built an AI Coworker for Telex.im
Description: 
Author: Isaac
Date: 2025-11-03T22:14:35.000Z
Robots: noindex,nofollow
Template: index
---
<p>So I wanted to build an AI agent that could help me generate content, and I ended up creating something pretty cool - a conversational AI coworker that lives in Telex.im and generates blog posts.</p>

<p>But here's the thing - it started as a newsletter generator. Yep, completely different use case. Let me tell you how it evolved.</p>

<h2>
  
  
  The Newsletter Pivot 🔄
</h2>

<p>Originally, I was building this to generate newsletters. The idea was simple: give the AI a title and content, and it spits out a nicely formatted newsletter. I even had it generating infographics and returning them as artifacts with this structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"data"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"My Newsletter"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"content"</span><span class="p">:</span><span class="w"> </span><span class="s2">"..."</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>But when I tried integrating it with Telex.im, the artifacts weren't rendering correctly. The platform just didn't know what to do with that <code>data</code> format, or I was probably doing it wrong (that's more likely the case). So I thought, "Why not just make it simpler?" </p>

<p>I pivoted to blog posts and changed the artifact format to plain text:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"text"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"text"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Here's your blog post content..."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Way cleaner, and Telex.im loved it. Sometimes the simplest solution is the best one!</p>

<h2>
  
  
  What I Built
</h2>

<p>An AI agent that:</p>

<ul>
<li>Has natural conversations (not just one-shot queries)</li>
<li>Asks for blog title and topic step-by-step</li>
<li>Generates professional blog posts using Groq's fast LLM API</li>
<li>Remembers the entire conversation</li>
<li>Works seamlessly in Telex.im workflows</li>
</ul>

<h2>
  
  
  The Stack
</h2>

<p><strong>Go</strong>: Because I wanted something fast and concurrent<br><br>
<strong>LangChain Go</strong>: For managing conversations and LLM interactions<br><br>
<strong>Groq</strong>: Lightning-fast inference with Llama 3.1 (seriously, 750+ tokens/second!)<br><br>
<strong>JSON-RPC 2.0</strong>: Standard protocol for communication<br><br>
<strong>Telex.im</strong>: Where the magic happens</p>
<h2>
  
  
  How the Conversation Flows
</h2>

<p>The agent doesn't just take a prompt and generate. It actually guides you through:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Agent: "Want to create a blog post?"
You: "yes"
Agent: "What's the title?"
You: "Building AI Agents"
Agent: "What's the topic?"
You: "Go and LangChain"
Agent: "Should I proceed?"
You: "yes"
Agent: ✅ [generates full blog post]
</code></pre>

</div>



<p>This conversational approach makes it feel less like using a tool and more like working with a teammate.</p>

<h2>
  
  
  The Code Structure
</h2>

<p>I kept it clean with separation of concerns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>blog-generator/
├── main.go                 # Entry point
├── config/                 # API keys &amp; settings
├── requests/                 # Request/Response types
├── services/              
│   ├── agent_service.go    # Conversation logic
│   └── blog_service.go     # Content generation
└── controllers/            # HTTP handlers
</code></pre>

</div>



<h2>
  
  
  Key Components
</h2>

<h3>
  
  
  1. Configuration
</h3>

<p>First, load your Groq API key (grab one free at <a href="https://console.groq.com/keys" rel="noopener noreferrer">console.groq.com</a>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">config</span>

<span class="k">import</span> <span class="s">"os"</span>

<span class="k">type</span> <span class="n">Config</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">GroqAPIKey</span> <span class="kt">string</span>
    <span class="n">Port</span>       <span class="kt">string</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">Load</span><span class="p">()</span> <span class="o">*</span><span class="n">Config</span> <span class="p">{</span>
    <span class="n">groqAPIKey</span> <span class="o">:=</span> <span class="n">os</span><span class="o">.</span><span class="n">Getenv</span><span class="p">(</span><span class="s">"GROQ_API_KEY"</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">groqAPIKey</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
        <span class="nb">panic</span><span class="p">(</span><span class="s">"GROQ_API_KEY not set"</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="n">port</span> <span class="o">:=</span> <span class="n">os</span><span class="o">.</span><span class="n">Getenv</span><span class="p">(</span><span class="s">"PORT"</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">port</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
        <span class="n">port</span> <span class="o">=</span> <span class="s">"8080"</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="o">&amp;</span><span class="n">Config</span><span class="p">{</span>
        <span class="n">GroqAPIKey</span><span class="o">:</span> <span class="n">groqAPIKey</span><span class="p">,</span>
        <span class="n">Port</span><span class="o">:</span>       <span class="n">port</span><span class="p">,</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Blog Service - The Content Generator
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">BlogPostService</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">llm</span> <span class="n">llms</span><span class="o">.</span><span class="n">Model</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">NewBlogPostService</span><span class="p">(</span><span class="n">groqAPIKey</span> <span class="kt">string</span><span class="p">)</span> <span class="o">*</span><span class="n">BlogPostService</span> <span class="p">{</span>
    <span class="n">llm</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">openai</span><span class="o">.</span><span class="n">New</span><span class="p">(</span>
        <span class="n">openai</span><span class="o">.</span><span class="n">WithToken</span><span class="p">(</span><span class="n">groqAPIKey</span><span class="p">),</span>
        <span class="n">openai</span><span class="o">.</span><span class="n">WithBaseURL</span><span class="p">(</span><span class="s">"https://api.groq.com/openai/v1"</span><span class="p">),</span>
        <span class="n">openai</span><span class="o">.</span><span class="n">WithModel</span><span class="p">(</span><span class="s">"groq/compound"</span><span class="p">),</span>
    <span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="nb">panic</span><span class="p">(</span><span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"failed to create blogpost service: %v"</span><span class="p">,</span> <span class="n">err</span><span class="p">))</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="o">&amp;</span><span class="n">BlogPostService</span><span class="p">{</span><span class="n">llm</span><span class="o">:</span> <span class="n">llm</span><span class="p">}</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">s</span> <span class="o">*</span><span class="n">BlogPostService</span><span class="p">)</span> <span class="n">getSystemPrompt</span><span class="p">()</span> <span class="kt">string</span> <span class="p">{</span>
    <span class="k">return</span> <span class="s">`GENERATE_NEWSLETTER|Title: `</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">s</span> <span class="o">*</span><span class="n">BlogPostService</span><span class="p">)</span> <span class="n">Generate</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">session</span> <span class="o">*</span><span class="n">requests</span><span class="o">.</span><span class="n">SessionData</span><span class="p">,</span> <span class="n">title</span> <span class="kt">string</span><span class="p">)</span> <span class="p">(</span><span class="kt">string</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">messages</span> <span class="o">:=</span> <span class="p">[]</span><span class="n">llms</span><span class="o">.</span><span class="n">MessageContent</span><span class="p">{</span>
        <span class="n">llms</span><span class="o">.</span><span class="n">TextParts</span><span class="p">(</span><span class="n">llms</span><span class="o">.</span><span class="n">ChatMessageTypeSystem</span><span class="p">,</span> <span class="n">s</span><span class="o">.</span><span class="n">getSystemPrompt</span><span class="p">()),</span>
        <span class="n">llms</span><span class="o">.</span><span class="n">TextParts</span><span class="p">(</span><span class="n">llms</span><span class="o">.</span><span class="n">ChatMessageTypeSystem</span><span class="p">,</span> <span class="s">"You are a blog writer. Create engaging, well-structured blog posts."</span><span class="p">),</span>
        <span class="n">llms</span><span class="o">.</span><span class="n">TextParts</span><span class="p">(</span><span class="n">llms</span><span class="o">.</span><span class="n">ChatMessageTypeHuman</span><span class="p">,</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">`Create a cool blog post based on the following:

Title and Content: %s

Generate a well-structured blogpost with:
- A catchy headline
- An engaging introduction
-if you notice any code snippets, include it in the blog explanation
- 3-4 key points or sections with clear headings (use format "## Heading")
- A conclusion or call-to-action

Keep it concise and professional.`</span><span class="p">,</span> <span class="n">title</span><span class="p">)),</span>
    <span class="p">}</span>

    <span class="n">memoryVars</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">session</span><span class="o">.</span><span class="n">Memory</span><span class="o">.</span><span class="n">LoadMemoryVariables</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="n">any</span><span class="p">{})</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">==</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">if</span> <span class="n">history</span><span class="p">,</span> <span class="n">ok</span> <span class="o">:=</span> <span class="n">memoryVars</span><span class="p">[</span><span class="s">"history"</span><span class="p">]</span><span class="o">.</span><span class="p">(</span><span class="kt">string</span><span class="p">);</span> <span class="n">ok</span> <span class="o">&amp;&amp;</span> <span class="n">history</span> <span class="o">!=</span> <span class="s">""</span> <span class="p">{</span>
            <span class="n">messages</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">messages</span><span class="p">,</span> <span class="n">llms</span><span class="o">.</span><span class="n">TextParts</span><span class="p">(</span><span class="n">llms</span><span class="o">.</span><span class="n">ChatMessageTypeHuman</span><span class="p">,</span> <span class="n">history</span><span class="p">))</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="n">messages</span> <span class="o">=</span> <span class="nb">append</span><span class="p">(</span><span class="n">messages</span><span class="p">,</span> <span class="n">llms</span><span class="o">.</span><span class="n">TextParts</span><span class="p">(</span><span class="n">llms</span><span class="o">.</span><span class="n">ChatMessageTypeHuman</span><span class="p">,</span> <span class="n">title</span><span class="p">))</span>

    <span class="n">response</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">s</span><span class="o">.</span><span class="n">llm</span><span class="o">.</span><span class="n">GenerateContent</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">messages</span><span class="p">,</span> <span class="n">llms</span><span class="o">.</span><span class="n">WithTemperature</span><span class="p">(</span><span class="m">0.7</span><span class="p">))</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="s">""</span><span class="p">,</span> <span class="n">err</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="n">response</span><span class="o">.</span><span class="n">Choices</span><span class="p">[</span><span class="m">0</span><span class="p">]</span><span class="o">.</span><span class="n">Content</span><span class="p">,</span> <span class="no">nil</span>
<span class="p">}</span>

</code></pre>

</div>



<h3>
  
  
  3. Agent Service - The Brain
</h3>

<p>This is where the conversation magic happens:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">AgentService</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">llm</span>      <span class="n">llms</span><span class="o">.</span><span class="n">Model</span>
    <span class="n">blogSvc</span>  <span class="o">*</span><span class="n">BlogService</span>
    <span class="n">sessions</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="o">*</span><span class="n">models</span><span class="o">.</span><span class="n">SessionData</span>
    <span class="n">mu</span>       <span class="n">sync</span><span class="o">.</span><span class="n">RWMutex</span>  <span class="c">// Thread safety!</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>sync.RWMutex</code> is crucial. Each HTTP request runs in its own goroutine, so we need to protect the sessions map from concurrent access issues.</p>

<p><strong>Session management:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">s</span> <span class="o">*</span><span class="n">AgentService</span><span class="p">)</span> <span class="n">getOrCreateSession</span><span class="p">(</span><span class="n">taskID</span> <span class="kt">string</span><span class="p">)</span> <span class="o">*</span><span class="n">models</span><span class="o">.</span><span class="n">SessionData</span> <span class="p">{</span>
    <span class="n">s</span><span class="o">.</span><span class="n">mu</span><span class="o">.</span><span class="n">Lock</span><span class="p">()</span>
    <span class="k">defer</span> <span class="n">s</span><span class="o">.</span><span class="n">mu</span><span class="o">.</span><span class="n">Unlock</span><span class="p">()</span>

    <span class="k">if</span> <span class="n">session</span><span class="p">,</span> <span class="n">exists</span> <span class="o">:=</span> <span class="n">s</span><span class="o">.</span><span class="n">sessions</span><span class="p">[</span><span class="n">taskID</span><span class="p">];</span> <span class="n">exists</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">session</span>
    <span class="p">}</span>

    <span class="c">// Create new session with LangChain memory</span>
    <span class="n">session</span> <span class="o">:=</span> <span class="o">&amp;</span><span class="n">models</span><span class="o">.</span><span class="n">SessionData</span><span class="p">{</span>
        <span class="n">ContextID</span><span class="o">:</span> <span class="n">uuid</span><span class="o">.</span><span class="n">New</span><span class="p">()</span><span class="o">.</span><span class="n">String</span><span class="p">(),</span>
        <span class="n">History</span><span class="o">:</span>   <span class="p">[]</span><span class="n">models</span><span class="o">.</span><span class="n">HistoryMessage</span><span class="p">{},</span>
        <span class="n">Memory</span><span class="o">:</span>    <span class="n">memory</span><span class="o">.</span><span class="n">NewConversationBuffer</span><span class="p">(),</span>
    <span class="p">}</span>
    <span class="n">s</span><span class="o">.</span><span class="n">sessions</span><span class="p">[</span><span class="n">MessageID</span><span class="p">]</span> <span class="o">=</span> <span class="n">session</span>
    <span class="k">return</span> <span class="n">session</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>ConversationBuffer</code> from LangChain automatically tracks the full conversation history, so the AI remembers everything that was said.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">s</span> <span class="o">*</span><span class="n">AgentService</span><span class="p">)</span> <span class="n">processAIResponse</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">session</span> <span class="o">*</span><span class="n">models</span><span class="o">.</span><span class="n">SessionData</span><span class="p">,</span> <span class="n">taskID</span><span class="p">,</span> <span class="n">aiResponse</span> <span class="kt">string</span><span class="p">)</span> <span class="o">*</span><span class="n">models</span><span class="o">.</span><span class="n">TaskResult</span> <span class="p">{</span>
    <span class="c">// Check for generation command</span>
    <span class="k">if</span> <span class="n">strings</span><span class="o">.</span><span class="n">Contains</span><span class="p">(</span><span class="n">aiResponse</span><span class="p">,</span> <span class="s">"GENERATE_BLOG|"</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">s</span><span class="o">.</span><span class="n">handleBlogGeneration</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">session</span><span class="p">,</span> <span class="n">aiResponse</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="c">// Otherwise just return the AI's message</span>
    <span class="k">return</span> <span class="n">s</span><span class="o">.</span><span class="n">buildResponse</span><span class="p">(</span><span class="n">taskID</span><span class="p">,</span> <span class="n">session</span><span class="p">,</span> <span class="n">aiResponse</span><span class="p">)</span>
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">s</span> <span class="o">*</span><span class="n">AgentService</span><span class="p">)</span> <span class="n">handleBlogPostGeneration</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">session</span> <span class="o">*</span><span class="n">requests</span><span class="o">.</span><span class="n">SessionData</span><span class="p">,</span> <span class="n">userText</span> <span class="kt">string</span><span class="p">)</span> <span class="p">(</span><span class="kt">string</span><span class="p">,</span> <span class="p">[]</span><span class="n">requests</span><span class="o">.</span><span class="n">Artifact</span><span class="p">)</span> <span class="p">{</span>


    <span class="n">title</span> <span class="o">:=</span> <span class="n">userText</span>

    <span class="n">session</span><span class="o">.</span><span class="n">Title</span> <span class="o">=</span> <span class="n">title</span>

    <span class="n">blogpost</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">s</span><span class="o">.</span><span class="n">blogpostSvc</span><span class="o">.</span><span class="n">Generate</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">session</span><span class="p">,</span> <span class="n">title</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"I apologize, but I encountered an error generating the blogpost: %v</span><span class="se">\n\n</span><span class="s">Would you like to try again?"</span><span class="p">,</span> <span class="n">err</span><span class="p">),</span> <span class="no">nil</span>
    <span class="p">}</span>

    <span class="n">session</span><span class="o">.</span><span class="n">BlogPost</span> <span class="o">=</span> <span class="n">blogpost</span>

    <span class="n">blog</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"</span><span class="se">\n</span><span class="s">Title: %s </span><span class="se">\n\n</span><span class="s"> Content: %s"</span><span class="p">,</span> <span class="n">title</span><span class="p">,</span> <span class="n">blogpost</span><span class="p">)</span>

    <span class="n">artifact</span> <span class="o">:=</span> <span class="n">requests</span><span class="o">.</span><span class="n">Artifact</span><span class="p">{</span>
        <span class="n">ArtifactID</span><span class="o">:</span> <span class="n">uuid</span><span class="o">.</span><span class="n">New</span><span class="p">()</span><span class="o">.</span><span class="n">String</span><span class="p">(),</span>
        <span class="n">Name</span><span class="o">:</span>       <span class="s">"blogpost"</span><span class="p">,</span>
        <span class="n">Parts</span><span class="o">:</span> <span class="p">[]</span><span class="n">requests</span><span class="o">.</span><span class="n">ResponsePart</span><span class="p">{</span>
            <span class="p">{</span>
                <span class="n">Kind</span><span class="o">:</span> <span class="s">"text"</span><span class="p">,</span>
                <span class="n">Text</span><span class="o">:</span> <span class="n">blog</span><span class="p">,</span>
            <span class="p">},</span>
        <span class="p">},</span>
    <span class="p">}</span>

    <span class="n">response</span> <span class="o">:=</span> <span class="s">"BlogPost generated successfully!"</span>

    <span class="k">return</span> <span class="n">response</span><span class="p">,</span> <span class="p">[]</span><span class="n">requests</span><span class="o">.</span><span class="n">Artifact</span><span class="p">{</span><span class="n">artifact</span><span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This separates the AI's decision-making from the implementation. Clean and modular!</p>

<h2>
  
  
  Integrating with Telex.im 🚀
</h2>

<p>This is where things got really fun. I wanted to turn my agent into a proper AI coworker in Telex.im.</p>

<p>I followed the docs at <a href="https://docs.telex.im/docs" rel="noopener noreferrer">docs.telex.im</a> and checked out <a href="https://fynix.dev/blog" rel="noopener noreferrer">Pheonix's blog</a> for some integration patterns. Both were super helpful!</p>

<h3>
  
  
  My Telex.im Workflow
</h3>

<p>Here's the workflow config I'm using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"active"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"category"</span><span class="p">:</span><span class="w"> </span><span class="s2">"utilities"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"A workflow that creates blogposts"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"iceu-blogpost-001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"blogpost_agent"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"long_description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"You are a helpful assistant that provides sample BlogPosts.</span><span class="se">\n\n</span><span class="s2">Your primary function is to help users generate BlogPosts from title and description they send. When responding:</span><span class="se">\n</span><span class="s2">- Always ask for the blog title if none is provided</span><span class="se">\n</span><span class="s2">- Ask for the content/topic description</span><span class="se">\n</span><span class="s2">- Confirm the details and ask if they want to proceed</span><span class="se">\n</span><span class="s2">- Keep responses concise but informative</span><span class="se">\n</span><span class="s2">- Use the blogpostgeneration tool with title and content"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"short_description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"creates blogpost from title and content given"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"nodes"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"blogpost_agent"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"blogpost agent"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{},</span><span class="w">
      </span><span class="nl">"position"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="mi">816</span><span class="p">,</span><span class="w"> </span><span class="mi">-112</span><span class="p">],</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"a2a/mastra-a2a-node"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"typeVersion"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
      </span><span class="nl">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"https://newsletter-ai-coworker-i-ceu4383-amxl8p89.leapcell.dev/agent"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"pinData"</span><span class="p">:</span><span class="w"> </span><span class="p">{},</span><span class="w">
  </span><span class="nl">"settings"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"executionOrder"</span><span class="p">:</span><span class="w"> </span><span class="s2">"v1"</span><span class="w">
  </span><span class="p">}</span><span class="w">  
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The workflow is pretty straightforward - it's a single node that points to my agent endpoint. The <code>long_description</code> gives Telex.im context about what my agent does and how to interact with it.</p>

<h3>
  
  
  How It Works in Telex
</h3>

<ol>
<li>User messages the workflow in Telex.im</li>
<li>Telex sends a JSON-RPC request to my endpoint</li>
<li>My agent processes it and responds conversationally</li>
<li>Telex displays the response to the user</li>
<li>The conversation continues naturally</li>
</ol>

<p>The cool part is that Telex maintains the <code>taskId</code> across messages, so my agent can track the full conversation and remember context.</p>

<h2>
  
  
  Running It
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Set your API key</span>
<span class="nb">export </span><span class="nv">GROQ_API_KEY</span><span class="o">=</span><span class="s2">"gsk_your_key_here"</span>

<span class="c"># Run</span>
go run main.go
</code></pre>

</div>



<p>Deploy it anywhere (I used Leapcell), and you've got yourself an AI coworker!</p>

<h2>
  
  
  What I Learned
</h2>

<p><strong>1. Start simple</strong> - I spent time building infographic generation, but for my use case, plain text was all I needed.</p>

<p><strong>2. Architecture matters</strong> - Separating the blog service from the agent service made it super easy to test and iterate.</p>

<p><strong>3. Conversation &gt; Commands</strong> - Making it conversational instead of a one-shot API feels way more natural.</p>

<p><strong>4. LangChain is awesome</strong> - The built-in memory management saved me from writing a bunch of boilerplate.</p>

<p><strong>5. Groq is fast</strong> - Seriously, the speed difference compared to other providers is wild.</p>

<h2>
  
  
  The Result
</h2>

<p>Now I have an AI coworker living in Telex.im that I can message anytime like:</p>

<blockquote>
<p>Me: "hey, need a blog post"<br><br>
Agent: "Sure! What's the title?"<br><br>
Me: "Building with Go"<br><br>
Agent: "Great! What should it be about?"<br><br>
Me: "Microservices patterns"<br><br>
Agent: "Got it! Should I proceed?"<br><br>
Me: "yes"<br><br>
Agent: ✅ [Full blog post appears]</p>
</blockquote>

<p>It's like having a content writer on standby. Pretty neat for when I need to quickly draft technical content!<br>
I'll hopefully work on fully integrating it into platforms like here or hashnode so it creates the blogs automatically. <br>
Might be a fun next step.</p>

<h2>
  
  
  Resources
</h2>

<p>Want to build your own?</p>

<ul>
<li><a href="https://github.com/tmc/langchaingo" rel="noopener noreferrer">LangChain Go docs</a></li>
<li><a href="https://console.groq.com" rel="noopener noreferrer">Groq console</a></li>
<li><a href="https://docs.telex.im/docs" rel="noopener noreferrer">Telex.im docs</a></li>
<li><a href="https://fynix.dev/blog" rel="noopener noreferrer">Pheonix's blog</a></li>
</ul>




<p>Have you built any AI agents? What are you using them for? Drop a comment below - I'd love to hear about your projects! 🚀</p>

<p><strong>Tags:</strong> #go #ai #langchain #groq #telex</p>

