---
Title: Best Practices for Building Agentic AI Systems.
Description: 
Author: Shayan
Date: 2025-08-14T21:30:46.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been experimenting with adding AI agents to <a href="https://userjot.com?utm_source=devto&amp;utm_medium=post&amp;utm_campaign=agentic-systems" rel="noopener noreferrer">UserJot</a>, our feedback, roadmap, and changelog platform. Not the simple "one prompt, one response" stuff. Real agent systems where multiple specialized agents communicate, delegate tasks, and somehow don't crash into each other.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu2ka1rgtzxlugqc50r4c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu2ka1rgtzxlugqc50r4c.png" alt="UserJot changelog agent interface screenshot" width="800" height="676"></a></p>

<p>The goal was to analyze customer feedback at scale. Find patterns across hundreds of posts. Auto-generate changelog entries. Things that were basically impossible to do manually. I spent weeks reverse engineering tools like Gemini CLI and OpenCode, running experiments, breaking things, fixing them, breaking them again. Just pushed a basic version to production as beta. It's working. Mostly.</p>

<p>Here's what I learned about building agent systems from studying what works in the wild and testing it myself.</p>

<h2>
  
  
  The Two-Tier Agent Model That Actually Works
</h2>

<p>Forget complex hierarchies. You need exactly two levels:</p>

<p><strong>Primary Agents</strong> handle the conversation. They understand context, break down tasks, and talk to users. Think of them as project managers who never write code.</p>

<p><strong>Subagents</strong> do one thing well. They get a task, complete it, return results. No memory. No context. Just pure function execution.</p>

<p>I tried three-tier systems. Four-tier systems. Agents talking to agents talking to agents. It all breaks down. Two tiers is the sweet spot.</p>

<p>Here's what I landed on:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User → Primary Agent (maintains context)
         ├─→ Research Agent (finds relevant feedback)
         ├─→ Analysis Agent (processes sentiment)
         └─→ Summary Agent (creates reports)
</code></pre>

</div>



<p>Each subagent runs in complete isolation. The primary agent handles all the orchestration. Simple.</p>

<h2>
  
  
  Stateless Subagents: The Most Important Rule
</h2>

<p>Every subagent call should be like calling a pure function. Same input, same output. No shared memory. No conversation history. No state.</p>

<p>This sounds limiting until you realize what it gives you:</p>

<ul>
<li>
<strong>Parallel execution</strong>: Run 10 subagents at once without them stepping on each other</li>
<li>
<strong>Predictable behavior</strong>: Same prompt always produces similar results</li>
<li>
<strong>Easy testing</strong>: Test each agent in isolation</li>
<li>
<strong>Simple caching</strong>: Cache results by prompt hash</li>
</ul>

<p>Here's how I structure subagent communication:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">Primary</span><span class="w"> </span><span class="err">→</span><span class="w"> </span><span class="err">Subagent</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"task"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Analyze sentiment in these 50 feedback items"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"context"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Focus on feature requests about mobile app"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="err">...</span><span class="p">],</span><span class="w">
  </span><span class="nl">"constraints"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"max_processing_time"</span><span class="p">:</span><span class="w"> </span><span class="mi">5000</span><span class="p">,</span><span class="w">
    </span><span class="nl">"output_format"</span><span class="p">:</span><span class="w"> </span><span class="s2">"structured_json"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">

</span><span class="err">//</span><span class="w"> </span><span class="err">Subagent</span><span class="w"> </span><span class="err">→</span><span class="w"> </span><span class="err">Primary</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"complete"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"result"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"positive"</span><span class="p">:</span><span class="w"> </span><span class="mi">32</span><span class="p">,</span><span class="w">
    </span><span class="nl">"negative"</span><span class="p">:</span><span class="w"> </span><span class="mi">8</span><span class="p">,</span><span class="w">
    </span><span class="nl">"neutral"</span><span class="p">:</span><span class="w"> </span><span class="mi">10</span><span class="p">,</span><span class="w">
    </span><span class="nl">"top_themes"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"navigation"</span><span class="p">,</span><span class="w"> </span><span class="s2">"performance"</span><span class="p">,</span><span class="w"> </span><span class="s2">"offline_mode"</span><span class="p">]</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"confidence"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.92</span><span class="p">,</span><span class="w">
  </span><span class="nl">"processing_time"</span><span class="p">:</span><span class="w"> </span><span class="mi">3200</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>No conversation. No "remember what we talked about." Just task in, result out.</p>

<h2>
  
  
  Task Decomposition: How to Break Things Down
</h2>

<p>You've got two strategies that work:</p>

<p><strong>Vertical decomposition</strong> for sequential tasks:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Analyze competitor pricing" →
  1. Gather pricing pages
  2. Extract pricing tiers
  3. Calculate per-user costs
  4. Compare with our pricing
