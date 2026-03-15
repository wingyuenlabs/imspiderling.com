---
Title: I Built an AI That Finds Your Bugs and Rewrites Your Code to Fix Them.
Description: 
Author: Kamaumbugua-dev
Date: 2026-03-15T21:36:43.000Z
Robots: noindex,nofollow
Template: index
---
<p>How I built CodeLens — a Groq-powered code review tool that detects SQL injection, memory leaks, and O(n²) algorithms, then rewrites your entire file with all issues resolved. Full breakdown of the architecture, prompt engineering tricks, and the LLM hallucination problem I had to solve.</p>

<p>Every developer has shipped a bug they should have caught.</p>

<p>Not because they were careless. Because code review is expensive. You're scanning hundreds of lines for subtle patterns: a missing <code>conn.close()</code>, an f-string wired directly into a SQL query, a nested loop that looks innocent at <code>n = 10</code> but detonates at <code>n = 10,000</code>.</p>

<p>I wanted to build a tool that never gets tired, never misses a pattern, and can tell you exactly what will go wrong in production — before you push.</p>

<p>That's <strong>CodeLens</strong>.</p>




<h2>
  
  
  What It Does
</h2>

<p>Paste any code. In seconds you get:</p>

<ul>
<li>A <strong>health score</strong> (0–100) with an animated gauge</li>
<li>Every vulnerability categorized by severity: <code>CRITICAL</code>, <code>WARNING</code>, <code>INFO</code>
</li>
<li>Exact line numbers, descriptions, fix suggestions, and predicted production impact</li>
<li>A <strong>"Rework Code"</strong> button that rewrites your entire file with every issue resolved, with inline <code># FIX:</code> comments explaining each change</li>
</ul>

<p>Here's what it catches on a simple Python file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CRITICAL  SQL Injection            L7     f-string in cursor.execute()
CRITICAL  Hardcoded Credentials    L27    password = "admin123"
CRITICAL  Unsafe eval()            L29    eval(open("config.txt").read())
CRITICAL  Plaintext Card Numbers   L15    print(f"...card {card_number}")
WARNING   Resource Leak             L16    file handle never closed
WARNING   Resource Leak             L42    db connection never closed
WARNING   O(n²) Complexity          L46    nested loop over same list
WARNING   Unbounded Cache           L38    dict with no eviction policy
INFO      Division by Zero Risk     L50    len(transactions) unchecked
</code></pre>

</div>



<p>Health score: <strong>28 / 100</strong>.</p>

<p>One click later, the LLM rewrites the file. Every issue fixed. Every change commented.</p>




<h2>
  
  
  The Stack
</h2>

<p>Deliberately lean:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>React 19 (Vercel)  →  FastAPI (Render)  →  Groq API (llama-3.3-70b)
</code></pre>

</div>



<p>No database. No auth. No queue. Every request is stateless — code goes in, analysis comes out.</p>

<p>The frontend is a three-panel layout:</p>

<ol>
<li>
<strong>Code editor</strong> — line numbers highlight affected lines in red</li>
<li>
<strong>Analysis dashboard</strong> — health gauge, metric bars, issue list with severity filters</li>
<li>
<strong>Vulnerability slides</strong> — right panel with CSS scroll-snap, one full-height card per vulnerability</li>
</ol>

<p>The backend has three endpoints worth talking about: <code>/analyze</code>, <code>/fix</code>, and <code>/github/analyze</code>.</p>




<h2>
  
  
  The Hard Part: Getting the LLM to Return Valid JSON Every Time
</h2>

<p>The analysis response needs to be machine-parseable. Every time. Across any language, any code quality, any edge case.</p>

<p>This is harder than it sounds. By default, models wrap JSON in markdown fences, add explanatory preamble, or truncate responses mid-object when they hit a token limit. Any of these breaks the frontend.</p>

<p>My system prompt ends with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Return ONLY valid JSON. No markdown, no code fences, no explanation outside the JSON.
</code></pre>

</div>



