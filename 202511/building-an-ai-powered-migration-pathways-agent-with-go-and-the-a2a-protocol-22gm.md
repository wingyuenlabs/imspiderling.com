---
Title: Building an AI-Powered Migration Pathways Agent with Go and the A2A Protocol
Description: 
Author: Obinna Aguwa
Date: 2025-11-03T21:42:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>Planning to relocate internationally? The visa research process can be overwhelming, with countless pathways, varying costs, complex requirements, and constantly changing policies. I built an AI agent to solve this: a real-time migration advisor that analyzes your profile and returns the single best visa pathway, complete with costs, processing times, and actionable next steps.</p>

<p>In this post, I'll walk through how I built this agent using <strong>Google's Gemini LLM</strong>, the <strong>A2A (Agent-to-Agent) protocol</strong>, and deployed it on <strong>Heroku</strong> for seamless integration with platforms like <a href="https://telex.im/" rel="noopener noreferrer">Telex.im.</a></p>

<p>First of all, Telex is a collaboration workspace like Slack that allows developers to create AI Agents called AI Coworkers. The AI coworkers will work like a real human colleague. I might go in-depth in another article. For now, let's jump into the step-by-step process I took in building this migration pathways AI Agent.</p>

<h2>
  
  
  What Does It Do?
</h2>

<p>The Migration Pathways Agent takes a simple natural language query like:</p>

<blockquote>
<p>"I'm a software engineer from Nigeria wanting to move to Canada with a $10,000 budget"<br>
And returns a structured response:<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Best Migration Option: Express Entry (Federal Skilled Worker)</span>
This pathway is often the most straightforward option for skilled professionals seeking permanent residency in Canada, especially if they possess strong English or French skills, relevant work experience, and educational qualifications.
<span class="gs">**Key Details:**</span>
<span class="p">-</span> Processing time: 6–12 months
<span class="p">-</span> Cost: $2,300 - $3,000 (including application fees, language testing, ECA, and settlement funds)
<span class="p">-</span> Success rate: Medium to High (dependent on CRS score relative to current cut-off scores)
<span class="p">-</span> Main requirements: Minimum of one year of skilled work experience, Language proficiency (CLB 7 or higher in English or French), Educational Credential Assessment (ECA) for foreign education
Next step: Take an IELTS or CELPIP test and get your educational credentials assessed by a recognized organization (e.g., World Education Services - WES).
</code></pre>

</div>



<h2>
  
  
  Architecture Overview
</h2>

<p>The system consists of three main components:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────┐
│ CLIENT/USER │
└─────────┬───────────┘
 │ HTTP
┌─────────▼───────────┐
│ A2A PROTOCOL │
│ ┌─────────────────┐ │
│ │ Agent Card │ │
│ │ JSON-RPC 2.0 │ │
│ └─────────────────┘ │
└─────────┬───────────┘
 │
┌─────────▼───────────┐
│ MIGRATION AGENT │
│ ┌─────────────────┐ │
│ │ Query Parser │ │
│ │ Task Manager │ │
│ │ Gemini Client │ │
│ └─────────────────┘ │
└─────────────────────┘
</code></pre>

</div>



<h3>
  
  
  1. A2A Protocol Layer
</h3>

<p>The A2A (Agent-to-Agent) protocol standardizes how AI agents communicate. I implemented two key endpoints:<br>
<strong>Agent Card</strong> (<code>GET /.well-known/agent.json</code>):<br>
Returns metadata about the agent's capabilities, supported methods, and configuration.<br>
<strong>Planner Endpoint</strong> (<code>POST /a2a/planner</code>):<br>
Accepts JSON-RPC 2.0 requests with methods:</p>

