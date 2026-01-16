---
Title: Building an Agentic Framework from scratch 🚀
Description: 
Author: Tyler Buell
Date: 2026-01-16T21:32:37.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Building an Agentic Framework for Smart, Reactive AI Agents
</h2>

<p>Artificial intelligence agents are increasingly part of modern software, but building them well is surprisingly tricky. In this article, I’ll share <strong>the design and development journey behind Reactive Agents</strong>, a framework I built to make <strong>reactive, adaptable AI agents easy to create</strong> while giving developers <strong>control, observability, and flexibility</strong>.</p>




<h2>
  
  
  The Challenge: Why Build a New Framework?
</h2>

<p>I was honestly curious what it would take to build agents that can think and reason using their own strategies similar to how we go about tasks. Agentic frameworks at the time had little control over how to build decent agents with less code, so I wanted to try my take at it.</p>

<p>Most existing agent frameworks either:</p>

<ul>
<li>Treat agents as <strong>simple LLM + tool loops</strong>
</li>
<li>Require verbose or repetitive setup</li>
<li>Limit flexibility in reasoning or strategy</li>
</ul>

<p>From day one, I wanted something different:</p>

<ul>
<li>Agents that can <strong>think, act, and pivot</strong> mid-task</li>
<li>Clear and <strong>type-safe APIs</strong> that are easy to reason about</li>
<li>
<strong>Composable strategies</strong> to adapt to any agentic task</li>
<li>Observable events so developers always know <strong>what the agent is doing and why</strong>
</li>
</ul>

<p>The goal was simple: <strong>make building smart, reactive AI agents intuitive, safe, and powerful</strong>, without sacrificing flexibility.</p>




<h2>
  
  
  Core Principles of Reactive Agents
</h2>

<p>When designing the framework, I focused on several key principles:</p>

<ol>
<li><strong>Reactive Decision-Making</strong></li>
</ol>

<ul>
<li>Agents respond to <strong>task inputs and intermediate results</strong> rather than following a fixed plan.</li>
<li>Built-in reflection allows them to <strong>pivot strategies</strong> if outcomes differ from expectations.</li>
</ul>

<ol>
<li><strong>Tool Integration and MCP Support</strong></li>
</ol>

<ul>
<li>Agents can execute <strong>custom Python tools</strong> or leverage <strong>MCP servers</strong> for modular workflows.</li>
<li>This enables distributed, multi-tool pipelines without complex orchestration.</li>
</ul>

<ol>
<li><strong>Observability and Transparency</strong></li>
</ol>

<ul>
<li>Every agent action emits <strong>observable events</strong>.</li>
<li>Developers can track reasoning, tool usage, and decision paths in real time.</li>
</ul>

<ol>
<li><strong>Composability and Type Safety</strong></li>
</ol>

<ul>
<li>Reasoning strategies are <strong>modular</strong>, easy to swap, extend, or combine.</li>
<li>Type-safe APIs reduce runtime errors and make development predictable.</li>
</ul>

<ol>
<li><strong>Ease of Use</strong></li>
</ol>

<ul>
<li>The <strong>builder pattern</strong> provides a clean, minimal boilerplate setup.</li>
<li>Quick startup time and wide model support make experimentation fast and rewarding.</li>
</ul>




<h2>
  
  
  Designing the Reactive Loop
</h2>

<p>At the heart of the framework is the <strong>reactive loop</strong>, which guides how an agent processes tasks:</p>

<ol>
<li>
<strong>Receive Task</strong> → Agent gets input from the user or system</li>
<li>
<strong>Reasoning Layer</strong> → Decide which tools or strategies to use</li>
<li>
<strong>Tool Execution</strong> → Run custom or MCP-based tools</li>
<li>
<strong>Observe Feedback</strong> → Track events, results, and outputs</li>
<li>
<strong>Reflect &amp; Adapt</strong> → Update strategy if needed, then repeat</li>
</ol>

<p>This architecture ensures agents are <strong>not static</strong>. They <strong>learn and adapt in-flight</strong>, making them suitable for a wide range of agentic tasks.</p>

<p><strong>Diagram Concept:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Task Input → Reasoning → Tool Execution → Observation → Reflection → Next Action
</code></pre>

</div>






<h2>
  
  
  Implementation Highlights
</h2>

<p>While building Reactive Agents, a few design choices stood out:</p>

