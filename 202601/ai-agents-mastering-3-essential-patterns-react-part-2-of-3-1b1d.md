---
Title: AI Agents: Mastering 3 Essential Patterns (ReAct). Part 2 of 3
Description: 
Author: Gabriel Melendez
Date: 2026-01-04T21:45:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Article [<a href="https://dev.to/gabrielmrojas/ai-agents-mastering-3-essential-patterns-tool-using-part-1-of-3-5d92">Part 1</a>]</p>

<p>The code for these patterns is available on Github. [<a href="https://github.com/gabrielmrojas/1-AI-Agent-Fundamentals-Patterns" rel="noopener noreferrer">Repo</a>]</p>

<h2>
  
  
  The ReAct Pattern (Reason + Act) – When the Agent Starts "Talking to Itself"
</h2>

<p>With the "Tool Using" pattern (Article 1), we gave the AI "hands" to interact with the outside world. With <strong>ReAct</strong>, we are installing a "functional brain."</p>

<p>Let's recap: in Article 1, everything was very linear. Stimulus → Response. You asked for a piece of data, the agent fired a tool, and that was it. No doubts, no plans. But let's be realistic, the real world is messy. What happens when the question isn't solved with a single click? What happens when you need to investigate, compare data, and then perform a calculation?</p>

<p>That is where the linear model crashes and <strong>ReAct</strong> (proposed by some very smart folks at Princeton and Google in 2022) comes into play, which is basically the industry standard today.</p>

<h2>
  
  
  What exactly is ReAct?
</h2>

<p>The name comes from <strong>Re</strong>asoning + <strong>Act</strong>ing.</p>

<p>The breakthrough idea here isn't just using tools (we already had that), but that we force the LLM to have an <strong>Internal Monologue</strong> ("Thought Trace"). Instead of rushing to answer blindly, the agent enters a cognitive loop:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuyi020bmcsyu1lrpvrao.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuyi020bmcsyu1lrpvrao.jpg" alt="Image 1" width="800" height="436"></a></p>

<p>The agent literally "talks to itself" in the logs. It plans its next step based on what it just discovered a second ago. This allows it to course-correct on the fly if things don't go as expected.</p>

<h3>
  
  
  Breaking Down the Loop: The Case of the "Historical Detective"
</h3>

<p>Let's see what happens under the hood with the example we just built. The user asks: <em>"What is the square root of the age of the current President of France?"</em></p>

<p>If you ask this to a raw model, it will hallucinate or tell you it doesn't know today's date. But watch how a ReAct agent handles it step-by-step:</p>

<ul>
<li><p><strong>Iteration 1:</strong><br>
Thought: "Okay, I have no idea how old the president is. In fact, I don't even know who the president is right now. Let's find that first."<br>
Action: <code>search_tool("Current President of France")</code><br>
Observation: "Emmanuel Macron."</p></li>
<li><p><strong>Iteration 2:</strong><br>
Thought: "Ok, we have the name: Macron. Now I need his age."<br>
Action: <code>search_tool("Emmanuel Macron age")</code><br>
Observation:** "47 years old."</p></li>
<li><p><strong>Iteration 3:</strong><br>
Thought: "I have the number 47. The user wanted the square root of this."<br>
Action: <code>calculator_tool("sqrt(47)")</code><br>
Observation: "6.8556..."</p></li>
<li><p><strong>Final Answer:</strong> "The square root of Emmanuel Macron's age (47) is approximately 6.85."</p></li>
</ul>

<p>See the magic? Step 3 was impossible without having discovered Step 2 first. The agent has been "connecting the dots."</p>

<h3>
  
  
  Now let's get to the code
</h3>

<p>We will use <strong>Agno</strong> to create our example of the ReAct pattern, implementing Think → Act → Observe → Repeat.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">os</span>
<span class="kn">import</span> <span class="n">sys</span>
<span class="kn">import</span> <span class="n">logging</span>
<span class="kn">import</span> <span class="n">traceback</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">List</span><span class="p">,</span> <span class="n">Optional</span>
<span class="kn">from</span> <span class="n">dotenv</span> <span class="kn">import</span> <span class="n">load_dotenv</span><span class="p">,</span> <span class="n">find_dotenv</span>
<span class="kn">from</span> <span class="n">agno.agent</span> <span class="kn">import</span> <span class="n">Agent</span>
<span class="kn">from</span> <span class="n">agno.models.openai</span> <span class="kn">import</span> <span class="n">OpenAIChat</span>
<span class="kn">from</span> <span class="n">agno.tools.tavily</span> <span class="kn">import</span> <span class="n">TavilyTools</span>