<ul>
<li>
<code>message/send</code> - Send a migration query</li>
<li>
<code>tasks/get</code> - Retrieve task results</li>
<li>
<code>tasks/send</code> - Legacy task submission
### 2. The Go Server
I chose Go for its simplicity, performance, and excellent Heroku support. The server structure:
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">MigrationAgent</span> <span class="k">struct</span> <span class="p">{</span>
<span class="err"> </span><span class="n">gemini</span> <span class="o">*</span><span class="n">GeminiClient</span>
<span class="err"> </span><span class="n">tasks</span> <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="o">*</span><span class="n">Task</span>
<span class="err"> </span><span class="n">mu</span> <span class="n">sync</span><span class="o">.</span><span class="n">RWMutex</span>
<span class="p">}</span>
</code></pre>

</div>


<p><strong>Key design decisions:</strong></p>

<ol>
<li>
<strong>In-memory task storage</strong>: Simple map for task management.</li>
<li>
<strong>Embedded agent card</strong>: Using Go's <code>embed</code> package to serve the agent.json</li>
<li>
<strong>Flexible message parsing</strong>: Supports both Telex's format and standard A2A
### 3. Gemini Integration
The magic happens in the Gemini client. Here's how I structured the prompt:
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">gc</span> <span class="o">*</span><span class="n">GeminiClient</span><span class="p">)</span> <span class="n">buildPrompt</span><span class="p">(</span><span class="n">userQuery</span><span class="p">,</span> <span class="n">_</span><span class="p">,</span> <span class="n">_</span> <span class="kt">string</span><span class="p">,</span> <span class="n">budget</span> <span class="kt">int</span><span class="p">)</span> <span class="kt">string</span> <span class="p">{</span>
<span class="err"> </span><span class="n">prompt</span><span class="err"> </span><span class="o">:=</span> <span class="s">`You are a migration planning expert.
CRITICAL BEHAVIOR RULES:
- Never ask the user for additional information
- Extract profession, origin country, and destination country from the query
- If information is unclear, make reasonable assumptions
- Output exactly ONE best migration option
USER QUERY:
"`</span> <span class="o">+</span> <span class="n">userQuery</span> <span class="o">+</span> <span class="s">`"
`</span>
<span class="err"> </span><span class="c">// … rest of prompt</span>
<span class="p">}</span>
</code></pre>

</div>


<p><strong>Why this approach?</strong><br>
Initially, I tried parsing professions, countries, and origins with hardcoded maps. Bad idea. Users said "USA" but the agent suggested Canada because my parsing was too rigid.<br>
<strong>The fix</strong>: Let Gemini extract ALL information from natural language. This supports any country, any profession, any language - no maintenance required.</p>
<h2>
  
  
  Implementation Deep Dive