</code></pre>

</div>



<p><strong>Horizontal decomposition</strong> for parallel work:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"Research top 5 competitors" →
  ├─ Research Competitor A
  ├─ Research Competitor B
  ├─ Research Competitor C
  ├─ Research Competitor D
  └─ Research Competitor E
  (all run simultaneously)
</code></pre>

</div>



<p>The trick is knowing when to use which. Sequential when there are dependencies. Parallel when tasks are independent. Mix them when needed.</p>

<p>I'm using mixed decomposition for feedback processing:</p>

<ol>
<li>
<strong>Phase 1 (Parallel)</strong>: Categorize feedback, extract sentiment, identify users</li>
<li>
<strong>Phase 2 (Sequential)</strong>: Group by theme → Prioritize by impact → Generate report</li>
</ol>

<p>Works every time.</p>

<h2>
  
  
  Communication Protocols That Don't Suck
</h2>

<p>Your agents need structured communication. Not "please analyze this when you get a chance." Actual structured protocols.</p>

<p>Every task from primary to subagent needs:</p>

<ul>
<li>Clear objective ("Find all feedback mentioning 'slow loading'")</li>
<li>Bounded context ("From the last 30 days")</li>
<li>Output specification ("Return as JSON with id, text, user fields")</li>
<li>Constraints ("Max 100 results, timeout after 5 seconds")</li>
</ul>

<p>Every response from subagent to primary needs:</p>

<ul>
<li>Status (complete/partial/failed)</li>
<li>Result (the actual data)</li>
<li>Metadata (processing time, confidence, decisions made)</li>
<li>Recommendations (follow-up tasks, warnings, limitations)</li>
</ul>

<p>No ambiguity. No interpretation. Just data.</p>

<h2>
  
  
  Agent Specialization Patterns
</h2>

<p>After studying OpenCode and other systems, I've found three ways to specialize agents that make sense:</p>

<p><strong>By capability</strong>: Research agents find stuff. Analysis agents process it. Creative agents generate content. Validation agents check quality.</p>

<p><strong>By domain</strong>: Legal agents understand contracts. Financial agents handle numbers. Technical agents read code.</p>

<p><strong>By model</strong>: Fast agents use Haiku for quick responses. Deep agents use Opus for complex reasoning. Multimodal agents handle images.</p>

<p>Don't over-specialize. I started with 15 different agent types. Now I have 6. Each one does one thing really well.</p>

<h2>
  
  
  Orchestration Patterns We Actually Use
</h2>

<p>Here are the four patterns that handle 95% of cases:</p>

<h3>
  
  
  Sequential Pipeline
</h3>

<p>Each output feeds the next input. Good for multi-step processes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Agent A → Agent B → Agent C → Result
</code></pre>

</div>



<p>I use this for report generation: gather data → analyze → format → deliver.</p>

<h3>
  
  
  MapReduce Pattern
</h3>

<p>Split work across multiple agents, combine results. Good for large-scale analysis.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>       ┌→ Agent 1 ─┐
Input ─┼→ Agent 2 ─┼→ Reducer → Result
       └→ Agent 3 ─┘
</code></pre>

</div>



<p>This is my go-to for feedback analysis in <a href="https://userjot.com?utm_source=devto&amp;utm_medium=post&amp;utm_campaign=agentic-systems" rel="noopener noreferrer">UserJot</a>. When someone's feedback board gets 1000+ posts (happens more than you'd think), I split them across 10 agents, each processes 100, then combine results. Takes 30 seconds instead of 5 minutes. Users get instant insights about what their customers actually want.</p>

<h3>
  
  
  Consensus Pattern
</h3>

<p>Multiple agents solve the same problem, compare answers. Good for critical decisions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>      ┌→ Agent 1 ─┐
Task ─┼→ Agent 2 ─┼→ Voting/Merge → Result
      └→ Agent 3 ─┘
</code></pre>

</div>



<p>I use this for sentiment analysis on important feedback. Three agents analyze independently, then we take the majority vote. Catches edge cases single agents miss.</p>

<h3>
  
  
  Hierarchical Delegation
</h3>

<p>Primary delegates to subagents, which can delegate to sub-subagents. Good for complex domains.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Primary Agent
  ├─ Subagent A
  │   ├─ Sub-subagent A1
  │   └─ Sub-subagent A2
  └─ Subagent B
</code></pre>

</div>



<p>Honestly, I rarely use this. Sounds good in theory, becomes a debugging nightmare in practice. Stick to two levels max.</p>

<p>In my beta version, I mostly use MapReduce for feedback analysis and Sequential for report generation. The fancy stuff rarely gets used.</p>

<h2>
  
  
  Context Management Without the Mess
</h2>

<p>How much context should subagents get? Less than you think.</p>

