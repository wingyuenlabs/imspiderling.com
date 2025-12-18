---
Title: I Stopped Writing Code. Here's What I Do Instead (Vibe Coding in 2025)
Description: 
Author: ishrat
Date: 2025-12-18T21:37:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been coding for over a decade. Last month, I built a complete SaaS MVP in 3 days.</p>

<p>Not because I suddenly became 10x faster at typing. Because I barely typed any code at all.</p>

<p>Welcome to <strong>vibe coding</strong> — and it's changing everything.</p>

<p>## What is Vibe Coding?</p>

<p>The term was coined by Andrej Karpathy (yes, the Tesla AI guy) in early 2025:</p>

<blockquote>
<p>"It's not really coding — I just see stuff, say stuff, run stuff, and copy-paste stuff, and it mostly works."</p>
</blockquote>

<p>Instead of writing code line-by-line, you:</p>

<ol>
<li>
<strong>Describe</strong> what you want in plain English</li>
<li>
<strong>Let AI</strong> generate the code</li>
<li>
<strong>Run it</strong> and see what happens</li>
<li>
<strong>Paste errors</strong> back to AI to fix</li>
</ol>

<p>That's it. That's the whole workflow.</p>

<p>## Why Should You Care?</p>

<p>According to JetBrains' 2025 developer survey:</p>

<ul>
<li>
<strong>85%</strong> of developers now use AI tools regularly</li>
<li>
<strong>62%</strong> rely on at least one AI coding assistant</li>
<li>Developers save <strong>1-8+ hours per week</strong>
</li>
</ul>

<p>This isn't a fad. This is the new normal.</p>

<p>## Free Tools to Start Vibe Coding Today (No Credit Card Needed)</p>

<p>You don't need to spend a dime to get started. Here are completely free options:</p>

<p>| Tool | What It Does | Why It's Great |<br>
  |------|--------------|----------------|<br>
  | <strong><a href="https://cursor.sh" rel="noopener noreferrer">Cursor</a></strong> | AI-powered IDE | Free tier with 2000 completions/month |<br>
  | <strong><a href="https://codeium.com" rel="noopener noreferrer">Codeium</a></strong> | Code autocomplete | 100% free, unlimited, no catch |<br>
  | <strong><a href="https://continue.dev" rel="noopener noreferrer">Continue.dev</a></strong> | Open source AI assistant | Free forever, works with VS Code |<br>
  | <strong><a href="https://aider.chat" rel="noopener noreferrer">Aider</a></strong> | Terminal AI pair programmer | Free &amp; open source |<br>
  | <strong><a href="https://claude.ai" rel="noopener noreferrer">Claude.ai</a></strong> | Chat-based coding help | Free tier available |<br>
  | <strong><a href="https://chat.openai.com" rel="noopener noreferrer">ChatGPT</a></strong> | General AI coding | Free tier available |<br>
  | <strong><a href="https://github.com/features/copilot" rel="noopener noreferrer">GitHub Copilot</a></strong> | Inline suggestions | Free for students &amp; open source maintainers |</p>

<p><strong>My recommendation for beginners:</strong> Start with <strong>Codeium</strong> (unlimited free) + <strong>Claude.ai</strong> (for complex questions). Zero cost, full power.</p>

<p>## Paid Tools (When You're Ready)</p>

<p>Once you're comfortable, these are worth the investment:</p>

<p>| Tool | Cost | Best For |<br>
  |------|------|----------|<br>
  | Cursor Pro | $20/mo | Full IDE experience |<br>
  | GitHub Copilot | $10/mo | Inline completions |<br>
  | Claude Pro | $20/mo | Complex reasoning |</p>

<p>But seriously, the free tools are incredibly capable. I used only free tools for my first 6 months of vibe coding.</p>

<p>## Real Example: Building a URL Shortener in 10 Minutes</p>

<p>Here's exactly what I typed into Claude (free tier):</p>

<p>Create a FastAPI URL shortener with:</p>

