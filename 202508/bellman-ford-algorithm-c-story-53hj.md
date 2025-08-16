---
Title: Bellman-Ford Algorithm C++: Story
Description: 
Author: Harsh Mishra
Date: 2025-08-16T20:57:41.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  ⚔️ <em>The Caravan Through the Lands of Shadows</em> — The Bellman-Ford Saga
</h2>

<blockquote>
<p><em>"In a land where roads could twist in deceit, the fastest path was not always the one that looked shortest…"</em><br>
— <em>Chronicles of the Desert Traders</em></p>
</blockquote>




<h3>
  
  
  🏜️ The Kingdoms of Sand
</h3>

<p>There were <strong>n cities</strong> scattered across the desert.<br>
Merchants traveled between them using roads — each road had a toll, sometimes fair, sometimes cruel, and sometimes <em>negative</em> when a generous city paid travelers to pass through.</p>

<p>But the desert was treacherous: <strong>some paths would loop forever, endlessly making profit</strong> — cursed loops, they called them.<br>
The Guild of Desert Traders needed a way to <strong>find the shortest path from a starting city</strong> to all others — and to know if any cursed loop existed.</p>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">&lt;iostream&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;vector&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;limits&gt;</span><span class="cp">
</span><span class="k">using</span> <span class="k">namespace</span> <span class="n">std</span><span class="p">;</span>

<span class="k">struct</span> <span class="nc">Edge</span> <span class="p">{</span>
    <span class="kt">int</span> <span class="n">u</span><span class="p">,</span> <span class="n">v</span><span class="p">,</span> <span class="n">w</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</div>



<p>The traders recorded every known road in a scroll of <strong>edges</strong>:</p>

<ul>
<li>
<code>u</code> — the city where the road began</li>
<li>
<code>v</code> — the city where it ended</li>
<li>
<code>w</code> — the toll or reward for using it</li>
</ul>




<h3>
  
  
  📜 The Guild Ledger
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">class</span> <span class="nc">BellmanFord</span> <span class="p">{</span>
    <span class="kt">int</span> <span class="n">n</span><span class="p">;</span>
    <span class="n">vector</span><span class="o">&lt;</span><span class="n">Edge</span><span class="o">&gt;</span> <span class="n">edges</span><span class="p">;</span>
    <span class="n">vector</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;</span> <span class="n">dist</span><span class="p">;</span>
    <span class="k">const</span> <span class="kt">int</span> <span class="n">INF</span> <span class="o">=</span> <span class="n">numeric_limits</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;::</span><span class="n">max</span><span class="p">();</span>
<span class="nl">public:</span>
    <span class="n">BellmanFord</span><span class="p">(</span><span class="kt">int</span> <span class="n">n</span><span class="p">)</span> <span class="o">:</span> <span class="n">n</span><span class="p">(</span><span class="n">n</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">dist</span><span class="p">.</span><span class="n">assign</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">INF</span><span class="p">);</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>The Guild kept a <strong>ledger of distances</strong> from the starting city to all others.<br>
At the start, all distances were <strong>infinite</strong> — for no one knew a way yet.</p>




<h3>
  
  
  🚪 Carving the Roads
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="kt">void</span> <span class="nf">addEdge</span><span class="p">(</span><span class="kt">int</span> <span class="n">u</span><span class="p">,</span> <span class="kt">int</span> <span class="n">v</span><span class="p">,</span> <span class="kt">int</span> <span class="n">w</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">edges</span><span class="p">.</span><span class="n">push_back</span><span class="p">({</span><span class="n">u</span><span class="p">,</span> <span class="n">v</span><span class="p">,</span> <span class="n">w</span><span class="p">});</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Every time a caravan returned with news of a road, it was carved into the scroll:<br>
<em>"From U to V, the toll is W."</em></p>




<h3>
  
  
  🏁 The Departure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="kt">void</span> <span class="nf">shortestPath</span><span class="p">(</span><span class="kt">int</span> <span class="n">src</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">dist</span><span class="p">[</span><span class="n">src</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
</code></pre>

</div>



<p>The Guildmaster began in the chosen starting city <code>src</code>.<br>
His distance to himself was <strong>0</strong> — no toll to stand where you already are.</p>




<h3>
  
  
  🔄 The n-1 Journeys
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>        <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;=</span> <span class="n">n</span> <span class="o">-</span> <span class="mi">1</span><span class="p">;</span> <span class="n">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">for</span> <span class="p">(</span><span class="k">auto</span> <span class="o">&amp;</span><span class="n">e</span> <span class="o">:</span> <span class="n">edges</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">if</span> <span class="p">(</span><span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">u</span><span class="p">]</span> <span class="o">!=</span> <span class="n">INF</span> <span class="o">&amp;&amp;</span> <span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">u</span><span class="p">]</span> <span class="o">+</span> <span class="n">e</span><span class="p">.</span><span class="n">w</span> <span class="o">&lt;</span> <span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">v</span><span class="p">])</span> <span class="p">{</span>
                    <span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">v</span><span class="p">]</span> <span class="o">=</span> <span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">u</span><span class="p">]</span> <span class="o">+</span> <span class="n">e</span><span class="p">.</span><span class="n">w</span><span class="p">;</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">}</span>
</code></pre>

</div>



<p>The caravans then began <strong>n-1 great journeys</strong> across the desert.</p>

<p>Why <code>n-1</code>?<br>
Because in the worst case, the shortest path to a city might have to pass through <strong>every other city exactly once</strong> — and each journey could only improve distances by one road at a time.</p>

<p>On each journey:</p>

<ul>
<li>The caravans looked at <strong>every road</strong> in the scroll.</li>
<li>If they could reach the starting point of a road (<code>dist[e.u] != INF</code>)
and taking that road improved the distance to its end city (<code>dist[e.u] + e.w &lt; dist[e.v]</code>),
they updated the ledger.</li>
</ul>

<p>Each improvement was like a rumor spreading: <em>"There's a faster way, through here!"</em></p>




<h3>
  
  
  🕳️ The Shadow Loops
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>        <span class="k">for</span> <span class="p">(</span><span class="k">auto</span> <span class="o">&amp;</span><span class="n">e</span> <span class="o">:</span> <span class="n">edges</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">u</span><span class="p">]</span> <span class="o">!=</span> <span class="n">INF</span> <span class="o">&amp;&amp;</span> <span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">u</span><span class="p">]</span> <span class="o">+</span> <span class="n">e</span><span class="p">.</span><span class="n">w</span> <span class="o">&lt;</span> <span class="n">dist</span><span class="p">[</span><span class="n">e</span><span class="p">.</span><span class="n">v</span><span class="p">])</span> <span class="p">{</span>
                <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"Negative cycle detected</span><span class="se">\n</span><span class="s">"</span><span class="p">;</span>
                <span class="k">return</span><span class="p">;</span>
            <span class="p">}</span>
        <span class="p">}</span>
