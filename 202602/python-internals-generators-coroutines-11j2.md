---
Title: Python Internals: Generators & Coroutines
Description: 
Author: aykhlf yassir
Date: 2026-02-16T21:24:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>There's a function that exists in almost every Python codebase. It looks harmless:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">get_trades</span><span class="p">(</span><span class="n">symbol</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="nb">dict</span><span class="p">]:</span>
    <span class="n">results</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">record</span> <span class="ow">in</span> <span class="n">enormous_database_cursor</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">record</span><span class="p">[</span><span class="sh">"</span><span class="s">symbol</span><span class="sh">"</span><span class="p">]</span> <span class="o">==</span> <span class="n">symbol</span><span class="p">:</span>
            <span class="n">results</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">record</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">results</span>

<span class="n">trades</span> <span class="o">=</span> <span class="nf">get_trades</span><span class="p">(</span><span class="sh">"</span><span class="s">AAPL</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># 💀 Waits. Waits. Waits. Then crashes.
</span></code></pre>

</div>



<p>The problems stack up fast:</p>

<ul>
<li>
<strong>High latency</strong>: You get <em>zero</em> items until the <em>entire</em> database is scanned</li>
<li>
<strong>Massive RAM</strong>: Every matching record is held in memory simultaneously</li>
<li>
<strong>Fragility</strong>: One spike in result size kills the process</li>
</ul>

<p>This is the "eager" pattern—do all the work, collect all the results, then hand them over. For small datasets, you'll never notice. For anything real-world, it's a time bomb.</p>

<p>The fix is a single keyword. But to use it correctly, you need to understand what it actually does to your function.</p>




<h2>
  
  
  1. The Basics: What <code>yield</code> Does to a Function
</h2>

<p>Every Python function you've written follows the same lifecycle:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Call → Execute → return value → Stack frame is destroyed → Done
</code></pre>

</div>



<p>Local variables evaporate. State is gone. The function has no memory that it ever ran.</p>

<p><code>yield</code> breaks this contract entirely.</p>

<h3>
  
  
  The Normal Function: A Sprint
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">countdown_list</span><span class="p">(</span><span class="n">n</span><span class="p">:</span> <span class="nb">int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="nb">int</span><span class="p">]:</span>
    <span class="n">result</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">while</span> <span class="n">n</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
        <span class="n">result</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">n</span><span class="p">)</span>
        <span class="n">n</span> <span class="o">-=</span> <span class="mi">1</span>
    <span class="k">return</span> <span class="n">result</span>  <span class="c1"># Hands you everything at once, then dies
</span></code></pre>

</div>



<p>One call. One massive result. The function's stack frame is created, used, and destroyed.</p>

<h3>
  
  
  The Generator: A Pause Button
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">countdown</span><span class="p">(</span><span class="n">n</span><span class="p">:</span> <span class="nb">int</span><span class="p">):</span>
    <span class="k">while</span> <span class="n">n</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
        <span class="k">yield</span> <span class="n">n</span>    <span class="c1"># Pause here, hand back n, wait to be resumed
</span>        <span class="n">n</span> <span class="o">-=</span> <span class="mi">1</span>
</code></pre>

</div>



<p>The moment Python sees <code>yield</code> in a function body, the rules change. Calling <code>countdown(5)</code> no longer executes a single line of code. Instead, Python hands you back a <strong>generator object</strong>—a suspended, ready-to-run machine.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">gen</span> <span class="o">=</span> <span class="nf">countdown</span><span class="p">(</span><span class="mi">5</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">gen</span><span class="p">)</span>          <span class="c1"># &lt;generator object countdown at 0x7f...&gt;
</span><span class="nf">print</span><span class="p">(</span><span class="nf">next</span><span class="p">(</span><span class="n">gen</span><span class="p">))</span>    <span class="c1"># 5  → Runs until yield, pauses, returns 5
</span><span class="nf">print</span><span class="p">(</span><span class="nf">next</span><span class="p">(</span><span class="n">gen</span><span class="p">))</span>    <span class="c1"># 4  → Resumes, runs until yield, pauses, returns 4
</span><span class="nf">print</span><span class="p">(</span><span class="nf">next</span><span class="p">(</span><span class="n">gen</span><span class="p">))</span>    <span class="c1"># 3  → Same
</span></code></pre>