<p><strong>Level 1: Complete Isolation</strong>: Subagent gets only the specific task. Use this 80% of the time.</p>

<p><strong>Level 2: Filtered Context</strong>: Subagent gets curated relevant background. Use when task needs some history.</p>

<p><strong>Level 3: Windowed Context</strong>: Subagent gets last N messages. Use sparingly, usually breaks things.</p>

<p>I pass context three ways:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Explicit summary
</span><span class="sh">"</span><span class="s">Previous analysis found 3 critical bugs. Now check if they</span><span class="sh">'</span><span class="s">re fixed in v2.1</span><span class="sh">"</span>

<span class="c1"># Structured context
</span><span class="p">{</span>
  <span class="sh">"</span><span class="s">background</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Analyzing Q3 feedback</span><span class="sh">"</span><span class="p">,</span>
  <span class="sh">"</span><span class="s">previous_findings</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="sh">"</span><span class="s">slow_loading</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">login_issues</span><span class="sh">"</span><span class="p">],</span>
  <span class="sh">"</span><span class="s">current_task</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Find related issues in Q4</span><span class="sh">"</span>
<span class="p">}</span>

<span class="c1"># Reference passing
</span><span class="sh">"</span><span class="s">Analyze document_xyz for quality issues</span><span class="sh">"</span>
<span class="c1"># Subagent fetches document independently
</span></code></pre>

</div>



<p>Less context = more predictable behavior.</p>

<h2>
  
  
  Error Handling That Actually Handles Errors
</h2>

<p>Agents fail. A lot. Here's how I handle it:</p>

<p><strong>Graceful degradation chain</strong>:</p>

<ol>
<li>Subagent fails → Primary agent attempts task</li>
<li>Still fails → Try different subagent</li>
<li>Still fails → Return partial results</li>
<li>Still fails → Ask user for clarification</li>
</ol>

<p><strong>Retry strategies that work</strong>:</p>

<ul>
<li>Immediate retry for network failures</li>
<li>Retry with rephrased prompt for unclear tasks</li>
<li>Retry with different model for capability issues</li>
<li>Exponential backoff for rate limits</li>
</ul>

<p><strong>Failure communication</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"failed"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"error_type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"timeout"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"partial_result"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"processed"</span><span class="p">:</span><span class="w"> </span><span class="mi">45</span><span class="p">,</span><span class="w">
    </span><span class="nl">"total"</span><span class="p">:</span><span class="w"> </span><span class="mi">100</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"suggested_action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"retry_with_smaller_batch"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Always return something useful, even when failing.</p>

<h2>
  
  
  Performance Optimization Without Overthinking
</h2>

<p><strong>Model selection</strong>: Simple tasks get Haiku. Complex reasoning gets Sonnet. Critical analysis gets Opus. Don't use Opus for everything.</p>

<p><strong>Parallel execution</strong>: Identify independent tasks. Launch them simultaneously. I regularly run 5-10 agents in parallel.</p>

<p><strong>Caching</strong>: Cache by prompt hash. Invalidate after 1 hour for dynamic content, 24 hours for static. Saves 40% of API calls.</p>

<p><strong>Batching</strong>: Process 50 feedback items in one agent call instead of 50 separate calls. Obvious but often missed.</p>

<h2>
  
  
  Monitoring: What to Actually Track
</h2>

<p>Track these four things:</p>

