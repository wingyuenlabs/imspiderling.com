---
Title: A Movie Finder with AI Reflexion using GoLang
Description: 
Author: erlangb
Date: 2026-03-11T21:34:36.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction: The "Vibes-Based" Engineering Trap
</h2>

<p>We’ve all been there. You ask an LLM for "underground 80s sci-fi," and it starts strong with Blade Runner (hardly underground). Then, desperate to please, it hallucinations: "Have you seen Neon Shadows (1984)?" It sounds perfect. It sounds real. It doesn’t exist.</p>

<p>In a side project, that’s a "lol." In production, that’s a total failure.</p>

<h2>
  
  
  The Problem: The Confidence Gap
</h2>

<p>LLMs aren't stupid; they are just pathologically helpful. They prioritize being pleasant over being factual because they lack a Skepticism Layer. Most developers fall into the trap of Linear Prompting:</p>

<p>Send request.</p>

<p>Hope for the best.</p>

<p>But hope is not an engineering strategy. To build reliable Agentic AI, we need to move from "sending prompts" to "building pipelines that verify."</p>

<h3>
  
  
  The Solution: Reflexion and Orchestration
</h3>

<p>To solve this for my "Movie Finder" use case, I didn't just write a better prompt. I implemented the <strong>Reflexion Pattern</strong>: an architectural loop where one agent's output is treated as a "draft" that must survive a rigorous audit by a second, skeptical agent.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzva7oq4rmg8mkq8jsbyh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzva7oq4rmg8mkq8jsbyh.png" alt=" " width="800" height="246"></a></p>

<p>To bridge this gap, I used the <strong>EINO framework</strong>. EINO (pronounced 'ay-no') is a Go-native orchestration framework designed specifically for LLM workflows. It allows you to model complex agentic logic as a graph of nodes, which was perfect for implementing the <strong>Reflexion Pattern</strong>.</p>

<p>🛠️ Open Source &amp; Local Setup<br>
The full source code for this project is available on GitHub. To visualize and monitor the agent's reasoning steps, I used two libraries I developed:</p>

<ul>
<li>
<a href="https://github.com/erlangb/agent_monitor" rel="noopener noreferrer">agent_monitor</a>: The core Go project to run usecases</li>
<li>
<a href="https://github.com/erlangb/agentmeter" rel="noopener noreferrer">agentmeter</a>: a library for capturing and printing agent internals.</li>
</ul>

<blockquote>
<p>"Why did I build an AI agent in <strong>Go</strong> instead of Python?<br>
The honest answer: <strong>It's what I know.</strong> But beyond familiarity, I wanted to explore the current 'state of the art' for Agentic AI in the Go ecosystem.</p>
</blockquote>

<p>In this article, I’ll show you how I moved beyond simple prompts to a dual-agent system. By pitting a Cinephile against a Clerk, I’ve built an adversarial loop where agents "argue" their way toward grounded truth.</p>

<p>It’s not just about getting an answer; it’s about building a system that uses systematic skepticism to virtually eliminate hallucinations.</p>
<h2>
  
  
  The Concept: What is the Reflexion Pattern?
</h2>

<p>At its core, the <strong>Reflexion Pattern</strong> is a design pattern for LLM agents that introduces a "self-correction" loop.<br>
Think of a standard AI agent as a solo freelancer working without an editor. They produce work, and you get what you get. The Reflexion Pattern turns that solo freelancer into a <strong>team of two</strong>: one who creates, and one who audits.</p>
<h3>
  
  
  How it works (The 3-Step Dance)
</h3>

<p>In my movie finder, the loop follows a specific cycle of <strong>Generation</strong>, <strong>Critique</strong>, and <strong>Correction</strong>:</p>

<ol>
<li>
<strong>Generation (The Draft):</strong> The first agent (The Cinephile) receives the user's request and generates a response. It operates purely on its internal training data—which, as we know, can be prone to "stochastic dreaming" (hallucinations).</li>
<li>
<strong>Critique (The Fact-Check):</strong> Instead of showing the user the result, the output is passed to the second agent (The Clerk). This agent is given a specific "Skeptic" persona and, crucially, access to <strong>External Tools</strong> (in this case, the <strong>Tavily Search API</strong>). Its only job is to find reasons why the first agent might be wrong.</li>
<li>
<strong>Correction (The Iteration):</strong> If the Clerk finds an error, it doesn't just fail the process. It generates a <strong>feedback signal</strong>—a structured message explaining <em>what</em> was wrong and <em>why</em>. This feedback is fed back into the first agent, which now has a "second chance" to get it right.</li>
</ol>
<h2>
  
  
  The Architecture: Mapping the Graph
