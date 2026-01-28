---
Title: I built a brain for Claude Code because it keeps forgetting everything
Description: 
Author: CyborgNinja1
Date: 2026-01-28T21:09:06.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The frustration that started this
</h2>

<p>If you use Claude Code for real work, you've hit this wall: you're deep in a session, you've made architectural decisions, debugged tricky issues, established patterns — and then context compaction happens. Claude summarizes your conversation to free up tokens, and suddenly it's forgotten that you switched from MongoDB to PostgreSQL three hours ago.</p>

<p>You explain it again. It forgets again. Repeat.</p>

<p>I got tired of re-explaining my own codebase to my AI assistant. So I built Claude Cortex — a memory system that works like a brain, not a notepad.</p>

<h2>
  
  
  What it actually does
</h2>

<p>Claude Cortex is an MCP server that gives Claude Code three types of memory:</p>

<ul>
<li>
<strong>Short-term memory (STM)</strong> — session-level, high detail, decays within hours</li>
<li>
<strong>Long-term memory (LTM)</strong> — cross-session, consolidated from STM, persists for weeks</li>
<li>
<strong>Episodic memory</strong> — specific events: "when I tried X, Y happened"</li>
</ul>

<p>The key insight: not everything is worth remembering. The system scores every piece of information for <strong>salience</strong> — how important it actually is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Remember that we're using PostgreSQL" → architecture decision → 0.9 salience
"Fixed the auth bug by clearing the token cache" → error resolution → 0.8 salience  
"The current file has 200 lines" → temporary context → 0.2 salience (won't persist)
</code></pre>

</div>



<p>Memories also <strong>decay over time</strong>, just like human memory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>score = base_salience × (0.995 ^ hours_since_access)
</code></pre>

</div>



<p>But every time a memory is accessed, it gets reinforced by 1.2×. Frequently useful memories survive. One-off details fade away. This isn't a key-value store — it's a system that learns what matters.</p>

<h2>
  
  
  The compaction problem, solved
</h2>

<p>Here's the specific workflow that used to drive me nuts:</p>

<p><strong>Before Cortex:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Session starts → Work for 2 hours → Compaction happens → 
Claude: "What database are you using?" → You: *screams internally*
</code></pre>

</div>



<p><strong>After Cortex:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Session starts → Work for 2 hours → Compaction happens →
PreCompact hook auto-extracts 3-5 important memories →
Claude: "Let me check my memory..." → 
Recalls: PostgreSQL, JWT auth, React frontend, modular architecture →
Continues working seamlessly
</code></pre>

</div>



<p>The <code>PreCompact</code> hook is the secret weapon. It runs automatically before every compaction event, scanning the conversation for decisions, error fixes, learnings, and architecture notes. No manual intervention needed.</p>

<h2>
  
  
  v1.6.0: The intelligence overhaul
</h2>

<p>The first version was essentially CRUD-with-decay. It worked, but the subsystems were isolated — search didn't improve linking, linking didn't improve search, salience was set once and never evolved.</p>

<p>v1.6.0 was a seven-task overhaul to make everything feed back into everything else:</p>

<h3>
  
  
  1. Semantic linking via embeddings
</h3>

<p>Previously, memories only linked if they shared tags. Now, two memories about PostgreSQL with completely different tags will still link — the system computes embedding similarity and creates connections at ≥0.6 cosine similarity.</p>

<h3>
  
  
  2. Search feedback loops
</h3>

<p>Every search now does three things:</p>

<ul>
<li>Returns results (obviously)</li>
<li>
<strong>Reinforces salience</strong> of returned memories (with diminishing returns)</li>
<li>
<strong>Creates links</strong> between co-returned results</li>
</ul>

<p>Your search patterns literally shape the knowledge graph.</p>

<h3>
  
  
  3. Dynamic salience evolution
</h3>

<p>Salience isn't static anymore. During consolidation:</p>

<ul>
<li>Hub memories (lots of links) get a logarithmic bonus</li>
<li>Contradicted memories get a small penalty</li>
<li>The system learns which memories are structurally important</li>
</ul>

<h3>
  
  
  4. Contradiction surfacing
</h3>

<p>If you told Claude "use PostgreSQL" in January and "use MongoDB" in March, the system detects this and flags it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>⚠️ WARNING: Contradicts "Use PostgreSQL" (Memory #42)
</code></pre>

</div>



<p>No more silently holding conflicting information.</p>

<h3>
  
  
  5. Memory enrichment
</h3>

<p>Memories accumulate context over time. If you search for "JWT auth" and the query contains information the memory doesn't have, it gets appended. Memories grow richer through use.</p>

<h3>
  
  
  6. Real consolidation
</h3>

<p>The old system just deduplicated exact matches. Now it clusters related STM memories and merges them into coherent LTM entries:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>STM: "Set up JWT tokens with RS256 signing"
STM: "JWT tokens expire after 24 hours"
STM: "Added JWT verification middleware"

→ Consolidated LTM: "JWT authentication system using RS256 signing.
   Tokens expire after 24 hours with 7-day refresh tokens.
   Verification middleware on all protected routes."
</code></pre>

</div>



<p>Three noisy short-term memories become one structured long-term memory.</p>

<h3>
  
  
  7. Activation weight tuning
</h3>

<p>Recently activated memories get a meaningful boost in search results. If you just looked at something, it's more likely to be relevant again.</p>

<h2>
  
  
  Getting started
</h2>

<h3>
  
  
  Install
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-g</span> claude-cortex
</code></pre>

</div>



<h3>
  
  
  Configure Claude Code
</h3>

<p>Create <code>.mcp.json</code> in your project (or <code>~/.claude/.mcp.json</code> for global):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"memory"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"stdio"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npx"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"args"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"-y"</span><span class="p">,</span><span class="w"> </span><span class="s2">"claude-cortex"</span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Set up the PreCompact hook
</h3>

<p>Add to <code>~/.claude/settings.json</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"PreCompact"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"matcher"</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npx -y claude-cortex-hook pre-compact"</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Restart Claude Code, approve the MCP server, and you're done. Claude will start remembering things automatically.</p>

<h3>
  
  
  Use it naturally
</h3>

<p>You don't need to learn new commands. Just talk to Claude:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Remember that we're using PostgreSQL for the database"
"What do you know about our auth setup?"
"Get the context for this project"
</code></pre>

</div>



<p>The system handles categorization, salience scoring, and storage behind the scenes.</p>

<h2>
  
  
  The dashboard
</h2>

<p>There's also an optional 3D brain visualization dashboard — because honestly, watching memories form as glowing nodes in a neural network is just cool.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx claude-cortex service <span class="nb">install</span>  <span class="c"># auto-start on login</span>
</code></pre>

</div>



<p>It shows your memory graph in real-time via WebSocket, with search, filters, stats, and even a SQL console for poking at the database directly. Memories are color-coded: blue for architecture, purple for patterns, green for preferences, red for errors, yellow for learnings.</p>

<h2>
  
  
  How it compares
</h2>

<p>Most MCP memory tools are flat key-value stores. You manually save and manually retrieve. Claude Cortex is different in a few ways:</p>

<ul>
<li>
<strong>Salience detection</strong> — it decides what's worth remembering, not you</li>
<li>
<strong>Temporal decay</strong> — old irrelevant stuff fades naturally</li>
<li>
<strong>STM → LTM consolidation</strong> — short-term memories get merged into long-term ones</li>
<li>
<strong>Semantic linking</strong> — memories form a knowledge graph, not a list</li>
<li>
<strong>PreCompact hook</strong> — survives Claude Code's context compaction automatically</li>
</ul>

<p>It's not perfect. Embeddings add some latency. The consolidation heuristics are tuned for my workflows and might need adjustment for yours. The dashboard is a nice-to-have, not a must-have. But for the core problem — Claude forgetting things it shouldn't forget — it works really well.</p>

<h2>
  
  
  The stack
</h2>

<ul>
<li>TypeScript, compiled to ESM</li>
<li>SQLite with FTS5 for full-text search</li>
<li>
<code>@huggingface/transformers</code> for local embeddings (v1.6.1 fixed ARM64 support)</li>
<li>MCP protocol for Claude Code integration</li>
<li>React + Three.js for the dashboard</li>
<li>56 passing tests, MIT licensed</li>
</ul>

<h2>
  
  
  Try it out
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-g</span> claude-cortex
</code></pre>

</div>



<ul>
<li>
<strong>GitHub:</strong> <a href="https://github.com/mkdelta221/claude-cortex" rel="noopener noreferrer">github.com/mkdelta221/claude-cortex</a>
</li>
<li>
<strong>npm:</strong> <a href="https://www.npmjs.com/package/claude-cortex" rel="noopener noreferrer">npmjs.com/package/claude-cortex</a>
</li>
</ul>

<p>If you're using Claude Code for anything beyond quick one-offs, give it a shot. The difference between an AI that remembers your project and one that doesn't is night and day.</p>

<p>Stars and feedback welcome — this is a solo project and I'm iterating fast.</p>