</code></pre>

</div>



<p>When the <strong>n-1 journeys</strong> ended, the Guildmaster sent scouts for one final check.</p>

<p>If <strong>any road</strong> could still improve a distance, it meant they had found a <strong>shadow loop</strong> — a cursed cycle where merchants could ride forever, endlessly decreasing their toll cost, and break the economy.</p>

<p>The Guild marked such roads as forbidden and ended the expedition.</p>




<h3>
  
  
  📖 The Ledger of Truth
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>        <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="n">n</span><span class="p">;</span> <span class="n">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">]</span> <span class="o">==</span> <span class="n">INF</span><span class="p">)</span> <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"INF "</span><span class="p">;</span>
            <span class="k">else</span> <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">]</span> <span class="o">&lt;&lt;</span> <span class="s">" "</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"</span><span class="se">\n</span><span class="s">"</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>If no cursed loop was found, the Guild unrolled the final ledger:</p>

<ul>
<li>If the number was finite, it was the shortest possible toll to reach that city.</li>
<li>If <code>INF</code>, the city was unreachable — lost beyond the dunes.</li>
</ul>




<h3>
  
  
  🧪 The Day of the Caravan
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">BellmanFord</span> <span class="n">g</span><span class="p">(</span><span class="mi">5</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">4</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">5</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="o">-</span><span class="mi">3</span><span class="p">);</span>

    <span class="n">g</span><span class="p">.</span><span class="n">shortestPath</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The expedition began in City 0.<br>
Caravans rode for <code>n-1</code> journeys, spreading rumors of faster paths, until all truth was known.</p>

<p>And in the end, the Guild could travel the desert <strong>without fear of deception — unless a shadow loop still lurked in the dunes</strong>.</p>