</h2>

<p>To implement the Reflexion pattern in Go, I used EINO's Graph composition. This allows us to treat our agents as independent nodes connected by edges, including a conditional "branch" that creates our self-correction loop.</p>
<h3>
  
  
  The Pipeline Logic
</h3>

<p>Here is the simplified implementation of the FindMoviesPipeline. Notice how the "Loop" isn't a complex for loop in the code, but a visual branch in the graph:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="n">NewFindMoviesPipeline</span><span class="p">(</span>
    <span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span>
    <span class="n">cinephile</span> <span class="o">*</span><span class="n">CinephileAgent</span><span class="p">,</span>
    <span class="n">clerk</span> <span class="o">*</span><span class="n">ClerkAgent</span><span class="p">,</span>
    <span class="n">curator</span> <span class="o">*</span><span class="n">CuratorChain</span><span class="p">,</span>
<span class="p">)</span> <span class="p">(</span><span class="n">FindMoviesPipeline</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="c">// 1. Initialize the Graph with a shared State</span>
    <span class="n">g</span> <span class="o">:=</span> <span class="n">compose</span><span class="o">.</span><span class="n">NewGraph</span><span class="p">[</span><span class="o">*</span><span class="n">appmodel</span><span class="o">.</span><span class="n">FindMoviesState</span><span class="p">,</span> <span class="o">*</span><span class="n">appmodel</span><span class="o">.</span><span class="n">FindMoviesState</span><span class="p">]()</span>

    <span class="c">// 2. Add the Agent Nodes</span>
    <span class="n">g</span><span class="o">.</span><span class="n">AddLambdaNode</span><span class="p">(</span><span class="s">"cinephile"</span><span class="p">,</span> <span class="n">compose</span><span class="o">.</span><span class="n">InvokableLambda</span><span class="p">(</span><span class="n">cinephile</span><span class="o">.</span><span class="n">Invoke</span><span class="p">))</span>
    <span class="n">g</span><span class="o">.</span><span class="n">AddLambdaNode</span><span class="p">(</span><span class="s">"clerk"</span><span class="p">,</span> <span class="n">compose</span><span class="o">.</span><span class="n">InvokableLambda</span><span class="p">(</span><span class="n">clerk</span><span class="o">.</span><span class="n">Invoke</span><span class="p">))</span>
    <span class="n">g</span><span class="o">.</span><span class="n">AddLambdaNode</span><span class="p">(</span><span class="s">"curator"</span><span class="p">,</span> <span class="n">compose</span><span class="o">.</span><span class="n">InvokableLambda</span><span class="p">(</span><span class="n">curator</span><span class="o">.</span><span class="n">Invoke</span><span class="p">))</span>

    <span class="c">// 3. Define the linear flow</span>
    <span class="n">g</span><span class="o">.</span><span class="n">AddEdge</span><span class="p">(</span><span class="n">compose</span><span class="o">.</span><span class="n">START</span><span class="p">,</span> <span class="s">"cinephile"</span><span class="p">)</span>
    <span class="n">g</span><span class="o">.</span><span class="n">AddEdge</span><span class="p">(</span><span class="s">"cinephile"</span><span class="p">,</span> <span class="s">"clerk"</span><span class="p">)</span>

    <span class="c">// 4. The Reflexion Branch: The "Skeptic" decides where to go next</span>
    <span class="n">branch</span> <span class="o">:=</span> <span class="n">compose</span><span class="o">.</span><span class="n">NewGraphBranch</span><span class="p">(</span>
       <span class="k">func</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">state</span> <span class="o">*</span><span class="n">appmodel</span><span class="o">.</span><span class="n">FindMoviesState</span><span class="p">)</span> <span class="p">(</span><span class="kt">string</span><span class="p">,</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
          <span class="c">// If the Clerk is happy OR we've tried too many times, move to curation</span>
          <span class="k">if</span> <span class="n">state</span><span class="o">.</span><span class="n">IsSatisfied</span> <span class="o">||</span> <span class="n">state</span><span class="o">.</span><span class="n">RetryCount</span> <span class="o">&gt;=</span> <span class="n">state</span><span class="o">.</span><span class="n">MaxRetries</span> <span class="p">{</span>
             <span class="k">return</span> <span class="s">"curator"</span><span class="p">,</span> <span class="no">nil</span>
          <span class="p">}</span>
          <span class="c">// Otherwise, send it back to the Cinephile for correction</span>
          <span class="k">return</span> <span class="s">"cinephile"</span><span class="p">,</span> <span class="no">nil</span>
       <span class="p">},</span>
       <span class="k">map</span><span class="p">[</span><span class="kt">string</span><span class="p">]</span><span class="kt">bool</span><span class="p">{</span><span class="s">"curator"</span><span class="o">:</span> <span class="no">true</span><span class="p">,</span> <span class="s">"cinephile"</span><span class="o">:</span> <span class="no">true</span><span class="p">},</span>
    <span class="p">)</span>

    <span class="n">g</span><span class="o">.</span><span class="n">AddBranch</span><span class="p">(</span><span class="s">"clerk"</span><span class="p">,</span> <span class="n">branch</span><span class="p">)</span>
    <span class="n">g</span><span class="o">.</span><span class="n">AddEdge</span><span class="p">(</span><span class="s">"curator"</span><span class="p">,</span> <span class="n">compose</span><span class="o">.</span><span class="n">END</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">g</span><span class="o">.</span><span class="n">Compile</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="n">compose</span><span class="o">.</span><span class="n">WithGraphName</span><span class="p">(</span><span class="s">"find_movies_graph"</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Why this code matters:
</h3>

<ul>
<li>The State Object: The FindMoviesState acts as the "Short-term Memory." It carries the current list of movies, the Clerk's critiques, and the RetryCount.</li>
<li>Decoupled Logic: The cinephile doesn't know the clerk exists. It just knows it receives a state and returns a state. This makes <strong>testing individual agents</strong> much easier.</li>
<li>The Branch is the Brain: The NewGraphBranch function is where the Reflexion happens. It forces the system to be honest. If state.IsSatisfied is false, the data is physically impossible to reach the curator node.</li>
</ul>

<h2>
  
  
  The Architecture: Inside the FindMoviesUseCase
</h2>

<p>In my FindMoviesUseCase, the data moves through four distinct stages.</p>

<h3>
  
  
  The Node Breakdown
</h3>

<p><strong>1. The Refiner (The Translator)</strong><br>
Before any "thinking" happens, we need structure. The <strong>RefinerChain</strong> takes the user's messy, natural language input—<em>"I want some weird 70s space movies that feel like David Bowie's music"</em>—and converts it into a clean Go struct.</p>

<ul>
<li>
<strong>Output:</strong> Structured parameters like <em>primary_genre</em>, <em>secondary_genres</em>, <em>end_year</em>, <em>start_year</em>
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User Input: &gt; "I want some weird 70s space movies that feel like David Bowie's music"

RefinerChain Output:
</code></pre>

</div>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
   </span><span class="nl">"primary_genre"</span><span class="p">:</span><span class="w"> </span><span class="s2">"science fiction"</span><span class="p">,</span><span class="w">
   </span><span class="nl">"secondary_genres"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"weird"</span><span class="p">,</span><span class="w"> </span><span class="s2">"space"</span><span class="p">,</span><span class="w"> </span><span class="s2">"musical vibe"</span><span class="p">],</span><span class="w">
   </span><span class="nl">"start_year"</span><span class="p">:</span><span class="w"> </span><span class="mi">1970</span><span class="p">,</span><span class="w">
   </span><span class="nl">"end_year"</span><span class="p">:</span><span class="w"> </span><span class="mi">1979</span><span class="p">,</span><span class="w">
   </span><span class="nl">"is_classic"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
   </span><span class="nl">"original_text"</span><span class="p">:</span><span class="w"> </span><span class="s2">"I want some weird 70s space movies that feel like David Bowie's music"</span><span class="p">,</span><span class="w">
   </span><span class="nl">"query_info"</span><span class="p">:</span><span class="w"> </span><span class="s2">"weird space science fiction 1970s Bowie vibe"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>