</h2>
<h3>
  
  
  Step 1: Setting Up the Gemini Client
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">GeminiClient</span> <span class="k">struct</span> <span class="p">{</span>
<span class="err"> </span><span class="n">APIKey</span> <span class="kt">string</span>
<span class="err"> </span><span class="n">BaseURL</span> <span class="kt">string</span>
<span class="err"> </span><span class="n">Model</span> <span class="kt">string</span>
<span class="p">}</span>
<span class="k">func</span> <span class="n">NewGeminiClient</span><span class="p">()</span> <span class="o">*</span><span class="n">GeminiClient</span> <span class="p">{</span>
<span class="err"> </span><span class="k">return</span> <span class="o">&amp;</span><span class="n">GeminiClient</span><span class="p">{</span>
<span class="err"> </span><span class="n">APIKey</span><span class="o">:</span> <span class="n">os</span><span class="o">.</span><span class="n">Getenv</span><span class="p">(</span><span class="s">"GEMINI_API_KEY"</span><span class="p">),</span>
<span class="err"> </span><span class="n">BaseURL</span><span class="o">:</span> <span class="s">"https://generativelanguage.googleapis.com/v1beta"</span><span class="p">,</span>
<span class="err"> </span><span class="n">Model</span><span class="o">:</span> <span class="s">"gemini-2.0-flash-exp"</span><span class="p">,</span>
<span class="err"> </span><span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>


<p>I used <code>gemini-2.0-flash-exp</code> for its speed and strong reasoning capabilities - critical for extracting structured information from unstructured queries.</p>
<h3>
  
  
  Step 2: Task Processing Flow
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">a</span> <span class="o">*</span><span class="n">MigrationAgent</span><span class="p">)</span> <span class="n">ProcessTask</span><span class="p">(</span><span class="n">taskID</span> <span class="kt">string</span><span class="p">,</span> <span class="n">message</span> <span class="n">Message</span><span class="p">)</span> <span class="p">(</span><span class="o">*</span><span class="n">Task</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
<span class="err"> </span><span class="c">// 1. Create task with "working" state</span>
<span class="err"> </span><span class="n">task</span><span class="err"> </span><span class="o">:=</span> <span class="o">&amp;</span><span class="n">Task</span><span class="p">{</span>
<span class="err"> </span><span class="n">ID</span><span class="o">:</span> <span class="n">taskID</span><span class="p">,</span>
<span class="err"> </span><span class="n">Kind</span><span class="o">:</span> <span class="s">"task"</span><span class="p">,</span>
<span class="err"> </span><span class="n">Status</span><span class="o">:</span> <span class="n">TaskStatus</span><span class="p">{</span>
<span class="err"> </span><span class="n">State</span><span class="o">:</span> <span class="s">"working"</span><span class="p">,</span>
<span class="err"> </span><span class="n">Timestamp</span><span class="o">:</span> <span class="n">time</span><span class="o">.</span><span class="n">Now</span><span class="p">()</span><span class="o">.</span><span class="n">UTC</span><span class="p">()</span><span class="o">.</span><span class="n">Format</span><span class="p">(</span><span class="n">time</span><span class="o">.</span><span class="n">RFC3339</span><span class="p">),</span>
<span class="err"> </span><span class="p">},</span>
<span class="err"> </span><span class="p">}</span>
<span class="err"> </span>
<span class="err"> </span><span class="c">// 2. Extract user query from message parts</span>
<span class="err"> </span><span class="k">var</span> <span class="n">userQuery</span> <span class="kt">string</span>
<span class="err"> </span><span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">part</span><span class="err"> </span><span class="o">:=</span> <span class="k">range</span> <span class="n">message</span><span class="o">.</span><span class="n">Parts</span> <span class="p">{</span>
<span class="err"> </span><span class="k">if</span> <span class="n">part</span><span class="o">.</span><span class="n">Type</span> <span class="o">==</span> <span class="s">"text"</span> <span class="p">{</span>
<span class="err"> </span><span class="n">userQuery</span> <span class="o">+=</span> <span class="n">part</span><span class="o">.</span><span class="n">Text</span> <span class="o">+</span> <span class="s">" "</span>
<span class="err"> </span><span class="p">}</span>
<span class="err"> </span><span class="p">}</span>
<span class="err"> </span>
<span class="err"> </span><span class="c">// 3. Query Gemini</span>
<span class="err"> </span><span class="n">responseText</span><span class="p">,</span> <span class="n">err</span><span class="err"> </span><span class="o">:=</span> <span class="n">a</span><span class="o">.</span><span class="n">gemini</span><span class="o">.</span><span class="n">GetMigrationPathways</span><span class="p">(</span>
<span class="err"> </span><span class="n">userQuery</span><span class="p">,</span> <span class="s">""</span><span class="p">,</span> <span class="s">""</span><span class="p">,</span> <span class="n">profile</span><span class="o">.</span><span class="n">Budget</span><span class="p">,</span>
<span class="err"> </span><span class="p">)</span>
<span class="err"> </span>
<span class="err"> </span><span class="c">// 4. Update task with results</span>
<span class="err"> </span><span class="n">task</span><span class="o">.</span><span class="n">Status</span> <span class="o">=</span> <span class="n">TaskStatus</span><span class="p">{</span>
<span class="err"> </span><span class="n">State</span><span class="o">:</span> <span class="s">"completed"</span><span class="p">,</span>
<span class="err"> </span><span class="n">Message</span><span class="o">:</span> <span class="o">&amp;</span><span class="n">StatusMessage</span><span class="p">{</span>
<span class="err"> </span><span class="n">Kind</span><span class="o">:</span> <span class="s">"message"</span><span class="p">,</span>
<span class="err"> </span><span class="n">Role</span><span class="o">:</span> <span class="s">"agent"</span><span class="p">,</span>
<span class="err"> </span><span class="n">Parts</span><span class="o">:</span> <span class="p">[]</span><span class="n">Part</span><span class="p">{{</span><span class="n">Kind</span><span class="o">:</span> <span class="s">"text"</span><span class="p">,</span> <span class="n">Text</span><span class="o">:</span> <span class="n">responseText</span><span class="p">}},</span>
<span class="err"> </span><span class="n">MessageID</span><span class="o">:</span> <span class="n">messageID</span><span class="p">,</span>
<span class="err"> </span><span class="n">TaskID</span><span class="o">:</span> <span class="n">taskID</span><span class="p">,</span>
<span class="err"> </span><span class="p">},</span>
<span class="err"> </span><span class="p">}</span>
<span class="err"> </span>
<span class="err"> </span><span class="k">return</span> <span class="n">task</span><span class="p">,</span> <span class="no">nil</span>
<span class="p">}</span>
</code></pre>