<p>And I strip artifacts post-response with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">raw_text</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">^```

(?:json)?\s*</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">,</span> <span class="n">raw_text</span><span class="p">)</span>
<span class="n">raw_text</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">\s*

```$</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">,</span> <span class="n">raw_text</span><span class="p">)</span>
<span class="n">analysis</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">raw_text</span><span class="p">)</span>
</code></pre>

</div>



<p>This handles 99% of cases. The remaining 1% raises a <code>json.JSONDecodeError</code> that returns a structured 500 to the client.</p>




<h2>
  
  
  The Line Number Hallucination Problem
</h2>

<p>This was the most interesting bug I fixed.</p>

<p>Early versions of CodeLens would confidently report issues on lines that didn't exist. A 50-line file would get issues flagged at lines 73, 91, 108. The model was pattern-matching against training data — it recognized the <em>type</em> of bug and estimated a line number based on where it typically appears in codebases it had seen, not in the code you gave it.</p>

<p>The fix is obvious in hindsight: <strong>give the model line numbers to reference.</strong></p>

<p>Instead of sending:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">sqlite3</span>

<span class="k">def</span> <span class="nf">get_user</span><span class="p">(</span><span class="n">username</span><span class="p">):</span>
    <span class="n">query</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">SELECT * FROM users WHERE username = </span><span class="sh">'</span><span class="si">{</span><span class="n">username</span><span class="si">}</span><span class="sh">'"</span>
</code></pre>

</div>



<p>I send:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="mi">1</span> <span class="o">|</span> <span class="kn">import</span> <span class="n">sqlite3</span>
<span class="mi">2</span> <span class="o">|</span>
<span class="mi">3</span> <span class="o">|</span> <span class="k">def</span> <span class="nf">get_user</span><span class="p">(</span><span class="n">username</span><span class="p">):</span>
<span class="mi">4</span> <span class="o">|</span>     <span class="n">query</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">SELECT * FROM users WHERE username = </span><span class="sh">'</span><span class="si">{</span><span class="n">username</span><span class="si">}</span><span class="sh">'"</span>
</code></pre>

</div>



<p>And I add an explicit constraint to the prompt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>The code has 50 lines total. You MUST only reference line numbers
that actually exist (1 to 50).
</code></pre>

</div>



<p>The implementation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">add_line_numbers</span><span class="p">(</span><span class="n">code</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="n">lines</span> <span class="o">=</span> <span class="n">code</span><span class="p">.</span><span class="nf">splitlines</span><span class="p">()</span>
    <span class="n">width</span> <span class="o">=</span> <span class="nf">len</span><span class="p">(</span><span class="nf">str</span><span class="p">(</span><span class="nf">len</span><span class="p">(</span><span class="n">lines</span><span class="p">)))</span>
    <span class="k">return</span> <span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span>
        <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">str</span><span class="p">(</span><span class="n">i</span><span class="o">+</span><span class="mi">1</span><span class="p">).</span><span class="nf">rjust</span><span class="p">(</span><span class="n">width</span><span class="p">)</span><span class="si">}</span><span class="s"> | </span><span class="si">{</span><span class="n">line</span><span class="si">}</span><span class="sh">"</span>
        <span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="n">line</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="n">lines</span><span class="p">)</span>
    <span class="p">)</span>
</code></pre>

</div>



<p>Hallucinated line numbers dropped to near zero. The model now has a concrete anchor instead of a floating reference.</p>




<h2>
  
  
  The Rework Pipeline
</h2>

<p>The "Rework Code" feature is a second LLM call chained to the first.</p>

<p>After analysis, the frontend sends the original code + the full issue list to <code>/fix</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">FixRequest</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">code</span><span class="p">:</span> <span class="nb">str</span>
    <span class="n">language</span><span class="p">:</span> <span class="nb">str</span>
    <span class="n">issues</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="n">Any</span><span class="p">]</span>
</code></pre>

</div>



<p>The fix prompt encodes every issue as a line-referenced instruction:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">Fix</span> <span class="n">ALL</span> <span class="n">of</span> <span class="n">the</span> <span class="n">following</span> <span class="n">issues</span> <span class="ow">in</span> <span class="n">this</span> <span class="n">python</span> <span class="n">code</span><span class="p">:</span>