<p><strong>2. The Cinephile (The Creative Brain)</strong><br>
This is our primary Generator. It uses the refined parameters to search its internal knowledge and propose a curated list of films.</p>

<ul>
<li>
<strong>The Risk:</strong> This is where hallucinations live. If the LLM "remembers" a movie that doesn't exist, it will confidently include it here.</li>
</ul>

<p><strong>3. The Clerk (The Auditor &amp; Tool User)</strong><br>
This is the heart of the <strong>Reflexion Loop</strong>. The Clerk is a "Skeptic" node equipped with the <strong>Tavily Search tool</strong>.</p>

<ul>
<li>
<strong>The Process:</strong> It takes every movie from the Cinephile's list and verifies it against the real world.</li>
<li>
<strong>The Decision:</strong> If the movies match the user query is set satisfied true, otherwise it returns the list of critiques for every movie found. If one or more movies have passed the critique, it the Cinephile will keep them, and add more movies to be verified by the clerk.</li>
</ul>

<p><strong>4. The Curator (The Final Editor)</strong><br>
Once the loop is broken (either through success or reaching the MaxRetries limit), the data hits the <strong>CuratorChain</strong>.</p>

<ul>
<li>It performs a final prune. It read the last clerk response and movie list and finalize the result to the end user.</li>
</ul>
<h2>
  
  
  The Loop in Action: A Real-World "Argument"