</div>

<h3>
  
  
  Step 3: A2A Response Format
</h3>

<p>The A2A spec requires specific response structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
</span><span class="err"> </span><span class="nl">"jsonrpc"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2.0"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-id"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"result"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
</span><span class="err"> </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-id"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
</span><span class="err"> </span><span class="nl">"state"</span><span class="p">:</span><span class="w"> </span><span class="s2">"completed"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"timestamp"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025–11–03T…"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"message"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
</span><span class="err"> </span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"message"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"role"</span><span class="p">:</span><span class="w"> </span><span class="s2">"agent"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"parts"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="nl">"kind"</span><span class="p">:</span><span class="w"> </span><span class="s2">"text"</span><span class="p">,</span><span class="w"> </span><span class="nl">"text"</span><span class="p">:</span><span class="w"> </span><span class="s2">"…"</span><span class="p">}],</span><span class="w">
</span><span class="err"> </span><span class="nl">"messageId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"msg-id"</span><span class="p">,</span><span class="w">
</span><span class="err"> </span><span class="nl">"taskId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"task-id"</span><span class="w">
</span><span class="err"> </span><span class="p">}</span><span class="w">
</span><span class="err"> </span><span class="p">},</span><span class="w">
</span><span class="err"> </span><span class="nl">"artifacts"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="err">…</span><span class="p">]</span><span class="w">
</span><span class="err"> </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This took iteration - initially I missed <code>messageId</code> and <code>taskId</code> in the status message, causing Telex integration to fail.</p>

<h2>
  
  
  Deployment on Heroku
</h2>

<p><strong>1. Procfile</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>web: bin/server
</code></pre>

</div>



<p><strong>2. Dynamic Port Binding</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">port</span><span class="err"> </span><span class="o">:=</span> <span class="n">os</span><span class="o">.</span><span class="n">Getenv</span><span class="p">(</span><span class="s">"PORT"</span><span class="p">)</span>
<span class="k">if</span> <span class="n">port</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
<span class="err"> </span><span class="n">port</span> <span class="o">=</span> <span class="s">"8080"</span>
<span class="p">}</span>
<span class="n">addr</span><span class="err"> </span><span class="o">:=</span> <span class="s">":"</span> <span class="o">+</span> <span class="n">port</span>
</code></pre>

</div>



<p><strong>3. Environment Variables</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>heroku config:set <span class="nv">GEMINI_API_KEY</span><span class="o">=</span>your_key_here
</code></pre>