<ul>
<li>Task success rate (are agents completing tasks?)</li>
<li>Response quality (confidence scores, validation rates)</li>
<li>Performance (latency, token usage, cost)</li>
<li>Error patterns (what's failing and why?)</li>
</ul>

<p>Here's my execution trace format:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Primary Agent Start [12:34:56]
  ├─ Feedback Analyzer Called
  │   ├─ Time: 2.3s
  │   ├─ Tokens: 1,250
  │   └─ Status: Success
  ├─ Sentiment Processor Called
  │   ├─ Time: 1.8s
  │   ├─ Tokens: 890
  │   └─ Status: Success
  └─ Total Time: 4.5s, Total Cost: $0.03
</code></pre>

</div>



<p>Simple. Readable. Actionable.</p>

<h2>
  
  
  What I Learned Building This at <a href="https://userjot.com?utm_source=devto&amp;utm_medium=post&amp;utm_campaign=agentic-systems" rel="noopener noreferrer">UserJot</a>
</h2>

<p>I've been testing this architecture with real feedback data from our users' boards. Everything from feature requests to bug reports to "wouldn't it be cool if" posts. Here's what surprised me:</p>

<p><strong>Stateless is non-negotiable</strong>. Every time I added state "just this once," things broke within days.</p>

<p><strong>Two tiers is enough</strong>. I tried complex hierarchies. They added complexity without adding value.</p>

<p><strong>Most tasks need simple agents</strong>. 90% of my agent calls use the simplest, fastest models.</p>

<p><strong>Explicit beats implicit</strong>. Clear task definitions. Structured responses. No magic.</p>

<p><strong>Parallel execution changes everything</strong>. Running 10 agents simultaneously turned 5-minute tasks into 30-second tasks.</p>

<h2>
  
  
  The Principles That Matter
</h2>

<p>After experimenting with this system and getting it to beta, here are the principles that actually matter:</p>

<ol>
<li>
<strong>Stateless by default</strong>: Subagents are pure functions</li>
<li>
<strong>Clear boundaries</strong>: Explicit task definitions and success criteria</li>
<li>
<strong>Fail fast</strong>: Quick failure detection and recovery</li>
<li>
<strong>Observable execution</strong>: Track everything, understand what's happening</li>
<li>
<strong>Composable design</strong>: Small, focused agents that combine well</li>
</ol>

<h2>
  
  
  Common Pitfalls I Hit (So You Don't Have To)
</h2>

<p><strong>The "Smart Agent" Trap</strong>: I tried making agents that could "figure out" what to do. They couldn't. Be explicit.</p>

<p><strong>The State Creep</strong>: "Just this one piece of state." Then another. Then another. Then everything breaks.</p>

<p><strong>The Deep Hierarchy</strong>: Four levels of agents seemed logical. It was a debugging nightmare.</p>

<p><strong>The Context Explosion</strong>: Passing entire conversation history to every agent. Tokens aren't free.</p>

<p><strong>The Perfect Agent</strong>: Trying to handle every edge case in one agent. Just make more specialized agents.</p>

<h2>
  
  
  Actually Implementing This
</h2>

<p>Start simple. One primary agent, two subagents. Get that working. Add agents as needed, not because you can.</p>

<p>Build monitoring from day one. You'll need it.</p>

<p>Test subagents in isolation. If they need context to work, they're not isolated enough.</p>

<p>Cache aggressively. Same prompt = same response = why call the API twice?</p>

<p>And remember: agents are tools, not magic. They're really good at specific tasks. They're terrible at figuring out what those tasks should be. That's your job.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxzr88187iq734kawa4l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxzr88187iq734kawa4l.png" alt="UserJot Dashboard" width="800" height="492"></a></p>

<p><em>P.S. If you're curious about how we're using these patterns to analyze user feedback at scale, <a href="https://userjot.com?utm_source=devto&amp;utm_medium=post&amp;utm_campaign=agentic-systems" rel="noopener noreferrer">UserJot</a> has a free plan. The agent features are still in beta, but the core feedback/roadmap/changelog stuff is rock solid. We eat our own dog food. All our product decisions come from patterns these agents find in our feedback.</em></p>

<h2>
  
  
  FAQ
</h2>

<h3>
  
  
  What's the minimum viable agent system?
</h3>

<p>One primary agent that maintains context, one subagent that does actual work. Start there. Add complexity only when needed.</p>

<h3>
  
  
  Should subagents really have zero memory?
</h3>

<p>Yes. The moment you add memory, you add state. The moment you add state, you add bugs. Keep them stateless.</p>

<h3>
  
  
  How do you handle rate limits with parallel agents?
</h3>

<p>Implement a token bucket rate limiter. Launch agents up to your limit, queue the rest. Process queued tasks as tokens become available.</p>

<h3>
  
  
  What's the best model for primary agents vs subagents?
</h3>

<p>Primary agents need reasoning ability. Use Sonnet or Opus. Subagents doing simple tasks can use Haiku. Match model to task complexity.</p>

<h3>
  
  
  How do you test agent systems?
</h3>

<p>Test subagents with fixed inputs and expected outputs. Test orchestration with mocked subagent responses. Test the full system with real but limited data.</p>

<h3>
  
  
  Can agents call other agents directly?
</h3>

<p>They can, but shouldn't. All communication should go through the primary agent. It's the only way to maintain control and visibility.</p>

<h3>
  
  
  What about agents that need to maintain conversation state?
</h3>

<p>That's the primary agent's job. It maintains state and passes relevant context to stateless subagents as needed.</p>

<h3>
  
  
  How do you handle long-running agent tasks?
</h3>

<p>Break them into smaller chunks. No agent task should run longer than 30 seconds. If it does, it needs to be decomposed further.</p>

<h3>
  
  
  What's the typical latency for multi-agent operations?
</h3>

<p>With parallel execution: 2 to 5 seconds for most operations. Sequential operations add 1 to 2 seconds per step. Cache hits are near-instant.</p>

<h3>
  
  
  Is this architecture overkill for simple use cases?
</h3>

<p>Yes. If you just need one API call with a prompt, use that. This architecture is for when you need multiple specialized agents working together.</p>