</h2>

<p>To see the value of the Reflexion pattern, we have to look at how the agents interact when things go wrong. In this example, I asked the system for:<br>
<em>"Italian movies from the late 90s about the new millennium."</em></p>
<h3>
  
  
  Round 1: The Cinephile's "Stochastic Dreaming"
</h3>

<p>The <strong>CinephileAgent</strong> generated three suggestions. They looked plausible, but there were hidden hallucinations:</p>

<ol>
<li>
<em>Ecco fatto</em> (1998)</li>
<li>
<em>Tutto l'amore che c'è</em> (1999)</li>
<li>
<em>Luna e l'altra</em> (1996)</li>
</ol>
<h3>
  
  
  Round 2: The Clerk's Skepticism (Tavily Search)
</h3>

<p>The <strong>ClerkAgent</strong> immediately triggered a series of parallel <strong>Tavily searches</strong> to verify these titles.<br>
It didn't just check if the movies existed; it checked the metadata against the user's specific constraints (Year: 1997–1999). Here is the "Correction Note" it generated:<br>
<strong>Clerk:</strong> <em>"isSatisfied: false. 'Tutto l'amore che c'è' is actually a 2000 film, not 1999. 'Luna e l'altra' is from 1996, which is outside the requested range. Replace these."</em></p>
<h3>
  
  
  Round 3: The Correction
</h3>

<p>The graph routed the state back to the <strong>Cinephile</strong>. Crucially, the Cinephile was "aware" of its previous mistakes because they were stored in the shared FindMoviesState.<br>
<strong>Cinephile:</strong> <em>"Previous critiques to fix: Replace Tutto l'amore... and Luna e l'altra."</em> It kept <em>Ecco fatto</em> (which passed) and proposed new candidates: <em>Cose che non ti ho mai detto</em> and <em>I piccoli maestri</em>.</p>
<h3>
  
  
  Round 4: Deep Verification
</h3>

<p>The Clerk is a tough critic. It rejected the new suggestions too, but for deeper reasons:</p>

<ul>
<li>
<strong>National Identity:</strong> It caught that <em>Cose che non ti ho mai detto</em> is actually a Spanish-American film, not Italian.</li>
<li>
<strong>Thematic Alignment:</strong> It caught that <em>I piccoli maestri</em> is a WWII resistance film—technically Italian and from 1998, but it has <strong>nothing</strong> to do with the "new millennium" theme.</li>
</ul>
<h3>
  
  
  The Final Result: Deterministic Success
</h3>

<p>Finally, the loop closed on a verified, accurate list:</p>

<ol>
<li>
<strong>Ecco fatto</strong> (1998)</li>
<li>
<strong>Tutti giù per terra</strong> (1997)</li>
</ol>

<p>Without the Reflexion loop, the user would have received a list where 66% of the data was technically wrong.</p>

<p>Following a slider where you can see all the output.</p>

<p><iframe height="600" src="https://codepen.io/daniele-dangeli/embed/VYKmVRp?height=600&amp;default-tab=result&amp;embed-version=2">
</iframe>
</p>

<h2>
  
  
  Conclusion: From "Stochastic" to "Deterministic"
</h2>

<p>LLMs are "stochastic" (probabilistic) by nature. They are built to predict the next word, not to tell the truth. By implementing the <strong>Reflexion Pattern</strong>, we transform that probability into a more "deterministic" system. If the Clerk doesn't find a factual match on the web, the data simply does not pass.</p>

<h2>
  
  
  Verification is the New Optimization
</h2>

<p>Instead of spending weeks fine-tuning a model or "begging" a prompt to be accurate, we can achieve better results by giving agents <strong>tools</strong> (like Tavily) and <strong>feedback</strong>. The "Cinephile vs. Clerk" interaction proves that two specialized agents working in a loop will always outperform a single "Generalist" agent trying to do everything at once.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>Building this wasn't just about finding niche Italian movies; it was about exploring how we can trust the software we build in the age of AI. If you are a Go developer, don't wait for a "Python-equivalent" to emerge. The tools are already here.<br>
The next time your LLM hallucinates, don't just change the prompt. <strong>Change the architecture.</strong></p>

