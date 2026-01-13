---
Title: Why Ralph Wiggum Needs Lisa
Description: 
Author: Tony Casey
Date: 2026-01-13T21:51:04.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Building Persistent Memory for Autonomous AI Agents
</h1>

<p><em>How we built knowledge graph-powered memory for Claude Code that actually persists</em></p>




<h2>
  
  
  The Problem Nobody's Talking About
</h2>

<p>Ralph Wiggum changed everything for Claude Code users.</p>

<p>For the first time, you could set Claude loose on your codebase and let it work autonomously for hours. Teams shipped entire repositories overnight. One team famously ran a 3-month loop that built a complete programming language.</p>

<p>But there's a problem.</p>

<p>Every morning, Ralph forgets everything.</p>

<h3>
  
  
  The Groundhog Day Problem
</h3>

<p>Picture this: You run Ralph for 8 hours. He refactors your entire API layer, discovers patterns, makes architectural decisions, and learns your preferences.</p>

<p>You close the terminal.</p>

<p>The next morning, you start a new session.</p>

<p>Ralph: "Hi! What should we work on?"</p>

<p>You: "Remember yesterday when you—"</p>

<p>Ralph: "Yesterday?"</p>

<p>This is the <strong>Groundhog Day Problem</strong> of autonomous AI agents. They can run forever, but they can't remember anything.</p>

<h2>
  
  
  Why Existing Solutions Don't Work
</h2>

<p>You might think: "Just save the conversation history!"</p>

<p>We tried that. It doesn't work. Here's why:</p>

<h3>
  
  
  1. Context Window Limits
</h3>

<p>Claude's context window is large (200K tokens), but not infinite. An 8-hour Ralph session generates millions of tokens. You can't fit that in a context window.</p>

<p>Even if you could, it's inefficient. Claude doesn't need to re-read every line of every conversation. It needs structured knowledge.</p>

<h3>
  
  
  2. Text Embeddings Aren't Enough
</h3>

<p>Many memory solutions use vector embeddings:</p>

<ul>
<li>Chunk the conversation</li>
<li>Embed each chunk</li>
<li>Retrieve similar chunks when needed</li>
</ul>

<p>This works for semantic search, but fails for:</p>

<ul>
<li>
<strong>Temporal reasoning</strong>: "What changed between yesterday and today?"</li>
<li>
<strong>Relationship queries</strong>: "How does Module A relate to Module B?"</li>
<li>
<strong>Decision tracking</strong>: "Why did we choose approach X over Y?"</li>
</ul>

<h3>
  
  
  3. The Lost Context Problem
</h3>

<p>Even with embeddings, you lose:</p>

<ul>
<li>Causal relationships</li>
<li>Decision rationale</li>
<li>Pattern evolution over time</li>
<li>Cross-project learning</li>
</ul>

<h2>
  
  
  The Solution: Knowledge Graphs + Skills
</h2>

<p>Lisa takes a different approach. Instead of saving everything, we build <strong>structured knowledge</strong> using Graphiti's knowledge graph engine.</p>

<h3>
  
  
  Architecture Overview
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────┐
│                     Claude Code                          │
│                   (Ralph Wiggum)                         │
└──────────────┬──────────────────────────────────────────┘
               │
               │ Session events (hooks)
               │
┌──────────────▼──────────────────────────────────────────┐
│                    Lisa (Skills Layer)                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Memory  │  │  Tasks   │  │  Prompt  │   ...         │
│  │  Skill   │  │  Skill   │  │  Skill   │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└──────────────┬──────────────────────────────────────────┘
               │
               │ MCP Protocol
               │
┌──────────────▼──────────────────────────────────────────┐
│            Graphiti MCP Server                           │
│    (Knowledge Graph Engine)                              │
│                                                           │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Nodes: Entities (files, modules, decisions)    │   │
│  │  Edges: Relationships (modified, depends_on)    │   │
│  │  Facts: Timestamped knowledge                   │   │
│  │  Episodes: Conversation context                 │   │
│  └─────────────────────────────────────────────────┘   │
└──────────────┬──────────────────────────────────────────┘
               │
               │ Storage
               │
┌──────────────▼──────────────────────────────────────────┐
│         Neo4j (Local Docker or Zep Cloud)                │
└──────────────────────────────────────────────────────────┘
</code></pre>