<ul>
<li>
<strong>Builder Pattern:</strong> Simple, readable agent creation.</li>
<li>
<strong>Composable Strategies:</strong> Swap reasoning modules without rewriting the agent.</li>
<li>
<strong>Observable Events:</strong> Makes debugging and insight seamless.</li>
<li>
<strong>Type-Safe APIs:</strong> Catch errors early, improve developer confidence.</li>
<li>
<strong>MCP Integration:</strong> Supports distributed agent architectures and modular workflows.</li>
</ul>

<p>These choices collectively differentiate the framework from simpler orchestration layers or monolithic agent implementations.</p>




<h2>
  
  
  Example: Minimal Reactive Agent
</h2>

<p>Below is an <strong>accurate, documented-aligned example</strong> showing a reactive agent using <strong>both a custom Python tool and an MCP tool</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">reactive_agents.agents</span> <span class="kn">import</span> <span class="n">ReactAgentBuilder</span>
<span class="kn">from</span> <span class="n">reactive_agents.tools.decorators</span> <span class="kn">import</span> <span class="n">tool</span>

<span class="c1"># Define a simple custom tool
</span><span class="nd">@tool</span><span class="p">(</span><span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Greet a user</span><span class="sh">"</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">greet</span><span class="p">(</span><span class="n">name</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Use this tool to greet the user by their provided name</span><span class="sh">"""</span>
    <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Hello, </span><span class="si">{</span><span class="n">name</span><span class="si">}</span><span class="s">! Welcome to Reactive Agents.</span><span class="sh">"</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">agent</span> <span class="o">=</span> <span class="nf">await </span><span class="p">(</span>
        <span class="nc">ReactAgentBuilder</span><span class="p">()</span>
        <span class="p">.</span><span class="nf">with_name</span><span class="p">(</span><span class="sh">"</span><span class="s">DemoAgent</span><span class="sh">"</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">with_model</span><span class="p">(</span><span class="sh">"</span><span class="s">ollama:qwen3:4b</span><span class="sh">"</span><span class="p">)</span>          <span class="c1"># Example model string
</span>        <span class="p">.</span><span class="nf">with_tools</span><span class="p">([</span><span class="sh">"</span><span class="s">brave-search</span><span class="sh">"</span><span class="p">,</span> <span class="n">greet</span><span class="p">])</span>  <span class="c1"># Auto-detects MCP tools vs custom tools
</span>        <span class="p">.</span><span class="nf">with_observable_events</span><span class="p">()</span>              <span class="c1"># Track events
</span>        <span class="p">.</span><span class="nf">build</span><span class="p">()</span>
    <span class="p">)</span>

    <span class="n">result</span> <span class="o">=</span> <span class="k">await</span> <span class="n">agent</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span>
        <span class="sh">"</span><span class="s">Greet a new user and fetch latest news about AI.</span><span class="sh">"</span>
    <span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">result</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">asyncio</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="nf">main</span><span class="p">())</span>
</code></pre>

</div>



<p>This snippet demonstrates <strong>reactive reasoning, tool execution, and observable events</strong> while remaining digestible for new users.</p>




<h2>
  
  
  Lessons Learned While Building the Framework
</h2>

<ol>
<li>
<strong>Design for observability early</strong> – It’s hard to debug complex agents without event tracking.</li>
<li>
<strong>Make reasoning modular</strong> – Composable strategies allow experimentation without breaking existing code.</li>
<li>
<strong>Keep APIs intuitive</strong> – Developers should spend more time designing agents than wrestling with setup.</li>
<li>
<strong>Support multiple tools and providers</strong> – Flexibility is essential for real-world agentic tasks.</li>
</ol>




<h2>
  
  
  Looking Forward
</h2>

<p>Reactive Agents is <strong>alpha-stage</strong>, but production-ready. APIs may evolve, but the <strong>core principles are stable</strong>: reactive loops, composable reasoning, observability, and tool integration.</p>

<p>I hope this framework inspires developers to <strong>explore agentic AI in new ways</strong>, creating smarter applications that don’t just respond, but <strong>reason, act, and improve</strong> dynamically.</p>




<h2>
  
  
  Get Involved
</h2>

<p>Try it, star the repo, or contribute:</p>

<ul>
<li>GitHub: <a href="https://github.com/tylerjrbuell/reactive-agents" rel="noopener noreferrer">https://github.com/tylerjrbuell/reactive-agents</a>
</li>
</ul>

<p>Whether it’s extending reasoning strategies, adding tools, or improving observability, <strong>your contributions help shape the next generation of reactive AI agents</strong>.</p>