<span class="n">ISSUES</span> <span class="n">TO</span> <span class="n">FIX</span><span class="p">:</span>
  <span class="o">-</span> <span class="p">[</span><span class="n">Line</span> <span class="mi">7</span><span class="p">]</span> <span class="p">[</span><span class="n">CRITICAL</span><span class="p">]</span> <span class="n">SQL</span> <span class="n">Injection</span><span class="p">:</span> <span class="n">Use</span> <span class="n">parameterized</span> <span class="n">queries</span>
  <span class="o">-</span> <span class="p">[</span><span class="n">Line</span> <span class="mi">27</span><span class="p">]</span> <span class="p">[</span><span class="n">CRITICAL</span><span class="p">]</span> <span class="n">Hardcoded</span> <span class="n">Credentials</span><span class="p">:</span> <span class="n">Use</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(...)</span>
  <span class="o">-</span> <span class="p">[</span><span class="n">Line</span> <span class="mi">29</span><span class="p">]</span> <span class="p">[</span><span class="n">CRITICAL</span><span class="p">]</span> <span class="n">Unsafe</span> <span class="nf">eval</span><span class="p">():</span> <span class="n">Use</span> <span class="n">json</span><span class="p">.</span><span class="nf">load</span><span class="p">()</span> <span class="n">instead</span>
  <span class="bp">...</span>

<span class="n">ORIGINAL</span> <span class="n">CODE</span><span class="p">:</span>
<span class="p">{</span><span class="n">code</span><span class="p">}</span>

<span class="n">Return</span> <span class="n">the</span> <span class="n">complete</span> <span class="n">fixed</span> <span class="n">code</span> <span class="k">with</span> <span class="n">inline</span> <span class="n">FIX</span> <span class="n">comments</span><span class="p">.</span>
</code></pre>

</div>



<p>The system prompt is strict:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">Return</span> <span class="n">RAW</span> <span class="n">CODE</span> <span class="n">ONLY</span><span class="p">.</span> <span class="n">No</span> <span class="n">markdown</span> <span class="n">fences</span><span class="p">,</span> <span class="n">no</span> <span class="n">explanation</span><span class="p">,</span> <span class="n">no</span> <span class="n">preamble</span><span class="p">.</span>
<span class="n">Add</span> <span class="n">inline</span> <span class="n">comments</span> <span class="n">prefixed</span> <span class="k">with</span> <span class="c1"># FIX: explaining each change.
</span></code></pre>

</div>



<p>The result gets placed back into the editor. The user sees their fixed file immediately.</p>




<h2>
  
  
  The CORS Bug That Burned Two Hours
</h2>

<p>Deploying to Vercel + Render exposed something I'd glossed over: <code>allow_origins=["*"]</code> and <code>allow_credentials=True</code> is <strong>invalid</strong> per the CORS specification.</p>

<p>Browsers enforce this at the preflight stage. Your OPTIONS request returns 200, but the browser rejects the response because the spec says wildcard origins cannot coexist with credentials. You get a cryptic console error and a silent failure in the UI.</p>

<p>The fix is one line:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">app</span><span class="p">.</span><span class="nf">add_middleware</span><span class="p">(</span>
    <span class="n">CORSMiddleware</span><span class="p">,</span>
    <span class="n">allow_origins</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">*</span><span class="sh">"</span><span class="p">],</span>
    <span class="n">allow_credentials</span><span class="o">=</span><span class="bp">False</span><span class="p">,</span>  <span class="c1"># must be False with wildcard origin
</span>    <span class="n">allow_methods</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">*</span><span class="sh">"</span><span class="p">],</span>
    <span class="n">allow_headers</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">*</span><span class="sh">"</span><span class="p">],</span>
<span class="p">)</span>
</code></pre>

</div>



<p>Worth knowing before you spend two hours debugging network tab preflight responses.</p>




<h2>
  
  
  The Vulnerability Slides
</h2>