</div>



<h3>
  
  
  How It Works
</h3>

<h4>
  
  
  1. Event Capture (Hooks)
</h4>

<p>Lisa uses Claude Code's hook system to capture events:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// session-start.ts</span>
<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">onSessionStart</span><span class="p">(</span><span class="nx">context</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Load relevant memories for this project</span>
  <span class="kd">const</span> <span class="nx">memories</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">mcpClient</span><span class="p">.</span><span class="nf">searchFacts</span><span class="p">({</span>
    <span class="na">group_id</span><span class="p">:</span> <span class="nx">context</span><span class="p">.</span><span class="nx">projectId</span><span class="p">,</span>
    <span class="na">query</span><span class="p">:</span> <span class="nx">context</span><span class="p">.</span><span class="nx">projectContext</span>
  <span class="p">});</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">systemPrompt</span><span class="p">:</span> <span class="s2">`You have these memories: </span><span class="p">${</span><span class="nx">memories</span><span class="p">}</span><span class="s2">`</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// user-prompt-submit.ts</span>
<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">onPromptSubmit</span><span class="p">(</span><span class="nx">prompt</span><span class="p">,</span> <span class="nx">context</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Store the prompt as an episode</span>
  <span class="k">await</span> <span class="nx">mcpClient</span><span class="p">.</span><span class="nf">addEpisode</span><span class="p">({</span>
    <span class="na">content</span><span class="p">:</span> <span class="nx">prompt</span><span class="p">,</span>
    <span class="na">timestamp</span><span class="p">:</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">(),</span>
    <span class="na">group_id</span><span class="p">:</span> <span class="nx">context</span><span class="p">.</span><span class="nx">projectId</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// session-stop.ts</span>
<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">onSessionStop</span><span class="p">(</span><span class="nx">context</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Extract and store learnings from this session</span>
  <span class="kd">const</span> <span class="nx">learnings</span> <span class="o">=</span> <span class="nf">extractLearnings</span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="nx">transcript</span><span class="p">);</span>

  <span class="k">await</span> <span class="nx">mcpClient</span><span class="p">.</span><span class="nf">addFacts</span><span class="p">(</span><span class="nx">learnings</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  2. Knowledge Extraction (Graphiti)
</h4>

<p>When you close a session, Lisa sends the transcript to Graphiti, which:</p>

<ol>
<li>
<strong>Extracts entities</strong>: files, functions, modules, people, concepts</li>
<li>
<strong>Identifies relationships</strong>: modified, depends_on, implements, etc.</li>
<li>
<strong>Creates facts</strong>: timestamped assertions ("File X was modified to fix bug Y")</li>
<li>
<strong>Links episodes</strong>: connects conversations to facts</li>
</ol>

<p>Example output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"nodes"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="nl">"uuid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"..."</span><span class="p">,</span><span class="w"> </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"src/api/users.ts"</span><span class="p">,</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"file"</span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="nl">"uuid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"..."</span><span class="p">,</span><span class="w"> </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Tony"</span><span class="p">,</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"person"</span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="nl">"uuid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"..."</span><span class="p">,</span><span class="w"> </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RESTful architecture"</span><span class="p">,</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"concept"</span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"edges"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"source"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Tony"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"target"</span><span class="p">:</span><span class="w"> </span><span class="s2">"src/api/users.ts"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"MODIFIED"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"fact"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Tony refactored users.ts to follow RESTful patterns"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"timestamp"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-01-11T10:30:00Z"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h4>
  
  
  3. Queryable Memory (Skills)
</h4>

<p>Next session, Claude can query this knowledge:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> lisa, what did we decide about the API architecture?

👧 Recent decision: You refactored the API to follow RESTful patterns.

   Specifically:
   - users.ts now uses resource-based endpoints
   - Moved from RPC-style to REST
   - Rationale: Better caching, clearer semantics

   Related files: users.ts, posts.ts, comments.ts
</code></pre>

</div>



<p>This isn't search. It's <strong>reasoning over a knowledge graph</strong>.</p>

<h3>
  
  
  Skills: Extensible Memory Modules
</h3>

<p>Lisa uses a "skills" architecture. Each skill is a specialized memory module:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.agents/skills/
├── memory/          # Core knowledge graph persistence
│   ├── scripts/memory.js
│   └── SKILL.md
├── tasks/           # Task tracking across sessions
│   ├── scripts/tasks.js
│   └── SKILL.md
├── prompt/          # Prompt history and patterns
│   ├── scripts/prompt.js
│   └── SKILL.md
└── lisa/            # Meta-skill for orchestration
    ├── scripts/storage.js
    └── SKILL.md
</code></pre>

</div>



<p>Each skill:</p>

<ul>
<li>Handles a specific type of memory</li>
<li>Uses Graphiti MCP under the hood</li>
<li>Can be invoked with natural language</li>
</ul>

<p>Example: The <code>tasks</code> skill<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> lisa, add task: refactor authentication

👧 Task added: <span class="s2">"Refactor authentication"</span>
   Status: todo
   Added to group: my-project
</code></pre>

</div>



<p>Later:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> lisa, what tasks are pending?

👧 Tasks:
   1. <span class="o">[</span>todo] Refactor authentication
   2. <span class="o">[</span>doing] Update API docs
   3. <span class="o">[</span><span class="k">done</span><span class="o">]</span> Fix login bug
</code></pre>

</div>



<p>Tasks persist across sessions, projects, and even months.</p>

<h2>
  
  
  The Ralph + Lisa Stack
</h2>

<p>Here's where it gets powerful. Combine Ralph (autonomous execution) with Lisa (persistent memory):</p>

<h3>
  
  
  Day 1: Ralph Learns
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start Ralph with a goal</span>
ralph <span class="nt">--goal</span> <span class="s2">"Refactor the API layer to follow RESTful principles"</span>

<span class="c"># Ralph runs for 6 hours:</span>
<span class="c"># - Analyzes current code</span>
<span class="c"># - Identifies non-RESTful patterns</span>
<span class="c"># - Refactors endpoints</span>
<span class="c"># - Updates tests</span>
<span class="c"># - Makes 47 commits</span>

<span class="c"># Meanwhile, Lisa is watching:</span>
<span class="c"># - Captures architectural decisions</span>
<span class="c"># - Stores refactoring patterns</span>
<span class="c"># - Tracks file relationships</span>
<span class="c"># - Records test updates</span>
</code></pre>

</div>



<h3>
  
  
  Day 2: Ralph Remembers
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start a new Ralph session</span>
ralph <span class="nt">--goal</span> <span class="s2">"Add new /products endpoint"</span>

<span class="c"># Lisa loads memories:</span>
<span class="c"># "Yesterday you refactored to RESTful patterns.</span>
<span class="c">#  Users endpoint uses: GET /users, POST /users, GET /users/:id</span>
<span class="c">#  You preferred Joi for validation.</span>
<span class="c">#  Tests follow AAA pattern."</span>

<span class="c"># Ralph uses this context:</span>
<span class="c"># - Creates /products with same pattern</span>
<span class="c"># - Uses Joi validation (without being told)</span>
<span class="c"># - Follows AAA test pattern</span>
<span class="c"># - Stays consistent with Day 1</span>
</code></pre>

</div>



<p><strong>The result:</strong> Ralph doesn't just loop. Ralph <strong>learns</strong>.</p>

<h2>
  
  
  Real-World Use Cases
</h2>

<h3>
  
  
  1. Multi-Day Features
</h3>

<p>Break a large feature across multiple sessions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>Day 1: <span class="s2">"Design the authentication system"</span>
Day 2: <span class="s2">"Implement JWT tokens"</span>
Day 3: <span class="s2">"Add refresh token rotation"</span>
Day 4: <span class="s2">"Write integration tests"</span>
</code></pre>

</div>



<p>Each day builds on previous decisions without re-explanation.</p>

<h3>
  
  
  2. Team Memory
</h3>

<p>Share knowledge across team members:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Alice's session</span>
alice&gt; <span class="s2">"We decided to use PostgreSQL for user data"</span>

<span class="c"># Bob's session (next day)</span>
bob&gt; <span class="s2">"What database should I use for orders?"</span>
lisa&gt; <span class="s2">"You're using PostgreSQL for user data. Use the same for orders."</span>
</code></pre>

</div>



<h3>
  
  
  3. Project Templates
</h3>

<p>Lisa learns your preferences:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># After several projects:</span>
<span class="o">&gt;</span> <span class="s2">"Start a new API project"</span>

lisa&gt; <span class="s2">"Creating Express project with:
       - TypeScript (you always use this)
       - Joi validation (your preference)
       - Jest for tests (95% coverage target)
       - PostgreSQL (your standard)

       Based on your last 3 projects."</span>
</code></pre>

</div>



<h3>
  
  
  4. Bug Pattern Recognition
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Week 1: Fix a null pointer bug</span>
<span class="o">&gt;</span> <span class="s2">"Fixed NPE in users.ts by adding null check"</span>

<span class="c"># Week 3: Similar code, similar bug</span>
lisa&gt; <span class="s2">"Warning: This looks like the null pointer pattern from users.ts.
       Consider adding the same null check."</span>
</code></pre>

</div>



<h2>
  
  
  Technical Deep Dive: Graphiti Integration
</h2>

<h3>
  
  
  Why Graphiti?
</h3>

<p>We evaluated several memory backends:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Solution</th>
<th>Pros</th>
<th>Cons</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Plain text files</strong></td>
<td>Simple</td>
<td>No querying, no relationships</td>
</tr>
<tr>
<td><strong>Vector embeddings</strong></td>
<td>Good for similarity</td>
<td>No temporal reasoning</td>
</tr>
<tr>
<td><strong>SQL database</strong></td>
<td>Queryable</td>
<td>Rigid schema, poor for graphs</td>
</tr>
<tr>
<td><strong>Graphiti</strong></td>
<td>Knowledge graphs, temporal reasoning, MCP protocol</td>
<td>Requires Neo4j</td>
</tr>
</tbody>
</table></div>

<p>Graphiti won because:</p>

<ol>
<li>
<strong>Knowledge Graphs</strong>: Natural fit for code relationships</li>
<li>
<strong>Temporal Reasoning</strong>: Tracks changes over time</li>
<li>
<strong>MCP Protocol</strong>: Standard integration with Claude Code</li>
<li>
<strong>Enterprise Ready</strong>: Battle-tested by Zep</li>
</ol>

<h3>
  
  
  MCP Integration
</h3>

<p>Lisa communicates with Graphiti via the Model Context Protocol (MCP):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// lib/mcp.ts - Simplified</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">McpClient</span> <span class="p">{</span>
  <span class="k">async</span> <span class="nf">searchFacts</span><span class="p">(</span><span class="nx">query</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">groupId</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nf">callTool</span><span class="p">(</span><span class="dl">'</span><span class="s1">search_facts</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="nx">query</span><span class="p">,</span>
      <span class="na">group_id</span><span class="p">:</span> <span class="nx">groupId</span><span class="p">,</span>
      <span class="na">max_facts</span><span class="p">:</span> <span class="mi">10</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">addEpisode</span><span class="p">(</span><span class="nx">content</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">groupId</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nf">callTool</span><span class="p">(</span><span class="dl">'</span><span class="s1">add_episode</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="nx">content</span><span class="p">,</span>
      <span class="na">group_id</span><span class="p">:</span> <span class="nx">groupId</span><span class="p">,</span>
      <span class="na">timestamp</span><span class="p">:</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This gives us:</p>

<ul>
<li>Standard protocol (not vendor-locked)</li>
<li>Multiple backend support (local Docker, Zep Cloud)</li>
<li>Future-proof architecture</li>
</ul>

<h3>
  
  
  Storage Modes
</h3>

<p>Lisa supports two storage modes:</p>

<h4>
  
  
  Local Mode (Docker)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># docker-compose.graphiti.yml</span>
<span class="na">services</span><span class="pi">:</span>
  <span class="na">neo4j</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">neo4j:latest</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">7474:7474"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">7687:7687"</span>

  <span class="na">graphiti</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">zepai/knowledge-graph-mcp:standalone</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">NEO4J_URI=bolt://neo4j:7687</span>
</code></pre>

</div>



<p><strong>Pros:</strong></p>

<ul>
<li>Full control</li>
<li>Private data</li>
<li>No API costs</li>
</ul>

<p><strong>Cons:</strong></p>

<ul>
<li>Requires Docker</li>
<li>Manual updates</li>
</ul>

<h4>
  
  
  Zep Cloud Mode
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># .agents/.env</span>
<span class="nv">GRAPHITI_ENDPOINT</span><span class="o">=</span>https://api.getzep.com
<span class="nv">GRAPHITI_API_KEY</span><span class="o">=</span>your-key-here
</code></pre>

</div>



<p><strong>Pros:</strong></p>

<ul>
<li>Zero setup</li>
<li>Managed updates</li>
<li>Team sharing</li>
</ul>

<p><strong>Cons:</strong></p>

<ul>
<li>Requires account</li>
<li>Data leaves localhost</li>
</ul>

<h3>
  
  
  Cache Fallback
</h3>

<p>If Graphiti is unavailable, Lisa falls back to local cache:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">try</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">facts</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">mcpClient</span><span class="p">.</span><span class="nf">searchFacts</span><span class="p">(</span><span class="nx">query</span><span class="p">);</span>
  <span class="c1">// Cache the result</span>
  <span class="k">await</span> <span class="nf">writeCache</span><span class="p">(</span><span class="dl">'</span><span class="s1">memory.log</span><span class="dl">'</span><span class="p">,</span> <span class="nx">facts</span><span class="p">);</span>
  <span class="k">return</span> <span class="nx">facts</span><span class="p">;</span>
<span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Return cached result</span>
  <span class="k">return</span> <span class="nf">readCache</span><span class="p">(</span><span class="dl">'</span><span class="s1">memory.log</span><span class="dl">'</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This ensures Lisa works even offline.</p>

<h2>
  
  
  Performance Considerations
</h2>

<h3>
  
  
  Token Efficiency
</h3>

<p>Instead of stuffing the full conversation history into every prompt, Lisa:</p>

<ol>
<li>Queries relevant facts: ~500 tokens</li>
<li>Injects into system prompt</li>
<li>Claude reasons over knowledge, not raw logs</li>
</ol>

<p><strong>Result:</strong> 10x reduction in context usage</p>

<h3>
  
  
  Query Speed
</h3>

<p>Graphiti queries are fast:</p>

<ul>
<li>Simple fact lookup: &lt;50ms</li>
<li>Complex graph traversal: &lt;200ms</li>
<li>Full project search: &lt;500ms</li>
</ul>

<p>Fast enough for real-time usage.</p>

<h3>
  
  
  Storage Scaling
</h3>

<p>Neo4j scales to billions of nodes. Unless you're storing every keystroke, you'll never hit limits.</p>

<p>Typical usage:</p>

<ul>
<li>1 session = 10-50 facts</li>
<li>100 sessions = 5,000 facts</li>
<li>Storage: &lt;1 MB</li>
</ul>

<h2>
  
  
  Lessons Learned
</h2>

<h3>
  
  
  1. Not Everything Deserves Memory
</h3>

<p>Early versions tried to remember everything. Bad idea.</p>

<p>Noise drowns signal. Now we filter:</p>

<ul>
<li>✅ Architectural decisions</li>
<li>✅ Bug patterns</li>
<li>✅ User preferences</li>
<li>❌ Routine operations</li>
<li>❌ Transient state</li>
</ul>

<h3>
  
  
  2. Timestamps Are Critical
</h3>

<p>Without timestamps, you can't answer:</p>

<ul>
<li>"What changed since yesterday?"</li>
<li>"When did we decide X?"</li>
<li>"What was the sequence?"</li>
</ul>

<p>Every fact needs <code>valid_at</code> and <code>created_at</code>.</p>

<h3>
  
  
  3. Group IDs Enable Multi-Project
</h3>

<p>Use <code>group_id</code> to separate projects:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">group_id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">project-alpha</span><span class="dl">"</span>  <span class="c1">// Work project</span>
<span class="nx">group_id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">project-beta</span><span class="dl">"</span>   <span class="c1">// Side project</span>
<span class="nx">group_id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">shared</span><span class="dl">"</span>         <span class="c1">// Team knowledge</span>
</code></pre>

</div>



<p>This prevents cross-contamination.</p>

<h3>
  
  
  4. Skills Beat Monoliths
</h3>

<p>The skills architecture allows:</p>

<ul>
<li>Independent development</li>
<li>Community contributions</li>
<li>Gradual learning curve</li>
</ul>

<p>Much better than one giant "memory" blob.</p>

<h2>
  
  
  The Future: What's Next
</h2>

<h3>
  
  
  Week 2-3: Ralph Integration Plugin
</h3>

<p>Official plugin for Ralph Wiggum:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @lisa/ralph-integration

<span class="c"># Automatic memory capture during Ralph loops</span>
<span class="c"># Zero configuration needed</span>
</code></pre>

</div>



<h3>
  
  
  Month 2: Team Features
</h3>

<ul>
<li>Shared memory pools</li>
<li>Permission controls</li>
<li>Audit logs</li>
</ul>

<h3>
  
  
  Month 3: Advanced Query
</h3>

<p>Natural language queries over knowledge:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="o">&gt;</span> <span class="s2">"Show me all API endpoints modified in the last week
   that depend on the auth module and have failing tests"</span>

lisa&gt; <span class="o">[</span>Graphical output with relationships]
</code></pre>

</div>



<h3>
  
  
  Long-term: Cross-Agent Memory
</h3>

<p>Imagine:</p>

<ul>
<li>
<strong>Cursor</strong> stores code patterns</li>
<li>
<strong>Claude Code</strong> stores decisions</li>
<li>
<strong>GitHub Copilot</strong> stores preferences</li>
<li>
<strong>Lisa</strong> unifies them all</li>
</ul>

<p>One memory graph. All your tools.</p>

<h2>
  
  
  Try It Yourself
</h2>

<h3>
  
  
  Quick Start
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install Lisa</span>
npx lisa init

<span class="c"># Choose local or cloud</span>
? Storage mode: Local <span class="o">(</span>Docker<span class="o">)</span>

<span class="c"># Lisa sets up everything</span>
✓ Docker Compose configured
✓ Graphiti MCP installed
✓ Skills deployed
✓ Hooks configured

<span class="c"># Start using it</span>
npx lisa

<span class="o">&gt;</span> lisa, remember that I prefer TypeScript over JavaScript

👧 Got it! I<span class="s1">'ll remember you prefer TypeScript.

&gt; lisa, what do you know about me?

👧 Recent memories:
   - You prefer TypeScript over JavaScript
   - You'</span>re working on the <span class="s1">'lisa'</span> project
</code></pre>

</div>



<h3>
  
  
  Integration with Ralph
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install both</span>
npx lisa init
npx @anthropic-ai/claude-code <span class="nb">install </span>ralph-wiggum

<span class="c"># Run Ralph</span>
ralph <span class="nt">--goal</span> <span class="s2">"Your goal here"</span>

<span class="c"># Lisa captures everything automatically</span>
<span class="c"># Next session: Ralph remembers</span>
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>Ralph Wiggum proved that autonomous AI agents can code for hours.</p>

<p>Lisa proves they can remember what they learned.</p>

<p>Together, they're not just tools. They're a platform for AI agents that actually get smarter over time.</p>

<p>The Simpsons accidentally gave us the perfect metaphor:</p>

<ul>
<li>
<strong>Ralph runs wild and breaks things</strong> (autonomous loops)</li>
<li>
<strong>Lisa remembers everything and makes it better</strong> (persistent memory)</li>
</ul>

<p>Stop repeating yourself. Lisa's listening.</p>




<p><strong>Try Lisa:</strong> <a href="https://github.com/TonyCasey/lisa" rel="noopener noreferrer">https://github.com/TonyCasey/lisa</a><br>
<strong>Discussions:</strong> <a href="https://github.com/TonyCasey/lisa/discussions" rel="noopener noreferrer">https://github.com/TonyCasey/lisa/discussions</a><br>
<strong>Contributing:</strong> See CONTRIBUTING.md</p>

<p>Built with:</p>

<ul>
<li>
<a href="https://github.com/getzep/graphiti" rel="noopener noreferrer">Graphiti</a> - Knowledge graph engine</li>
<li>
<a href="https://claude.com/claude-code" rel="noopener noreferrer">Claude Code</a> - AI coding assistant</li>
<li>
<a href="https://neo4j.com" rel="noopener noreferrer">Neo4j</a> - Graph database</li>
<li>
<a href="https://modelcontextprotocol.org" rel="noopener noreferrer">MCP</a> - Model Context Protocol</li>
</ul>




<p><em>If you found this useful, star the repo and share your experience. What would you build with an AI agent that never forgets?</em></p>

<h1>
  
  
  AI #MachineLearning #ClaudeCode #KnowledgeGraphs #DeveloperTools
</h1>