<ul>
<li>POST /shorten - accepts URL, returns short code</li>
<li>GET /{code} - redirects to original URL</li>
<li>SQLite storage</li>
<li>Rate limiting (10 req/min per IP)</li>
</ul>

<p>What AI generated (working code):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>  <span class="kn">from</span> <span class="n">fastapi</span> <span class="kn">import</span> <span class="n">FastAPI</span><span class="p">,</span> <span class="n">HTTPException</span><span class="p">,</span> <span class="n">Request</span>
  <span class="kn">from</span> <span class="n">fastapi.responses</span> <span class="kn">import</span> <span class="n">RedirectResponse</span>
  <span class="kn">from</span> <span class="n">slowapi</span> <span class="kn">import</span> <span class="n">Limiter</span>
  <span class="kn">from</span> <span class="n">slowapi.util</span> <span class="kn">import</span> <span class="n">get_remote_address</span>
  <span class="kn">import</span> <span class="n">sqlite3</span>
  <span class="kn">import</span> <span class="n">string</span>
  <span class="kn">import</span> <span class="n">random</span>

  <span class="n">app</span> <span class="o">=</span> <span class="nc">FastAPI</span><span class="p">()</span>
  <span class="n">limiter</span> <span class="o">=</span> <span class="nc">Limiter</span><span class="p">(</span><span class="n">key_func</span><span class="o">=</span><span class="n">get_remote_address</span><span class="p">)</span>

  <span class="k">def</span> <span class="nf">init_db</span><span class="p">():</span>
      <span class="n">conn</span> <span class="o">=</span> <span class="n">sqlite3</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="sh">"</span><span class="s">urls.db</span><span class="sh">"</span><span class="p">)</span>
      <span class="n">conn</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"""</span><span class="s">
          CREATE TABLE IF NOT EXISTS urls (
              code TEXT PRIMARY KEY,
              url TEXT NOT NULL
          )
      </span><span class="sh">"""</span><span class="p">)</span>
      <span class="k">return</span> <span class="n">conn</span>

  <span class="nd">@app.post</span><span class="p">(</span><span class="sh">"</span><span class="s">/shorten</span><span class="sh">"</span><span class="p">)</span>
  <span class="nd">@limiter.limit</span><span class="p">(</span><span class="sh">"</span><span class="s">10/minute</span><span class="sh">"</span><span class="p">)</span>
  <span class="k">async</span> <span class="k">def</span> <span class="nf">shorten</span><span class="p">(</span><span class="n">request</span><span class="p">:</span> <span class="n">Request</span><span class="p">,</span> <span class="n">url</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
      <span class="n">conn</span> <span class="o">=</span> <span class="nf">init_db</span><span class="p">()</span>
      <span class="n">code</span> <span class="o">=</span> <span class="sh">''</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">choices</span><span class="p">(</span><span class="n">string</span><span class="p">.</span><span class="n">ascii_letters</span><span class="p">,</span> <span class="n">k</span><span class="o">=</span><span class="mi">6</span><span class="p">))</span>
      <span class="n">conn</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">INSERT INTO urls VALUES (?, ?)</span><span class="sh">"</span><span class="p">,</span> <span class="p">(</span><span class="n">code</span><span class="p">,</span> <span class="n">url</span><span class="p">))</span>
      <span class="n">conn</span><span class="p">.</span><span class="nf">commit</span><span class="p">()</span>
      <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">short_url</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">http://localhost:8000/</span><span class="si">{</span><span class="n">code</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>

  <span class="nd">@app.get</span><span class="p">(</span><span class="sh">"</span><span class="s">/{code}</span><span class="sh">"</span><span class="p">)</span>
  <span class="k">async</span> <span class="k">def</span> <span class="nf">redirect</span><span class="p">(</span><span class="n">code</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
      <span class="n">conn</span> <span class="o">=</span> <span class="nf">init_db</span><span class="p">()</span>
      <span class="n">result</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span>
          <span class="sh">"</span><span class="s">SELECT url FROM urls WHERE code = ?</span><span class="sh">"</span><span class="p">,</span> <span class="p">(</span><span class="n">code</span><span class="p">,)</span>
      <span class="p">).</span><span class="nf">fetchone</span><span class="p">()</span>
      <span class="k">if</span> <span class="ow">not</span> <span class="n">result</span><span class="p">:</span>
          <span class="k">raise</span> <span class="nc">HTTPException</span><span class="p">(</span><span class="mi">404</span><span class="p">,</span> <span class="sh">"</span><span class="s">Not found</span><span class="sh">"</span><span class="p">)</span>
      <span class="k">return</span> <span class="nc">RedirectResponse</span><span class="p">(</span><span class="n">result</span><span class="p">[</span><span class="mi">0</span><span class="p">])</span>