</div>



<p><strong>4. Deploy</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git push heroku main
</code></pre>

</div>



<p>The Heroku Go buildpack automatically runs <code>go install</code>, compiles the binary to <code>bin/server</code>, and the Procfile starts it.</p>

<h2>
  
  
  Key Learnings
</h2>

<h3>
  
  
  1. Don't Over-Engineer Parsing
</h3>

<p>My first version had elaborate country/profession dictionaries. Gemini handles this better than any regex or dictionary lookup.</p>

<h3>
  
  
  2. A2A Protocol Details Matter
</h3>

<p>Small spec violations (missing <code>messageId</code>, wrong JSON structure) break integrations. Test with actual A2A clients early.</p>

<h3>
  
  
  3. Prompt Engineering Is Critical
</h3>

<p>The difference between "suggest migration options" and "provide THE SINGLE BEST option without asking questions" is massive. The second version eliminates back-and-forth.</p>

<h3>
  
  
  4. LLM Temperature Matters
</h3>

<p>For structured output, low temperature (default) works better than high creativity settings.</p>

<h2>
  
  
  Testing
</h2>

<p>I created <code>.http</code> files for VS Code REST Client:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight http"><code><span class="err">POST {{host}}/a2a/planner
Content-Type: application/json
{
 "jsonrpc": "2.0",
 "method": "message/send",
 "params": {
 "message": {
 "role": "user",
 "parts": [{
 "type": "text",
 "text": "I am a nurse from Philippines wanting to move to UK"
 }]
 }
 },
 "id": 1
}
</span></code></pre>

</div>



<p>This made iteration fast - no need to rebuild integrations for every test.</p>

<h2>
  
  
  Try It Yourself
</h2>

<p><strong>Live API</strong>: <a href="https://migration-pathways-agent-ca4e1c945e86.herokuapp.com/a2a/planner" rel="noopener noreferrer">https://migration-pathways-agent-ca4e1c945e86.herokuapp.com/a2a/planner</a><br>
<strong>Agent Card</strong>: <a href="https://migration-pathways-agent-ca4e1c945e86.herokuapp.com/.well-known/agent.json" rel="noopener noreferrer">https://migration-pathways-agent-ca4e1c945e86.herokuapp.com/.well-known/agent.json</a><br>
<strong>Example cURL</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST https://migration-pathways-agent-ca4e1c945e86.herokuapp.com/a2a/planner <span class="se">\</span>
 -H <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
 -d <span class="s1">'{
 "jsonrpc": "2.0",
 "method": "message/send",
 "params": {
 "message": {
 "role": "user",
 "parts": [{
 "type": "text",
 "text": "I am a data scientist from India wanting to move to USA with $10000"
 }]
 }
 },
 "id": 1
 }'</span>
</code></pre>

</div>



<h2>
  
  
  Screenshot of the AI Agent on Telex
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F54hyyo1300wpt7v8zdxa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F54hyyo1300wpt7v8zdxa.png" alt=" " width="800" height="724"></a></p>

<h2>
  
  
  Conclusion
</h2>

<p>Building an AI agent isn't just about calling an LLM API - it's about:</p>

<ul>
<li>Choosing the right protocol (A2A for interoperability)</li>
<li>Crafting precise prompts that eliminate ambiguity</li>
<li>Structuring responses for both humans and machines</li>
<li>Deploying reliably with proper error handling
The Migration Pathways Agent demonstrates that with the right architecture, you can build practical, production-ready AI tools that solve real problems.</li>
</ul>

<p><strong>Repository</strong>: <a href="https://github.com/brainox/immigration-pathways-agent" rel="noopener noreferrer">github.com/brainox/immigration-pathways-agent</a><br>
 </p>

<p><em>Questions? Feedback? Hit me up on X  @ObinnaAguwa or try the agent and let me know what visa pathway it suggests for you!</em></p>