<p>The right panel uses CSS <code>scroll-snap-type: y mandatory</code>. Each vulnerability gets its own full-height card:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight css"><code><span class="nt">scroll-snap-type</span><span class="o">:</span> <span class="nt">y</span> <span class="nt">mandatory</span><span class="o">;</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="p">&lt;</span><span class="nt">div</span> <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">height</span><span class="p">:</span> <span class="dl">"</span><span class="s2">100%</span><span class="dl">"</span><span class="p">,</span> <span class="na">scrollSnapAlign</span><span class="p">:</span> <span class="dl">"</span><span class="s2">start</span><span class="dl">"</span> <span class="p">}</span><span class="si">}</span><span class="p">&gt;</span>
  <span class="p">&lt;</span><span class="nc">VulnSlide</span> <span class="na">issue</span><span class="p">=</span><span class="si">{</span><span class="nx">issue</span><span class="si">}</span> <span class="p">/&gt;</span>
<span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
</code></pre>

</div>



<p>There's a dot navigation sidebar that syncs with the scroll position:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="nx">onScroll</span><span class="o">=</span><span class="p">{(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">idx</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span>
    <span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">scrollTop</span> <span class="o">/</span> <span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">clientHeight</span>
  <span class="p">);</span>
  <span class="nf">setActiveSlide</span><span class="p">(</span><span class="nx">idx</span><span class="p">);</span>
<span class="p">}}</span>
</code></pre>

</div>



<p>Rounding (not flooring) prevents the active dot from flickering during the snap animation — the snap always settles on an integer, but <code>scrollTop</code> passes through fractional values mid-animation.</p>

<p>Each slide has a "SLIDE" button in the issue list that calls:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">slidesRef</span><span class="p">.</span><span class="nx">current</span><span class="p">.</span><span class="nf">scrollTo</span><span class="p">({</span>
  <span class="na">top</span><span class="p">:</span> <span class="nx">idx</span> <span class="o">*</span> <span class="nx">slidesRef</span><span class="p">.</span><span class="nx">current</span><span class="p">.</span><span class="nx">clientHeight</span><span class="p">,</span>
  <span class="na">behavior</span><span class="p">:</span> <span class="dl">"</span><span class="s2">smooth</span><span class="dl">"</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Bi-directional sync between the list and the slides, no state management library needed.</p>




<h2>
  
  
  Deployment Notes
</h2>

<p>A few things that bit me:</p>

<p><strong>Render cold starts.</strong> The free tier sleeps services after 15 minutes of inactivity. First request after sleep takes 30–50 seconds. I added a loading state with an explanation so users wait instead of leave.</p>

<p><strong>Vite bakes env vars at build time.</strong> <code>VITE_API_BASE</code> is injected into the bundle when Vercel builds — not at runtime. Old preview deployment URLs serve old bundles permanently. The production domain always reflects the latest build. If your frontend is still hitting the wrong backend, you're on an old preview URL.</p>

<p><strong>Railway port mismatch.</strong> I originally deployed on Railway. The dashboard had the networking port set to 8000, but the <code>$PORT</code> environment variable was 8080. Internal healthchecks passed (Railway probed the container directly), but external traffic failed at the edge with persistent 502s. Moved to Render, problem gone.</p>




<h2>
  
  
  Try It
</h2>

<p><strong>Live:</strong> <a href="https://codelens-new.vercel.app" rel="noopener noreferrer">codelens-new.vercel.app</a></p>

<p><strong>Source:</strong> <a href="https://github.com/Kamaumbugua-dev/CODELENS" rel="noopener noreferrer">github.com/Kamaumbugua-dev/CODELENS</a></p>

<p>Paste the worst code you can find. The demo loads a Python file with SQL injection, hardcoded secrets, unsafe <code>eval()</code>, and an O(n²) algorithm. Hit Analyze, then Rework. The whole thing takes about 10 seconds on a warm backend.</p>




<p>Built by <strong>Steven K.</strong> — Head of <strong>AXON LATTICE LABS™</strong></p>

<p><em>CodeLens™ — See your code's future before it ships.</em></p>