</code></pre>

</div>



<p>Ran it. Worked first try. 10 minutes including testing. Cost: $0</p>

<p>The Prompting Secret Nobody Talks About</p>

<p>Bad prompt:<br>
  make a todo app</p>

<p>Good prompt:<br>
  Create a React TypeScript todo app with:</p>

<ul>
<li>Add/delete/toggle todos</li>
<li>LocalStorage persistence</li>
<li>Filter: all, active, completed</li>
<li>Tailwind CSS styling</li>
<li>Responsive design</li>
</ul>

<p>The difference? Specificity.</p>

<p>The more constraints you give, the better the output. Think of it like giving directions — "go somewhere nice" vs "go to the Italian restaurant on 5th street."</p>

<p>When Vibe Coding Fails</p>

<p>Let's be real. It's not magic. It struggles with:</p>

<ul>
<li>Complex algorithms — AI often produces O(n²) when O(n) exists</li>
<li>Security-critical code — Always review auth/crypto manually</li>
<li>Large codebase refactors — Context window limits hurt</li>
<li>Highly custom business logic — It doesn't know YOUR domain</li>
</ul>

<p>I still write code manually for ~30% of my work. But that 70% automation? Game changer.</p>

<p>How to Start Today (5-Minute Setup)</p>

<p>Option A: Browser Only (Easiest)</p>

<ol>
<li>Go to <a href="https://claude.ai" rel="noopener noreferrer">https://claude.ai</a> or <a href="https://chat.openai.com" rel="noopener noreferrer">https://chat.openai.com</a>
</li>
<li>Describe what you want to build</li>
<li>Copy the code, run locally</li>
</ol>

<p>Option B: IDE Integration (Better)</p>

<ol>
<li>Install <a href="https://code.visualstudio.com" rel="noopener noreferrer">https://code.visualstudio.com</a>
</li>
<li>Add <a href="https://codeium.com" rel="noopener noreferrer">https://codeium.com</a> (free)</li>
<li>Start typing — AI suggests completions</li>
</ol>

<p>Option C: Full Vibe Coding Setup (Best)</p>

<ol>
<li>Install <a href="https://cursor.sh" rel="noopener noreferrer">https://cursor.sh</a> (free tier)</li>
<li>Open any project</li>
<li>Press Cmd+K / Ctrl+K</li>
<li>Type what you want in English</li>
<li>Hit enter and watch</li>
</ol>

<p>Start with small tasks:</p>

<ul>
<li>"Add a loading spinner to this button"</li>
<li>"Convert this function to async/await"</li>
<li>"Write tests for this module"</li>
</ul>

<p>Build confidence, then go bigger.</p>

<p>The Future is Conversational</p>

<p>In 5 years, I believe:</p>

<ul>
<li>Junior dev interviews will test prompting skills</li>
<li>"Vibe coder" will be a real job title</li>
<li>Writing code manually will feel like writing assembly</li>
</ul>

<p>The developers who adapt now will have a massive advantage.</p>




<p>Want More Developer Tutorials?</p>

<p>I write in-depth guides on AI, Python, and modern development at <a href="https://techyowls.io" rel="noopener noreferrer">https://techyowls.io</a> — practical tutorials from engineers who actually ship products.</p>




<p>What's your experience with AI coding tools? Drop a comment — I reply to everyone.</p>




