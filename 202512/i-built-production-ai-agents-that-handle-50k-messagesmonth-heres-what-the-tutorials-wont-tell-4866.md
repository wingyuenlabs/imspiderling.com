---
Title: I Built Production AI Agents That Handle 50K Messages/Month - Here's What the Tutorials Won't Tell You
Description: 
Author: Richard Sakaguchi
Date: 2025-12-15T21:54:46.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1677442136019-21780ecad995%3Fw%3D1200%26q%3D80" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1677442136019-21780ecad995%3Fw%3D1200%26q%3D80" alt="AI Agent Dashboard" width="1200" height="675"></a></p>

<p>Three months ago, I deployed an AI agent to production. Today, it handles 50,000+ messages monthly with zero downtime. But here's the thing - <strong>none of the tutorials prepared me for what actually happened</strong>.</p>

<p>Everyone shows you the shiny "hello world" chatbot. Nobody shows you what happens when real users spam your API at 3 AM, or when your LLM decides to hallucinate customer data.</p>

<p>This is that story.</p>

<h2>
  
  
  The Promise vs. The Reality
</h2>

<p><strong>What tutorials show you:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># The "perfect" AI agent
</span><span class="n">agent</span> <span class="o">=</span> <span class="nc">AIAgent</span><span class="p">(</span><span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gpt-4</span><span class="sh">"</span><span class="p">)</span>
<span class="n">response</span> <span class="o">=</span> <span class="n">agent</span><span class="p">.</span><span class="nf">chat</span><span class="p">(</span><span class="sh">"</span><span class="s">Hello!</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">response</span><span class="p">)</span>  <span class="c1"># Magic! ✨
</span></code></pre>

</div>



<p><strong>What production looks like:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1558494949-ef010cbdcc31%3Fw%3D1200%26q%3D80" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1558494949-ef010cbdcc31%3Fw%3D1200%26q%3D80" alt="System Architecture" width="1200" height="673"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>graph TB
    A[User Message] --&gt; B{Rate Limiter}
    B --&gt;|Allowed| C[Queue System]
    B --&gt;|Blocked| D[429 Response]
    C --&gt; E{Health Check}
    E --&gt;|Healthy| F[AI Agent]
    E --&gt;|Degraded| G[Fallback Handler]
    F --&gt; H{Response Validator}
    H --&gt;|Valid| I[User]
    H --&gt;|Hallucination| J[Retry Logic]
    G --&gt; I
    J --&gt; F
</code></pre>

</div>



<p>Notice the difference? <strong>Production AI agents need six layers of protection that tutorials never mention.</strong></p>

<h2>
  
  
  The Five Hard Truths About Production AI Agents
</h2>

<h3>
  
  
  1. Rate Limiting Isn't Optional - It's Survival
</h3>

<p><strong>The tutorial way:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># YOLO approach
</span><span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
    <span class="n">message</span> <span class="o">=</span> <span class="nf">get_message</span><span class="p">()</span>
    <span class="n">response</span> <span class="o">=</span> <span class="n">ai_agent</span><span class="p">.</span><span class="nf">process</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>The production way:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">collections</span> <span class="kn">import</span> <span class="n">defaultdict</span>
<span class="kn">from</span> <span class="n">datetime</span> <span class="kn">import</span> <span class="n">datetime</span><span class="p">,</span> <span class="n">timedelta</span>

<span class="k">class</span> <span class="nc">AdaptiveRateLimiter</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">base_limit</span><span class="o">=</span><span class="mi">100</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">limits</span> <span class="o">=</span> <span class="nf">defaultdict</span><span class="p">(</span><span class="k">lambda</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="sh">"</span><span class="s">reset</span><span class="sh">"</span><span class="p">:</span> <span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">()})</span>
        <span class="n">self</span><span class="p">.</span><span class="n">base_limit</span> <span class="o">=</span> <span class="n">base_limit</span>

    <span class="k">def</span> <span class="nf">check_limit</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">user_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">risk_score</span><span class="p">:</span> <span class="nb">float</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">bool</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Adaptive rate limiting based on user behavior</span><span class="sh">"""</span>
        <span class="n">limit_data</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">limits</span><span class="p">[</span><span class="n">user_id</span><span class="p">]</span>

        <span class="c1"># Reset window
</span>        <span class="k">if</span> <span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">&gt;</span> <span class="n">limit_data</span><span class="p">[</span><span class="sh">"</span><span class="s">reset</span><span class="sh">"</span><span class="p">]:</span>
            <span class="n">limit_data</span><span class="p">[</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span>
            <span class="n">limit_data</span><span class="p">[</span><span class="sh">"</span><span class="s">reset</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">+</span> <span class="nf">timedelta</span><span class="p">(</span><span class="n">hours</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>

        <span class="c1"># Adjust limit based on risk
</span>        <span class="n">adjusted_limit</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">base_limit</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="n">risk_score</span><span class="p">))</span>

        <span class="k">if</span> <span class="n">limit_data</span><span class="p">[</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">]</span> <span class="o">&gt;=</span> <span class="n">adjusted_limit</span><span class="p">:</span>
            <span class="k">return</span> <span class="bp">False</span>

        <span class="n">limit_data</span><span class="p">[</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">]</span> <span class="o">+=</span> <span class="mi">1</span>
        <span class="k">return</span> <span class="bp">True</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1551288049-bebda4e38f71%3Fw%3D1200%26q%3D80" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1551288049-bebda4e38f71%3Fw%3D1200%26q%3D80" alt="Rate Limiting Dashboard" width="1200" height="800"></a></p>

<p><strong>Why it matters:</strong> In month one, I blocked <strong>2,847 abuse attempts</strong>. Without rate limiting, that's $500+ in wasted API calls.</p>

<h3>
  
  
  2. LLMs Hallucinate - Always Validate Output
</h3>

<p>This one hurt. A user asked for their account balance. The AI agent confidently responded: <strong>"Your balance is $127,549.32"</strong></p>

<p>Actual balance? <strong>$47.15</strong></p>

<p><strong>The fix:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">re</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Optional</span>

<span class="k">class</span> <span class="nc">ResponseValidator</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="c1"># Patterns that should NEVER appear in responses
</span>        <span class="n">self</span><span class="p">.</span><span class="n">forbidden_patterns</span> <span class="o">=</span> <span class="p">[</span>
            <span class="sa">r</span><span class="sh">'</span><span class="s">\$[\d,]+\.\d{2}</span><span class="sh">'</span><span class="p">,</span>  <span class="c1"># Dollar amounts
</span>            <span class="sa">r</span><span class="sh">'</span><span class="s">\b\d{3}-\d{2}-\d{4}\b</span><span class="sh">'</span><span class="p">,</span>  <span class="c1"># SSN
</span>            <span class="sa">r</span><span class="sh">'</span><span class="s">\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b</span><span class="sh">'</span><span class="p">,</span>  <span class="c1"># Emails
</span>        <span class="p">]</span>

    <span class="k">def</span> <span class="nf">validate</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">response</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">user_context</span><span class="p">:</span> <span class="nb">dict</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Optional</span><span class="p">[</span><span class="nb">str</span><span class="p">]:</span>
        <span class="sh">"""</span><span class="s">Validate AI response against business rules</span><span class="sh">"""</span>

        <span class="c1"># Check for hallucinated data
</span>        <span class="k">for</span> <span class="n">pattern</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">forbidden_patterns</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">re</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span><span class="n">pattern</span><span class="p">,</span> <span class="n">response</span><span class="p">,</span> <span class="n">re</span><span class="p">.</span><span class="n">IGNORECASE</span><span class="p">):</span>
                <span class="k">return</span> <span class="bp">None</span>  <span class="c1"># Reject response
</span>
        <span class="c1"># Verify facts against database
</span>        <span class="k">if</span> <span class="sh">"</span><span class="s">balance</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="nf">lower</span><span class="p">():</span>
            <span class="n">claimed_balance</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">extract_balance</span><span class="p">(</span><span class="n">response</span><span class="p">)</span>
            <span class="n">actual_balance</span> <span class="o">=</span> <span class="n">user_context</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">balance</span><span class="sh">"</span><span class="p">)</span>

            <span class="k">if</span> <span class="n">claimed_balance</span> <span class="ow">and</span> <span class="nf">abs</span><span class="p">(</span><span class="n">claimed_balance</span> <span class="o">-</span> <span class="n">actual_balance</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mf">0.01</span><span class="p">:</span>
                <span class="k">return</span> <span class="bp">None</span>  <span class="c1"># Hallucination detected
</span>
        <span class="k">return</span> <span class="n">response</span>
</code></pre>

</div>



<p><strong>Result:</strong> Zero incidents of hallucinated financial data in production.</p>

<h3>
  
  
  3. Context Window Management Is an Art
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1551033406-611cf9a28f67%3Fw%3D1200%26q%3D80" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1551033406-611cf9a28f67%3Fw%3D1200%26q%3D80" alt="Context Window Visualization" width="1200" height="1800"></a></p>

<p>Here's what nobody tells you: <strong>managing conversation context at scale is harder than building the agent itself</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">collections</span> <span class="kn">import</span> <span class="n">deque</span>
<span class="kn">from</span> <span class="n">dataclasses</span> <span class="kn">import</span> <span class="n">dataclass</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">List</span>

<span class="nd">@dataclass</span>
<span class="k">class</span> <span class="nc">Message</span><span class="p">:</span>
    <span class="n">role</span><span class="p">:</span> <span class="nb">str</span>
    <span class="n">content</span><span class="p">:</span> <span class="nb">str</span>
    <span class="n">tokens</span><span class="p">:</span> <span class="nb">int</span>
    <span class="n">importance</span><span class="p">:</span> <span class="nb">float</span>  <span class="c1"># 0-1 score
</span>
<span class="k">class</span> <span class="nc">SmartContextManager</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">max_tokens</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">4000</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">max_tokens</span> <span class="o">=</span> <span class="n">max_tokens</span>
        <span class="n">self</span><span class="p">.</span><span class="n">messages</span> <span class="o">=</span> <span class="nf">deque</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">add_message</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">message</span><span class="p">:</span> <span class="n">Message</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">_trim_context</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">_trim_context</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Keep most important messages within token limit</span><span class="sh">"""</span>
        <span class="n">total_tokens</span> <span class="o">=</span> <span class="nf">sum</span><span class="p">(</span><span class="n">m</span><span class="p">.</span><span class="n">tokens</span> <span class="k">for</span> <span class="n">m</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">messages</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">total_tokens</span> <span class="o">&lt;=</span> <span class="n">self</span><span class="p">.</span><span class="n">max_tokens</span><span class="p">:</span>
            <span class="k">return</span>

        <span class="c1"># Sort by importance, keep system prompts
</span>        <span class="n">sorted_msgs</span> <span class="o">=</span> <span class="nf">sorted</span><span class="p">(</span>
            <span class="p">[</span><span class="n">m</span> <span class="k">for</span> <span class="n">m</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">messages</span> <span class="k">if</span> <span class="n">m</span><span class="p">.</span><span class="n">role</span> <span class="o">!=</span> <span class="sh">"</span><span class="s">system</span><span class="sh">"</span><span class="p">],</span>
            <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="p">.</span><span class="n">importance</span>
        <span class="p">)</span>

        <span class="c1"># Remove least important until we fit
</span>        <span class="k">while</span> <span class="n">total_tokens</span> <span class="o">&gt;</span> <span class="n">self</span><span class="p">.</span><span class="n">max_tokens</span> <span class="ow">and</span> <span class="n">sorted_msgs</span><span class="p">:</span>
            <span class="n">removed</span> <span class="o">=</span> <span class="n">sorted_msgs</span><span class="p">.</span><span class="nf">pop</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
            <span class="n">self</span><span class="p">.</span><span class="n">messages</span><span class="p">.</span><span class="nf">remove</span><span class="p">(</span><span class="n">removed</span><span class="p">)</span>
            <span class="n">total_tokens</span> <span class="o">-=</span> <span class="n">removed</span><span class="p">.</span><span class="n">tokens</span>
</code></pre>

</div>



<p>This saved me <strong>~$1,200/month</strong> in API costs by intelligently pruning conversation history.</p>

<h3>
  
  
  4. Monitoring Needs to Be Obsessive
</h3>

<p><strong>Metrics that actually matter:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>pie title "What Breaks AI Agents in Production"
    "Rate Limit Abuse" : 35
    "LLM Timeouts" : 25
    "Hallucinations" : 20
    "Network Issues" : 15
    "Database Locks" : 5
</code></pre>

</div>



<p><strong>My monitoring stack:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">dataclasses</span> <span class="kn">import</span> <span class="n">dataclass</span>
<span class="kn">from</span> <span class="n">datetime</span> <span class="kn">import</span> <span class="n">datetime</span>
<span class="kn">import</span> <span class="n">logging</span>

<span class="nd">@dataclass</span>
<span class="k">class</span> <span class="nc">AgentMetrics</span><span class="p">:</span>
    <span class="n">timestamp</span><span class="p">:</span> <span class="n">datetime</span>
    <span class="n">response_time_ms</span><span class="p">:</span> <span class="nb">float</span>
    <span class="n">tokens_used</span><span class="p">:</span> <span class="nb">int</span>
    <span class="n">cost_usd</span><span class="p">:</span> <span class="nb">float</span>
    <span class="n">user_satisfaction</span><span class="p">:</span> <span class="nb">float</span>
    <span class="n">error_type</span><span class="p">:</span> <span class="n">Optional</span><span class="p">[</span><span class="nb">str</span><span class="p">]</span>

    <span class="k">def</span> <span class="nf">log</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">logging</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span>
            <span class="sa">f</span><span class="sh">"</span><span class="s">agent_response</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">extra</span><span class="o">=</span><span class="p">{</span>
                <span class="sh">"</span><span class="s">duration_ms</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">response_time_ms</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">tokens</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">tokens_used</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">cost</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">cost_usd</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">satisfaction</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">user_satisfaction</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">error_type</span>
            <span class="p">}</span>
        <span class="p">)</span>

<span class="k">class</span> <span class="nc">AgentMonitor</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">metrics</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="n">self</span><span class="p">.</span><span class="n">alerts</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">high_latency</span><span class="sh">"</span><span class="p">:</span> <span class="mi">2000</span><span class="p">,</span>  <span class="c1"># ms
</span>            <span class="sh">"</span><span class="s">low_satisfaction</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.6</span><span class="p">,</span>  <span class="c1"># 0-1
</span>            <span class="sh">"</span><span class="s">error_rate</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.05</span>  <span class="c1"># 5%
</span>        <span class="p">}</span>

    <span class="k">async</span> <span class="k">def</span> <span class="nf">track_request</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">request_fn</span><span class="p">):</span>
        <span class="n">start</span> <span class="o">=</span> <span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span>
        <span class="n">error</span> <span class="o">=</span> <span class="bp">None</span>

        <span class="k">try</span><span class="p">:</span>
            <span class="n">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">request_fn</span><span class="p">()</span>
            <span class="n">satisfaction</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">calculate_satisfaction</span><span class="p">(</span><span class="n">result</span><span class="p">)</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">error</span> <span class="o">=</span> <span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">)</span>
            <span class="k">raise</span>
        <span class="k">finally</span><span class="p">:</span>
            <span class="n">duration</span> <span class="o">=</span> <span class="p">(</span><span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">-</span> <span class="n">start</span><span class="p">).</span><span class="nf">total_seconds</span><span class="p">()</span> <span class="o">*</span> <span class="mi">1000</span>

            <span class="n">metric</span> <span class="o">=</span> <span class="nc">AgentMetrics</span><span class="p">(</span>
                <span class="n">timestamp</span><span class="o">=</span><span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">(),</span>
                <span class="n">response_time_ms</span><span class="o">=</span><span class="n">duration</span><span class="p">,</span>
                <span class="n">tokens_used</span><span class="o">=</span><span class="nf">getattr</span><span class="p">(</span><span class="n">result</span><span class="p">,</span> <span class="sh">'</span><span class="s">tokens</span><span class="sh">'</span><span class="p">,</span> <span class="mi">0</span><span class="p">),</span>
                <span class="n">cost_usd</span><span class="o">=</span><span class="n">self</span><span class="p">.</span><span class="nf">calculate_cost</span><span class="p">(</span><span class="n">result</span><span class="p">),</span>
                <span class="n">user_satisfaction</span><span class="o">=</span><span class="n">satisfaction</span> <span class="k">if</span> <span class="n">error</span> <span class="ow">is</span> <span class="bp">None</span> <span class="k">else</span> <span class="mi">0</span><span class="p">,</span>
                <span class="n">error_type</span><span class="o">=</span><span class="n">error</span>
            <span class="p">)</span>

            <span class="n">metric</span><span class="p">.</span><span class="nf">log</span><span class="p">()</span>
            <span class="n">self</span><span class="p">.</span><span class="nf">check_alerts</span><span class="p">(</span><span class="n">metric</span><span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1460925895917-afdab827c52f%3Fw%3D1200%26q%3D80" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1460925895917-afdab827c52f%3Fw%3D1200%26q%3D80" alt="Monitoring Dashboard" width="1200" height="855"></a></p>

<h3>
  
  
  5. Fallbacks Save Your Reputation
</h3>

<p><strong>The moment of truth:</strong> Your AI provider goes down at 2 AM. What happens?</p>

<p><strong>Bad approach:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Hope and pray
</span><span class="n">response</span> <span class="o">=</span> <span class="n">openai</span><span class="p">.</span><span class="n">ChatCompletion</span><span class="p">.</span><span class="nf">create</span><span class="p">(...)</span>
</code></pre>

</div>



<p><strong>Production approach:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">List</span><span class="p">,</span> <span class="n">Callable</span>
<span class="kn">import</span> <span class="n">asyncio</span>

<span class="k">class</span> <span class="nc">AIAgentWithFallbacks</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">providers</span> <span class="o">=</span> <span class="p">[</span>
            <span class="n">self</span><span class="p">.</span><span class="n">primary_ai</span><span class="p">,</span>      <span class="c1"># OpenAI GPT-4
</span>            <span class="n">self</span><span class="p">.</span><span class="n">secondary_ai</span><span class="p">,</span>    <span class="c1"># Anthropic Claude
</span>            <span class="n">self</span><span class="p">.</span><span class="n">rule_based</span><span class="p">,</span>      <span class="c1"># Template responses
</span>            <span class="n">self</span><span class="p">.</span><span class="n">human_handoff</span>    <span class="c1"># Last resort
</span>        <span class="p">]</span>

    <span class="k">async</span> <span class="k">def</span> <span class="nf">get_response</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">message</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">max_retries</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">3</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Try providers in order until success</span><span class="sh">"""</span>

        <span class="k">for</span> <span class="n">provider</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">providers</span><span class="p">:</span>
            <span class="k">for</span> <span class="n">attempt</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">max_retries</span><span class="p">):</span>
                <span class="k">try</span><span class="p">:</span>
                    <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">provider</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>
                    <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="nf">is_valid_response</span><span class="p">(</span><span class="n">response</span><span class="p">):</span>
                        <span class="k">return</span> <span class="n">response</span>
                <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
                    <span class="n">logging</span><span class="p">.</span><span class="nf">warning</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">provider</span><span class="p">.</span><span class="n">__name__</span><span class="si">}</span><span class="s"> failed: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
                    <span class="k">await</span> <span class="n">asyncio</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">2</span> <span class="o">**</span> <span class="n">attempt</span><span class="p">)</span>  <span class="c1"># Exponential backoff
</span>                    <span class="k">continue</span>

        <span class="c1"># All providers failed
</span>        <span class="k">return</span> <span class="sh">"</span><span class="s">I apologize, but I</span><span class="sh">'</span><span class="s">m having technical difficulties. A human agent will assist you shortly.</span><span class="sh">"</span>
</code></pre>

</div>



<p><strong>Stats from production:</strong></p>

<ul>
<li>Primary provider uptime: 99.2%</li>
<li>Fallback triggers: 124 times/month</li>
<li>User complaints about downtime: <strong>0</strong>
</li>
</ul>

<h2>
  
  
  The Architecture That Actually Works
</h2>

<p>After three months of iteration, here's the stack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>graph LR
    A[User] --&gt; B[Load Balancer]
    B --&gt; C[API Gateway]
    C --&gt; D{Rate Limiter}
    D --&gt; E[Message Queue]
    E --&gt; F[Agent Pool]
    F --&gt; G[Primary AI]
    F --&gt; H[Fallback AI]
    F --&gt; I[Rules Engine]
    G --&gt; J[Validator]
    H --&gt; J
    I --&gt; J
    J --&gt; K[Response Cache]
    K --&gt; A

    L[Monitor] -.-&gt; F
    L -.-&gt; G
    L -.-&gt; H
    M[Database] -.-&gt; F
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1451187580459-43490279c0fa%3Fw%3D1200%26q%3D80" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1451187580459-43490279c0fa%3Fw%3D1200%26q%3D80" alt="Production Architecture" width="1200" height="798"></a></p>

<p><strong>Key components:</strong></p>

<ol>
<li>
<strong>Load balancer</strong> - Distributes traffic</li>
<li>
<strong>Rate limiter</strong> - Protects against abuse</li>
<li>
<strong>Message queue</strong> - Handles spikes</li>
<li>
<strong>Agent pool</strong> - Scales horizontally</li>
<li>
<strong>Validator</strong> - Catches hallucinations</li>
<li>
<strong>Cache</strong> - Reduces costs 40%</li>
<li>
<strong>Monitor</strong> - Real-time alerts</li>
</ol>

<h2>
  
  
  Real Numbers After 3 Months
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Total messages</td>
<td>52,847</td>
</tr>
<tr>
<td>Avg response time</td>
<td>847ms</td>
</tr>
<tr>
<td>Uptime</td>
<td>99.97%</td>
</tr>
<tr>
<td>Cost per message</td>
<td>$0.034</td>
</tr>
<tr>
<td>User satisfaction</td>
<td>4.6/5.0</td>
</tr>
<tr>
<td>Hallucinations caught</td>
<td>38</td>
</tr>
<tr>
<td>Abuse attempts blocked</td>
<td>2,847</td>
</tr>
<tr>
<td>Fallback activations</td>
<td>124</td>
</tr>
</tbody>
</table></div>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1551288049-bebda4e38f71%3Fw%3D1200%26q%3D80" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1551288049-bebda4e38f71%3Fw%3D1200%26q%3D80" alt="Results Dashboard" width="1200" height="800"></a></p>

<h2>
  
  
  What I'd Do Differently
</h2>

<p><strong>If I started over today:</strong></p>

<ol>
<li>✅ <strong>Start with rate limiting</strong> - Day 1, not day 30</li>
<li>✅ <strong>Build monitoring first</strong> - You can't fix what you can't see</li>
<li>✅ <strong>Plan for hallucinations</strong> - They WILL happen</li>
<li>✅ <strong>Design fallbacks early</strong> - Don't wait for an outage</li>
<li>✅ <strong>Cache aggressively</strong> - 40% cost reduction, zero effort</li>
</ol>

<p><strong>What worked perfectly:</strong></p>

<ul>
<li>SQLite for conversation history (yes, SQLite in production)</li>
<li>Bun for API server (3x faster than Node)</li>
<li>Simple rule-based fallbacks (saved my reputation twice)</li>
</ul>

<h2>
  
  
  The Code (Open Source)
</h2>

<p>Want to see the actual implementation? I open-sourced the core components:</p>

<p><strong>GitHub:</strong> <a href="https://github.com/Richardmsbr/atlas-ai-chat" rel="noopener noreferrer">github.com/Richardmsbr/atlas-ai-chat</a></p>

<p>Includes:</p>

<ul>
<li>Rate limiter with adaptive limits</li>
<li>Response validator</li>
<li>Context manager</li>
<li>Fallback system</li>
<li>Monitoring stack</li>
</ul>

<h2>
  
  
  Questions for You
</h2>

<p>I'm curious about your experience:</p>

<ol>
<li><strong>What's your biggest AI agent production challenge?</strong></li>
<li><strong>Have you dealt with hallucinations? How did you handle it?</strong></li>
<li><strong>What's your monitoring strategy?</strong></li>
</ol>

<p>Drop your answers below - I respond to every comment.</p>




<p><strong>Want the full deep dive?</strong> I wrote a complete Portuguese version with more code examples on my blog: <a href="https://blog.sakaguchi.ia.br/ai-agents-producao-realidade" rel="noopener noreferrer">blog.sakaguchi.ia.br/ai-agents-producao-realidade</a></p>

<p><strong>Connect with me:</strong></p>

<ul>
<li>GitHub: <a href="https://github.com/Richardmsbr" rel="noopener noreferrer">@Richardmsbr</a>
</li>
<li>Building AI agents at scale</li>
<li>Solutions Architect focusing on production AI systems</li>
</ul>




<p>Images: <a href="https://unsplash.com" rel="noopener noreferrer">Unsplash</a></p>