<span class="c1"># 1. Global Logging and Error Handling Configuration
</span><span class="n">LOG_DIR</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">dirname</span><span class="p">(</span><span class="n">__file__</span><span class="p">),</span> <span class="sh">"</span><span class="s">log</span><span class="sh">"</span><span class="p">)</span>
<span class="n">LOG_FILE</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">LOG_DIR</span><span class="p">,</span> <span class="sh">"</span><span class="s">logs.txt</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="ow">not</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">exists</span><span class="p">(</span><span class="n">LOG_DIR</span><span class="p">):</span>
    <span class="n">os</span><span class="p">.</span><span class="nf">makedirs</span><span class="p">(</span><span class="n">LOG_DIR</span><span class="p">)</span>

<span class="n">logging</span><span class="p">.</span><span class="nf">basicConfig</span><span class="p">(</span>
    <span class="n">level</span><span class="o">=</span><span class="n">logging</span><span class="p">.</span><span class="n">INFO</span><span class="p">,</span>
    <span class="nb">format</span><span class="o">=</span><span class="sh">"</span><span class="s">%(asctime)s [%(levelname)s] %(message)s</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">handlers</span><span class="o">=</span><span class="p">[</span>
        <span class="n">logging</span><span class="p">.</span><span class="nc">FileHandler</span><span class="p">(</span><span class="n">LOG_FILE</span><span class="p">,</span> <span class="n">encoding</span><span class="o">=</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">),</span>
        <span class="n">logging</span><span class="p">.</span><span class="nc">StreamHandler</span><span class="p">(</span><span class="n">sys</span><span class="p">.</span><span class="n">stdout</span><span class="p">)</span>
    <span class="p">]</span>
<span class="p">)</span>

<span class="n">logger</span> <span class="o">=</span> <span class="n">logging</span><span class="p">.</span><span class="nf">getLogger</span><span class="p">(</span><span class="n">__name__</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">global_exception_handler</span><span class="p">(</span><span class="n">exctype</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tb</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Captures unhandled exceptions and records them in the log.</span><span class="sh">"""</span>
    <span class="n">error_msg</span> <span class="o">=</span> <span class="sh">""</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">traceback</span><span class="p">.</span><span class="nf">format_exception</span><span class="p">(</span><span class="n">exctype</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tb</span><span class="p">))</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Unhandled exception:</span><span class="se">\n</span><span class="si">{</span><span class="n">error_msg</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">sys</span><span class="p">.</span><span class="nf">__excepthook__</span><span class="p">(</span><span class="n">exctype</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tb</span><span class="p">)</span>

<span class="n">sys</span><span class="p">.</span><span class="n">excepthook</span> <span class="o">=</span> <span class="n">global_exception_handler</span>

<span class="c1"># 2. Environment Variables Loading
</span><span class="n">env_path</span> <span class="o">=</span> <span class="nf">find_dotenv</span><span class="p">()</span>
<span class="k">if</span> <span class="n">env_path</span><span class="p">:</span>
    <span class="nf">load_dotenv</span><span class="p">(</span><span class="n">env_path</span><span class="p">)</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">.env file loaded from: </span><span class="si">{</span><span class="n">env_path</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="k">else</span><span class="p">:</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">warning</span><span class="p">(</span><span class="sh">"</span><span class="s">.env file not found</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># 3. Tool Definitions
</span><span class="k">def</span> <span class="nf">calculate</span><span class="p">(</span><span class="n">expression</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">
    Solves a simple mathematical expression (addition, subtraction, multiplication, division).
    Useful for calculating year or date differences.

    Args:
        expression (str): The mathematical expression to evaluate (e.g., </span><span class="sh">"</span><span class="s">2024 - 1789</span><span class="sh">"</span><span class="s">).
    </span><span class="sh">"""</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="c1"># Allow only safe characters for basic eval
</span>        <span class="n">allowed_chars</span> <span class="o">=</span> <span class="sh">"</span><span class="s">0123456789+-*/(). </span><span class="sh">"</span>
        <span class="k">if</span> <span class="nf">all</span><span class="p">(</span><span class="n">c</span> <span class="ow">in</span> <span class="n">allowed_chars</span> <span class="k">for</span> <span class="n">c</span> <span class="ow">in</span> <span class="n">expression</span><span class="p">):</span>
            <span class="n">result</span> <span class="o">=</span> <span class="nf">eval</span><span class="p">(</span><span class="n">expression</span><span class="p">)</span>
            <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Result: </span><span class="si">{</span><span class="n">result</span><span class="si">}</span><span class="sh">"</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">return</span> <span class="sh">"</span><span class="s">Error: Disallowed characters in expression.</span><span class="sh">"</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Error while calculating: </span><span class="si">{</span><span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span>

<span class="c1"># 4. Agno Agent Configuration (ReAct Pattern)
</span><span class="n">model_id</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">BASE_MODEL</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">gpt-4o</span><span class="sh">"</span><span class="p">)</span>

<span class="n">agent</span> <span class="o">=</span> <span class="nc">Agent</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="nc">OpenAIChat</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="n">model_id</span><span class="p">),</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[</span><span class="nc">TavilyTools</span><span class="p">(),</span> <span class="n">calculate</span><span class="p">],</span>
    <span class="n">instructions</span><span class="o">=</span><span class="p">[</span>
        <span class="sh">"</span><span class="s">You are a researcher using the ReAct (Reason + Act) method.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">1. Think step-by-step about what information you need to answer the user</span><span class="sh">'</span><span class="s">s question.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">2. Use the search tool (Tavily) to find specific dates, facts, or data.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">3. Use the calculator (</span><span class="sh">'</span><span class="s">calculate</span><span class="sh">'</span><span class="s">) for any mathematical operation or time calculation.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">4. Do not guess historical information. If you don</span><span class="sh">'</span><span class="s">t have a piece of data, look it up.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">5. Show your reasoning clearly: </span><span class="sh">'</span><span class="s">Thought:</span><span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="s">Action:</span><span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="s">Observation:</span><span class="sh">'</span><span class="s">.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">6. Continue investigating until you have a complete and verified answer.</span><span class="sh">"</span>
    <span class="p">],</span>
<span class="p">)</span>

<span class="c1"># 5. User Interface
</span><span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">Starting Historical Detective Agent (ReAct)...</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">--- Historical Detective - ReAct Pattern ---</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Type </span><span class="sh">'</span><span class="s">exit</span><span class="sh">'</span><span class="s"> to quit.</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">user_input</span> <span class="o">=</span> <span class="nf">input</span><span class="p">(</span><span class="sh">"</span><span class="s">Researcher, what is your question?: </span><span class="sh">"</span><span class="p">)</span>

            <span class="k">if</span> <span class="n">user_input</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span> <span class="o">==</span> <span class="sh">"</span><span class="s">exit</span><span class="sh">"</span><span class="p">:</span>
                <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">The user has ended the session.</span><span class="sh">"</span><span class="p">)</span>
                <span class="k">break</span>

            <span class="k">if</span> <span class="ow">not</span> <span class="n">user_input</span><span class="p">.</span><span class="nf">strip</span><span class="p">():</span>
                <span class="k">continue</span>

            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">User query: </span><span class="si">{</span><span class="n">user_input</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Investigating...</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>
            <span class="n">agent</span><span class="p">.</span><span class="nf">print_response</span><span class="p">(</span><span class="n">user_input</span><span class="p">,</span> <span class="n">stream</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">show_tool_calls</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">except</span> <span class="nb">KeyboardInterrupt</span><span class="p">:</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">Keyboard interrupt detected.</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">break</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Error in main loop: </span><span class="si">{</span><span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">An error occurred: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>

</code></pre>

</div>



<p><strong>Where is ReAct in the code?</strong></p>

<ul>
<li><p><strong>THINK:</strong> It is defined in the agent's <code>instructions</code>.<br>
Code: <code>instructions=["You are a researcher using the ReAct method...", "Think step-by-step...", "Show your reasoning..."]</code>.<br>
This forces the LLM to generate a "Thought Trace" (internal monologue) before doing anything.</p></li>
<li><p><strong>ACT:</strong> The agent has external capabilities defined in <code>tools</code>.<br>
Code: <code>tools=[TavilyTools(), calculate]</code>.<br>
If it needs data, it uses Tavily; if it needs math, it uses calculate.</p></li>
<li><p><strong>OBSERVE:</strong> The framework (Agno) executes the tool and returns the result to the agent.<br>
Code: Occurs automatically within the <code>Agent</code> class. The agent "reads" the return from <code>calculate</code> (e.g., "Result: 6.85").</p></li>
<li><p><strong>REPEAT:</strong> The loop continues until the agent has enough information.<br>
Code: <code>agent.print_response(..., show_tool_calls=True)</code>.<br>
The <code>show_tool_calls=True</code> parameter is vital: it allows you to see in the console how the agent "fires" actions in real-time.</p></li>
</ul>

<h3>
  
  
  Why do we like this pattern so much? (Pros)
</h3>

<ul>
<li>
<strong>Solving Multi-hop Problems:</strong> It's the ability to answer questions where A leads you to B, and B leads you to C.</li>
<li>
<strong>Self-Healing Capability:</strong> This is vital. Imagine it searches for "Macron's Age" and Google fails. A normal script would crash. A ReAct agent thinks: <em>"Wow, the search failed. Well, I'll search for his date of birth and calculate the age myself."</em>
</li>
<li>
<strong>Goodbye Black Box:</strong> As a developer, you can read the <code>Thoughts</code>. You know exactly <em>why</em> the agent decided to use the calculator and not something else.</li>
<li>
<strong>Fewer Hallucinations:</strong> By forcing it to base every step on a real observation ("Observation"), it is much harder for it to invent data.</li>
</ul>

<h3>
  
  
  It's not all roses (Cons)
</h3>

<ul>
<li>
<strong>It's Slow (Latency):</strong> ReAct is sequential. Thinking 3 times means calling the LLM 3 times + executing tools. Be prepared for waits of 10 to 30 seconds.</li>
<li>
<strong>The API Bill (Cost):</strong> That "internal monologue" consumes tokens like there's no tomorrow. Your context history fills up very quickly.</li>
<li>
<strong>The Risk of Loops:</strong> Sometimes they get obsessed. <em>"I search for X → It's not there → I think I should search for X again → I search for X..."</em>
</li>
<li>
<strong>Delicate Prompting:</strong> You need a very well-tuned System Prompt so the model knows when to stop thinking and give you the answer.</li>
</ul>

<h3>
  
  
  Tips from the Trenches (Engineering)
</h3>

<p>If you are going to put this into production (using Agno, LangChain, or whatever), burn this into your brain:</p>

<ol>
<li>
<strong>Put a Brake on it (Max Iterations):</strong> Always configure an iteration limit (e.g., 10 steps). If it hasn't solved the problem in 10 steps, it won't solve it in 100 and it's just burning your money.</li>
<li>
<strong>Teach it when to stop (Stop Sequences):</strong> Technically, the LLM must stop writing right after launching the <strong>Action</strong>. If you don't cut it off there, it will start inventing the <strong>Observation</strong> itself (hallucinating the tool result). Modern frameworks usually handle this, but keep an eye on it.</li>
<li>
<strong>Clean up the trash (Context Management):</strong> In long chats, delete old thought traces and keep only the final answer. Otherwise, the agent will run out of "RAM" (context window) immediately.</li>
</ol>

<h3>
  
  
  Where is this used in the real world?
</h3>

<ul>
<li>
<strong>Coding Agents (like Devin or Copilot):</strong> The flow is: "I write code → Execute → See error → Think how to fix it → Rewrite".</li>
<li>
<strong>Level 2 Tech Support:</strong> "Read ticket → Check server status → Check user logs → Cross-reference data → Answer".</li>
<li>
<strong>Financial Analysts:</strong> "Search current price → Search breaking news → Compare with history → Generate recommendation".</li>
</ul>

<p>Happy Coding! 🤖</p>