</div>



<p><strong>What makes this possible?</strong> When a generator pauses at <code>yield</code>, its entire stack frame—local variables, current line number, the value of <code>n</code>—is moved from the stack to the <strong>heap</strong>. It doesn't disappear. It waits, frozen in time, until <code>next()</code> is called again.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Normal function:    [Stack Frame] → return → [Destroyed]

Generator:          [Stack Frame] → yield → [Moved to Heap, frozen]
                                          ↓
                    next() called  → [Thawed, execution resumes]
                                          ↓
                                   yield → [Frozen again]
</code></pre>

</div>



<h3>
  
  
  Old Way vs. New Way: Side by Side
</h3>

<p>Before generators, you had to implement the iterator protocol manually—a verbose class with <code>__iter__</code> and <code>__next__</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># THE OLD WAY: Class-based iterator (20 lines of boilerplate)
</span><span class="k">class</span> <span class="nc">Countdown</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">start</span><span class="p">:</span> <span class="nb">int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="n">self</span><span class="p">.</span><span class="n">current</span> <span class="o">=</span> <span class="n">start</span>

    <span class="k">def</span> <span class="nf">__iter__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="n">self</span>

    <span class="k">def</span> <span class="nf">__next__</span><span class="p">(</span><span class="n">self</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">int</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">current</span> <span class="o">&lt;=</span> <span class="mi">0</span><span class="p">:</span>
            <span class="k">raise</span> <span class="nb">StopIteration</span>
        <span class="n">value</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">current</span>
        <span class="n">self</span><span class="p">.</span><span class="n">current</span> <span class="o">-=</span> <span class="mi">1</span>
        <span class="k">return</span> <span class="n">value</span>

<span class="c1"># THE NEW WAY: Generator function (4 lines, zero boilerplate)
</span><span class="k">def</span> <span class="nf">countdown</span><span class="p">(</span><span class="n">n</span><span class="p">:</span> <span class="nb">int</span><span class="p">):</span>
    <span class="k">while</span> <span class="n">n</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
        <span class="k">yield</span> <span class="n">n</span>
        <span class="n">n</span> <span class="o">-=</span> <span class="mi">1</span>
</code></pre>

</div>



<p>Same behavior. Same memory efficiency. Same protocol compatibility—<code>countdown(5)</code> works anywhere <code>Countdown(5)</code> does.</p>

<p>The <code>yield</code> keyword is a class-based iterator, fully implemented, in one line.</p>




<h2>
  
  
  2. Infinite Data Pipelines: The "Pull" Model
</h2>

<p>Here's where generators move from "interesting" to "indispensable."</p>

<p>Consider the difference:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">sys</span>

<span class="c1"># Eager: allocate the entire sequence in RAM
</span><span class="n">big_list</span> <span class="o">=</span> <span class="p">[</span><span class="n">x</span> <span class="o">**</span> <span class="mi">2</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">1_000_000</span><span class="p">)]</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">List size:      </span><span class="si">{</span><span class="n">sys</span><span class="p">.</span><span class="nf">getsizeof</span><span class="p">(</span><span class="n">big_list</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">12</span><span class="p">,</span><span class="si">}</span><span class="s"> bytes</span><span class="sh">"</span><span class="p">)</span>  
<span class="c1"># List size:       8,448,728 bytes (~8 MB)
</span>
<span class="c1"># Lazy: a tiny object that knows *how* to produce values
</span><span class="n">big_gen</span> <span class="o">=</span> <span class="p">(</span><span class="n">x</span> <span class="o">**</span> <span class="mi">2</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">1_000_000</span><span class="p">))</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Generator size: </span><span class="si">{</span><span class="n">sys</span><span class="p">.</span><span class="nf">getsizeof</span><span class="p">(</span><span class="n">big_gen</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">12</span><span class="p">,</span><span class="si">}</span><span class="s"> bytes</span><span class="sh">"</span><span class="p">)</span>
<span class="c1"># Generator size:         104 bytes (~104 bytes)
</span></code></pre>

</div>



<p><strong>Eight megabytes vs. 104 bytes.</strong> The generator doesn't store the squares—it stores the <em>recipe</em> for producing the next one. Scale this to 10GB of log files or a live market feed, and this difference is what separates a working system from a crashed one.</p>

<h3>
  
  
  Generator Expressions: Lazy List Comprehensions
</h3>

<p>The <code>(x for x in ...)</code> syntax is a <strong>generator expression</strong>—the lazy sibling of list comprehensions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># List comprehension: eager, executes immediately
</span><span class="n">squares_list</span> <span class="o">=</span> <span class="p">[</span><span class="n">x</span> <span class="o">**</span> <span class="mi">2</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">10</span><span class="p">)]</span>    <span class="c1"># All 10 computed NOW
</span>
<span class="c1"># Generator expression: lazy, executes on demand
</span><span class="n">squares_gen</span>  <span class="o">=</span> <span class="p">(</span><span class="n">x</span> <span class="o">**</span> <span class="mi">2</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">10</span><span class="p">))</span>    <span class="c1"># NONE computed yet
</span></code></pre>

</div>



<p>Square brackets → list (eager). Parentheses → generator (lazy).</p>

<h3>
  
  
  The Pipeline Architecture
</h3>

<p>Generators compose naturally into pipelines—a chain of lazy transformations where data flows through only when pulled from the end:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  Source          Filter               Processor
    │                │                     │
market_ticker() ──► (t for t              ──► trading logic
[infinite stream]    if t == 'AAPL')          (processes one
                    [lazy filter]              tick at a time)
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">market_ticker</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Simulates an infinite stream of market data.</span><span class="sh">"""</span>
    <span class="kn">import</span> <span class="n">random</span><span class="p">,</span> <span class="n">itertools</span>
    <span class="n">symbols</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">AAPL</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">GOOG</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">MSFT</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">AMZN</span><span class="sh">'</span><span class="p">]</span>
    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="n">itertools</span><span class="p">.</span><span class="nf">count</span><span class="p">():</span>
        <span class="k">yield</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">symbol</span><span class="sh">'</span><span class="p">:</span> <span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">symbols</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">:</span> <span class="nf">round</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">uniform</span><span class="p">(</span><span class="mi">100</span><span class="p">,</span> <span class="mi">300</span><span class="p">),</span> <span class="mi">2</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">volume</span><span class="sh">'</span><span class="p">:</span> <span class="n">random</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">100</span><span class="p">,</span> <span class="mi">10000</span><span class="p">),</span>
        <span class="p">}</span>

<span class="c1"># Build the pipeline — nothing executes yet
</span><span class="n">ticker</span>    <span class="o">=</span> <span class="nf">market_ticker</span><span class="p">()</span>                             <span class="c1"># Source: infinite generator
</span><span class="n">aapl_only</span> <span class="o">=</span> <span class="p">(</span><span class="n">t</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">ticker</span> <span class="k">if</span> <span class="n">t</span><span class="p">[</span><span class="sh">'</span><span class="s">symbol</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="sh">'</span><span class="s">AAPL</span><span class="sh">'</span><span class="p">)</span> <span class="c1"># Filter: lazy expression
</span>
<span class="c1"># Data flows ONLY when we pull from the end
</span><span class="n">tick</span> <span class="o">=</span> <span class="nf">next</span><span class="p">(</span><span class="n">aapl_only</span><span class="p">)</span>  <span class="c1"># NOW it runs: pulls from ticker until it finds AAPL
</span><span class="nf">print</span><span class="p">(</span><span class="n">tick</span><span class="p">)</span>  <span class="c1"># {'symbol': 'AAPL', 'price': 172.34, 'volume': 4821}
</span></code></pre>

</div>



<p><strong>Nothing ran when we built the pipeline.</strong> No data was fetched, no filtering occurred. The entire chain is dormant until we call <code>next()</code>. This is <strong>lazy evaluation</strong>—the pipeline pulls data through only as fast as you consume it.</p>

<p>This is how you process a terabyte log file with 104 bytes of working memory.</p>




<h2>
  
  
  3. The Deep Dive: Generators as Coroutines
</h2>

<p>Everything above treats generators as <strong>producers</strong>: you pull data <em>out</em> of them via <code>next()</code>.</p>

<p>But generators can also be <strong>consumers</strong>: you push data <em>into</em> them via <code>.send()</code>. This transforms a generator from a simple stream into a <strong>stateful processing unit</strong>—what computer scientists call a <em>coroutine</em>.</p>

<h3>
  
  
  <code>yield</code> as an Expression
</h3>

<p>Normally, <code>yield value</code> is a statement—it sends a value out. But it can also be an expression that <em>receives</em> a value:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">accumulator</span><span class="p">():</span>
    <span class="n">total</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">value</span> <span class="o">=</span> <span class="p">(</span><span class="k">yield</span> <span class="n">total</span><span class="p">)</span>  <span class="c1"># Pause: send out total, wait to receive a value
</span>        <span class="k">if</span> <span class="n">value</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
            <span class="n">total</span> <span class="o">+=</span> <span class="n">value</span>

<span class="n">acc</span> <span class="o">=</span> <span class="nf">accumulator</span><span class="p">()</span>
<span class="nf">next</span><span class="p">(</span><span class="n">acc</span><span class="p">)</span>        <span class="c1"># Prime the coroutine (advance to first yield)
</span><span class="n">acc</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="mi">10</span><span class="p">)</span>     <span class="c1"># Push 10 in → total becomes 10
</span><span class="n">acc</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="mi">20</span><span class="p">)</span>     <span class="c1"># Push 20 in → total becomes 30
</span><span class="n">result</span> <span class="o">=</span> <span class="n">acc</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="mi">5</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">result</span><span class="p">)</span>    <span class="c1"># 35
</span></code></pre>

</div>



<p>The <strong>priming</strong> step (<code>next(acc)</code>) is required. A fresh generator is frozen at the <em>start</em> of the function, before any <code>yield</code> has been reached. You must advance it to the first <code>yield</code> before you can send anything to it.</p>

<h3>
  
  
  The Three Generator Controls
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Operation</th>
<th>Syntax</th>
<th>What it does</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Pull</strong></td>
<td><code>next(gen)</code></td>
<td>Resume, run until next <code>yield</code>, return yielded value</td>
</tr>
<tr>
<td><strong>Push</strong></td>
<td><code>gen.send(val)</code></td>
<td>Resume with <code>val</code> as the result of <code>yield</code>, run until next <code>yield</code>
</td>
</tr>
<tr>
<td><strong>Throw</strong></td>
<td><code>gen.throw(ExcType)</code></td>
<td>Resume by raising an exception <em>at the yield point</em>
</td>
</tr>
<tr>
<td><strong>Close</strong></td>
<td><code>gen.close()</code></td>
<td>Throw <code>GeneratorExit</code>, shut down cleanly</td>
</tr>
</tbody>
</table></div>

<p>The <code>.throw()</code> method is particularly powerful. Instead of crashing your pipeline when bad data appears, you can inject the error directly at the coroutine's pause point and let it handle recovery internally.</p>

<h3>
  
  
  Building a Finite State Machine with <code>yield</code>
</h3>

<p>A coroutine's "current line number" <em>is</em> its state. No state variables. No <code>if state == "WATCHING"</code> branching at the top. The control flow itself encodes the state.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">enum</span> <span class="kn">import</span> <span class="n">Enum</span><span class="p">,</span> <span class="n">auto</span>

<span class="k">class</span> <span class="nc">BotState</span><span class="p">(</span><span class="n">Enum</span><span class="p">):</span>
    <span class="n">WATCHING</span> <span class="o">=</span> <span class="nf">auto</span><span class="p">()</span>
    <span class="n">ACTIVE</span>   <span class="o">=</span> <span class="nf">auto</span><span class="p">()</span>

<span class="k">def</span> <span class="nf">trading_bot</span><span class="p">(</span><span class="n">entry_threshold</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">150.0</span><span class="p">,</span>
                <span class="n">exit_threshold</span><span class="p">:</span>  <span class="nb">float</span> <span class="o">=</span> <span class="mf">200.0</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">
    A coroutine FSM with two states:
      WATCHING → waiting for a low price to enter a position
      ACTIVE   → holding a position, waiting to exit at profit
    </span><span class="sh">"""</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">[BOT] Initialized. State: WATCHING</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">entry_price</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">0.0</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="c1"># ── STATE: WATCHING ──────────────────────────────
</span>            <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
                <span class="n">price</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="p">(</span><span class="k">yield</span><span class="p">)</span>               <span class="c1"># Wait for next tick
</span>                <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[WATCHING] AAPL @ $</span><span class="si">{</span><span class="n">price</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
                <span class="k">if</span> <span class="n">price</span> <span class="o">&lt;=</span> <span class="n">entry_threshold</span><span class="p">:</span>
                    <span class="n">entry_price</span> <span class="o">=</span> <span class="n">price</span>
                    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[SIGNAL] Entry at $</span><span class="si">{</span><span class="n">entry_price</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="s"> → switching to ACTIVE</span><span class="sh">"</span><span class="p">)</span>
                    <span class="k">break</span>                            <span class="c1"># Transition to ACTIVE
</span>
            <span class="c1"># ── STATE: ACTIVE ────────────────────────────────
</span>            <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
                <span class="n">price</span> <span class="o">=</span> <span class="p">(</span><span class="k">yield</span><span class="p">)</span>                      <span class="c1"># Wait for next tick
</span>                <span class="n">pnl</span>   <span class="o">=</span> <span class="n">price</span> <span class="o">-</span> <span class="n">entry_price</span>
                <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[ACTIVE]   AAPL @ $</span><span class="si">{</span><span class="n">price</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="s"> | PnL: $</span><span class="si">{</span><span class="n">pnl</span><span class="si">:</span><span class="o">+</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
                <span class="k">if</span> <span class="n">price</span> <span class="o">&gt;=</span> <span class="n">exit_threshold</span><span class="p">:</span>
                    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[SIGNAL] Exit at $</span><span class="si">{</span><span class="n">price</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="s"> | Profit: $</span><span class="si">{</span><span class="n">pnl</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="s"> → switching to WATCHING</span><span class="sh">"</span><span class="p">)</span>
                    <span class="k">break</span>                            <span class="c1"># Transition back to WATCHING
</span>
        <span class="k">except</span> <span class="nb">ValueError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="c1"># Bad tick injected via .throw() — reset to WATCHING without crashing
</span>            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[ERROR] Bad data received: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="s">. Resetting to WATCHING.</span><span class="sh">"</span><span class="p">)</span>
            <span class="n">entry_price</span> <span class="o">=</span> <span class="mf">0.0</span>
            <span class="c1"># Loop continues: back to WATCHING state
</span></code></pre>

</div>






<h2>
  
  
  4. The Grand Finale: The High-Frequency Trading Bot
</h2>

<p>Let's wire everything together. Four components. One elegant pipeline.</p>

<h3>
  
  
  The Architecture
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────┐
│                    PIPELINE OVERVIEW                     │
│                                                          │
│  [Source]          [Filter]           [Sink]             │
│  market_ticker()──►aapl_stream ──────►trading_bot()      │
│  (Generator)       (Gen. Expression)  (Coroutine FSM)    │
│       │                  │                  │            │
│   Produces all       Passes only        Consumes AAPL   │
│   symbols lazily     AAPL ticks         ticks, manages  │
│                                         state internally │
│                                                          │
│                   [Bridge]                               │
│                   for loop with                          │
│                   .send() / .throw()                     │
└─────────────────────────────────────────────────────────┘
</code></pre>

</div>



<h3>
  
  
  The Complete System
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">random</span>
<span class="kn">import</span> <span class="n">itertools</span>
<span class="kn">import</span> <span class="n">sys</span>

<span class="c1"># ── COMPONENT 1: THE SOURCE ──────────────────────────────────────
</span><span class="k">def</span> <span class="nf">market_ticker</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Infinite stream of market ticks. Never terminates.</span><span class="sh">"""</span>
    <span class="n">symbols</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">AAPL</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">GOOG</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">MSFT</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">AMZN</span><span class="sh">'</span><span class="p">]</span>
    <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="n">itertools</span><span class="p">.</span><span class="nf">count</span><span class="p">():</span>
        <span class="k">yield</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">symbol</span><span class="sh">'</span><span class="p">:</span> <span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">symbols</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">:</span>  <span class="nf">round</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">uniform</span><span class="p">(</span><span class="mi">100</span><span class="p">,</span> <span class="mi">300</span><span class="p">),</span> <span class="mi">2</span><span class="p">),</span>
        <span class="p">}</span>

<span class="c1"># ── COMPONENT 2: THE FILTER ──────────────────────────────────────
</span><span class="k">def</span> <span class="nf">build_pipeline</span><span class="p">():</span>
    <span class="n">ticker</span>    <span class="o">=</span> <span class="nf">market_ticker</span><span class="p">()</span>
    <span class="c1"># Occasionally inject a bad tick to test error handling
</span>    <span class="n">aapl_stream</span> <span class="o">=</span> <span class="p">(</span>
        <span class="n">t</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">ticker</span>
        <span class="k">if</span> <span class="n">t</span><span class="p">[</span><span class="sh">'</span><span class="s">symbol</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="sh">'</span><span class="s">AAPL</span><span class="sh">'</span>
    <span class="p">)</span>
    <span class="k">return</span> <span class="n">aapl_stream</span>

<span class="c1"># ── COMPONENT 3: THE CONSUMER (FSM Coroutine) ────────────────────
# (trading_bot as defined in Section 3 above)
</span>
<span class="c1"># ── COMPONENT 4: THE BRIDGE ──────────────────────────────────────
</span><span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">tick_limit</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">30</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
    <span class="n">stream</span> <span class="o">=</span> <span class="nf">build_pipeline</span><span class="p">()</span>
    <span class="n">bot</span>    <span class="o">=</span> <span class="nf">trading_bot</span><span class="p">(</span><span class="n">entry_threshold</span><span class="o">=</span><span class="mf">150.0</span><span class="p">,</span> <span class="n">exit_threshold</span><span class="o">=</span><span class="mf">200.0</span><span class="p">)</span>

    <span class="c1"># Prime the coroutine — advance it to the first yield
</span>    <span class="nf">next</span><span class="p">(</span><span class="n">bot</span><span class="p">)</span>

    <span class="n">ticks_processed</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="k">for</span> <span class="n">tick</span> <span class="ow">in</span> <span class="n">stream</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">ticks_processed</span> <span class="o">&gt;=</span> <span class="n">tick_limit</span><span class="p">:</span>
            <span class="k">break</span>

        <span class="n">price</span> <span class="o">=</span> <span class="n">tick</span><span class="p">[</span><span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">]</span>

        <span class="c1"># Simulate occasional bad data (1-in-10 chance)
</span>        <span class="k">if</span> <span class="n">random</span><span class="p">.</span><span class="nf">random</span><span class="p">()</span> <span class="o">&lt;</span> <span class="mf">0.1</span><span class="p">:</span>
            <span class="n">bad_price</span> <span class="o">=</span> <span class="o">-</span><span class="nf">abs</span><span class="p">(</span><span class="n">price</span><span class="p">)</span>  <span class="c1"># Corrupt tick: negative price
</span>            <span class="k">try</span><span class="p">:</span>
                <span class="n">bot</span><span class="p">.</span><span class="nf">throw</span><span class="p">(</span><span class="nc">ValueError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Negative price: </span><span class="si">{</span><span class="n">bad_price</span><span class="si">}</span><span class="sh">"</span><span class="p">))</span>
                <span class="nf">next</span><span class="p">(</span><span class="n">bot</span><span class="p">)</span>            <span class="c1"># Re-prime after error recovery
</span>            <span class="k">except</span> <span class="nb">StopIteration</span><span class="p">:</span>
                <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">[BRIDGE] Bot shut down during error recovery.</span><span class="sh">"</span><span class="p">)</span>
                <span class="k">break</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">try</span><span class="p">:</span>
                <span class="n">bot</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="n">price</span><span class="p">)</span>      <span class="c1"># Normal operation: push price to bot
</span>            <span class="k">except</span> <span class="nb">StopIteration</span><span class="p">:</span>
                <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">[BRIDGE] Bot has shut down.</span><span class="sh">"</span><span class="p">)</span>
                <span class="k">break</span>

        <span class="n">ticks_processed</span> <span class="o">+=</span> <span class="mi">1</span>

    <span class="n">bot</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>  <span class="c1"># Send GeneratorExit — clean shutdown
</span>    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">[BRIDGE] Pipeline complete. Processed </span><span class="si">{</span><span class="n">ticks_processed</span><span class="si">}</span><span class="s"> AAPL ticks.</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">run</span><span class="p">(</span><span class="n">tick_limit</span><span class="o">=</span><span class="mi">30</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Why <code>.throw()</code> is the Pro Pattern
</h3>

<p>Most tutorials show <code>.send()</code> and call it a day. But <code>.throw()</code> is what makes a coroutine-based FSM <strong>production-grade</strong>.</p>

<p>The alternative to <code>.throw()</code> is sentinel values:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Naive approach: use magic values to signal errors
</span><span class="n">bot</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span>  <span class="c1"># Hope the bot understands -1 means "bad data"
</span></code></pre>

</div>



<p>This is fragile. It poisons your data channel with control signals. What if -1 is a legitimate (if unusual) price? What if you need to distinguish between different error types?</p>

<p><code>.throw()</code> keeps the error channel and the data channel <strong>separate</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># The Bridge: clear separation of concerns
</span><span class="k">for</span> <span class="n">tick</span> <span class="ow">in</span> <span class="n">stream</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">tick</span><span class="p">[</span><span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">]</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">:</span>
        <span class="n">bot</span><span class="p">.</span><span class="nf">throw</span><span class="p">(</span><span class="nc">ValueError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Corrupt tick: </span><span class="si">{</span><span class="n">tick</span><span class="si">}</span><span class="sh">"</span><span class="p">))</span>  <span class="c1"># Error channel
</span>    <span class="k">else</span><span class="p">:</span>
        <span class="n">bot</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="n">tick</span><span class="p">[</span><span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">])</span>                          <span class="c1"># Data channel
</span></code></pre>

</div>



<p>The coroutine catches it in a <code>try/except</code> at its yield point—exactly like a normal function. The state machine resets cleanly. The pipeline keeps running. Zero sentinel values. Zero ambiguity.</p>




<h2>
  
  
  Conclusion: Why This Matters
</h2>

<p>Let's close with three concrete reasons this mental model changes how you write code.</p>

<h3>
  
  
  1. Memory Efficiency: O(1) Space for Infinite Streams
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># This processes a 10GB log file in constant memory
</span><span class="k">def</span> <span class="nf">find_errors</span><span class="p">(</span><span class="n">path</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">path</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="k">yield</span> <span class="nf">from </span><span class="p">(</span><span class="n">line</span> <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">f</span> <span class="k">if</span> <span class="sh">"</span><span class="s">ERROR</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">line</span><span class="p">)</span>

<span class="k">for</span> <span class="n">error_line</span> <span class="ow">in</span> <span class="nf">find_errors</span><span class="p">(</span><span class="sh">"</span><span class="s">application.log</span><span class="sh">"</span><span class="p">):</span>
    <span class="nf">alert</span><span class="p">(</span><span class="n">error_line</span><span class="p">)</span>
</code></pre>

</div>



<p>No list. No <code>.readlines()</code>. A single line lives in memory at a time.</p>

<h3>
  
  
  2. State Management: Your Line Number Is Your State
</h3>

<p>The <code>trading_bot()</code> coroutine has zero explicit state variables for its FSM transitions. The <code>while True</code> loop it's currently executing in <em>is</em> the state. Python's own call stack manages it.</p>

<p>Compare that to the class-based alternative:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># The non-generator version: manual state management
</span><span class="k">class</span> <span class="nc">TradingBot</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">state</span> <span class="o">=</span> <span class="sh">"</span><span class="s">WATCHING</span><span class="sh">"</span>    <span class="c1"># Explicit state
</span>        <span class="n">self</span><span class="p">.</span><span class="n">entry_price</span> <span class="o">=</span> <span class="mf">0.0</span>

    <span class="k">def</span> <span class="nf">process</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">price</span><span class="p">:</span> <span class="nb">float</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">state</span> <span class="o">==</span> <span class="sh">"</span><span class="s">WATCHING</span><span class="sh">"</span><span class="p">:</span>   <span class="c1"># State checks everywhere
</span>            <span class="bp">...</span>
        <span class="k">elif</span> <span class="n">self</span><span class="p">.</span><span class="n">state</span> <span class="o">==</span> <span class="sh">"</span><span class="s">ACTIVE</span><span class="sh">"</span><span class="p">:</span>
            <span class="bp">...</span>
</code></pre>

</div>



<p>More code. More surface area for bugs. More branching to read and maintain.</p>

<h3>
  
  
  3. Composability: UNIX Pipes for Your Data
</h3>

<p>Each component in our pipeline does exactly one thing: the source generates ticks, the filter screens symbols, the bot manages trades. They're connected by convention—the iterator protocol—not by inheritance or tight coupling.</p>

<p>You can swap any component without touching the others:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Swap source: real broker API instead of random data
</span><span class="n">ticker</span> <span class="o">=</span> <span class="n">broker_api</span><span class="p">.</span><span class="nf">stream</span><span class="p">()</span>           <span class="c1"># Same interface, different source
</span>
<span class="c1"># Swap filter: multiple symbols
</span><span class="n">stream</span> <span class="o">=</span> <span class="p">(</span><span class="n">t</span> <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">ticker</span> <span class="k">if</span> <span class="n">t</span><span class="p">[</span><span class="sh">'</span><span class="s">symbol</span><span class="sh">'</span><span class="p">]</span> <span class="ow">in</span> <span class="p">{</span><span class="sh">'</span><span class="s">AAPL</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">MSFT</span><span class="sh">'</span><span class="p">})</span>

<span class="c1"># Swap sink: logging bot instead of trading bot
</span><span class="n">bot</span> <span class="o">=</span> <span class="nf">audit_logger</span><span class="p">(</span><span class="n">output</span><span class="o">=</span><span class="sh">"</span><span class="s">trades.log</span><span class="sh">"</span><span class="p">)</span>  <span class="c1"># Same .send() interface
</span></code></pre>

</div>



<p>Small tools. Single responsibilities. Glued by protocol.</p>

