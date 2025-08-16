---
Title: Floyd-Warshall's Algorithm C++: Story
Description: 
Author: Harsh Mishra
Date: 2025-08-16T20:55:38.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🌌 <em>The Conclave of All Roads</em> — The Floyd-Warshall Chronicles
</h2>

<blockquote>
<p><em>"In the Age of a Thousand Paths, no traveler could truly know the shortest road until every rumor was tested against every road, from every land to every land."</em><br>
— <em>Annals of the Great Cartographer</em></p>
</blockquote>




<h3>
  
  
  🏰 The World Before Maps
</h3>

<p>The world was vast — <strong>n kingdoms</strong> scattered across valleys, mountains, and rivers.<br>
Every kingdom knew of the others, but only through <strong>fragmented tales of roads</strong>:</p>

<ul>
<li><em>"From my town to yours, it takes 5 days."</em></li>
<li><em>"I heard that through the mountain pass via K, it might be faster."</em></li>
</ul>

<p>But no one <strong>truly knew</strong> the best way to travel from <em>every kingdom</em> to <em>every other kingdom</em>.</p>

<p>So the <strong>High Conclave of the Great Mapmakers</strong> convened to settle the matter once and for all.</p>

<p>They would compare <strong>every pair of kingdoms (i, j)</strong> — but in a methodical, almost mystical way.</p>




<h3>
  
  
  📜 The Sacred Ledger of Distances
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">&lt;iostream&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;vector&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;limits&gt;</span><span class="cp">
</span><span class="k">using</span> <span class="k">namespace</span> <span class="n">std</span><span class="p">;</span>

<span class="k">const</span> <span class="kt">int</span> <span class="n">INF</span> <span class="o">=</span> <span class="mf">1e9</span><span class="p">;</span>
</code></pre>

</div>



<p>The Mapmakers first declared:<br>
<strong>Any road unknown shall be called <em>Infinite</em>.</strong><br>
If no known path exists, they record <code>INF</code> — a polite way of saying:</p>

<blockquote>
<p><em>"Impossible to travel… unless fate changes."</em></p>
</blockquote>




<h3>
  
  
  🗺️ The Conclave Hall
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">class</span> <span class="nc">Graph</span> <span class="p">{</span>
    <span class="kt">int</span> <span class="n">n</span><span class="p">;</span>
    <span class="n">vector</span><span class="o">&lt;</span><span class="n">vector</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;&gt;</span> <span class="n">dist</span><span class="p">;</span>
<span class="nl">public:</span>
    <span class="n">Graph</span><span class="p">(</span><span class="kt">int</span> <span class="n">n</span><span class="p">)</span> <span class="o">:</span> <span class="n">n</span><span class="p">(</span><span class="n">n</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">dist</span><span class="p">.</span><span class="n">assign</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">vector</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">INF</span><span class="p">));</span>
        <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="n">n</span><span class="p">;</span> <span class="n">i</span><span class="o">++</span><span class="p">)</span>
            <span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="n">i</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="c1">// Zero distance to self</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>The Great Hall of the Mapmakers is square — <strong>n by n</strong> tables.<br>
On table <code>(i, j)</code> lies the current known shortest distance from Kingdom <em>i</em> to Kingdom <em>j</em>.</p>

<p>At the start:</p>

<ul>
<li>0 for traveling to oneself.</li>
<li>
<code>INF</code> for unknown paths.</li>
<li>Any known road will be filled in.</li>
</ul>




<h3>
  
  
  🚪 Carving Known Roads
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="kt">void</span> <span class="nf">addEdge</span><span class="p">(</span><span class="kt">int</span> <span class="n">u</span><span class="p">,</span> <span class="kt">int</span> <span class="n">v</span><span class="p">,</span> <span class="kt">int</span> <span class="n">w</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">dist</span><span class="p">[</span><span class="n">u</span><span class="p">][</span><span class="n">v</span><span class="p">]</span> <span class="o">=</span> <span class="n">w</span><span class="p">;</span> <span class="c1">// Direct known road</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Whenever a messenger brings news —<br>
<em>"There is a direct road from U to V, taking W days"</em> —<br>
the scribes write it directly into the ledger.</p>

<p>No arguments. This is the truth… <strong>for now</strong>.</p>




<h3>
  
  
  🔮 The Threefold Ritual
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="kt">void</span> <span class="nf">floydWarshall</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">k</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">k</span> <span class="o">&lt;</span> <span class="n">n</span><span class="p">;</span> <span class="n">k</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="n">n</span><span class="p">;</span> <span class="n">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">j</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">j</span> <span class="o">&lt;</span> <span class="n">n</span><span class="p">;</span> <span class="n">j</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
                    <span class="k">if</span> <span class="p">(</span><span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="n">k</span><span class="p">]</span> <span class="o">&lt;</span> <span class="n">INF</span> <span class="o">&amp;&amp;</span> <span class="n">dist</span><span class="p">[</span><span class="n">k</span><span class="p">][</span><span class="n">j</span><span class="p">]</span> <span class="o">&lt;</span> <span class="n">INF</span><span class="p">)</span>
                        <span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="n">j</span><span class="p">]</span> <span class="o">=</span> <span class="n">min</span><span class="p">(</span><span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="n">j</span><span class="p">],</span> <span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="n">k</span><span class="p">]</span> <span class="o">+</span> <span class="n">dist</span><span class="p">[</span><span class="n">k</span><span class="p">][</span><span class="n">j</span><span class="p">]);</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>The Conclave begins the <strong>Threefold Ritual</strong>:</p>

<ol>
<li><p><strong>Outer Loop — The Gatekeeper <code>k</code>:</strong><br>
Imagine opening the gates of one kingdom <code>k</code> at a time and asking:<br>
<em>"If we allowed travelers to pass through k, could we improve any journey?"</em><br>
That’s why <code>k</code> is <strong>outermost</strong> — it represents the "current allowed checkpoint" the mapmakers are testing.</p></li>
<li><p><strong>Middle Loop — The Start <code>i</code>:</strong><br>
For each starting kingdom <code>i</code>, they ask:<br>
<em>"What if we start here, and travel through k?"</em></p></li>
<li><p><strong>Inner Loop — The End <code>j</code>:</strong><br>
For each destination <code>j</code>, they check:<br>
<em>"Would going i → k → j be shorter than our currently recorded i → j?"</em></p></li>
</ol>

<p>They test <strong>all combinations</strong>, like a council testing every rumor, every gossip, every possible detour through k.</p>

<p>If <code>dist[i][k] + dist[k][j]</code> is better than the existing <code>dist[i][j]</code>, they <strong>update it</strong>.<br>
The scroll is rewritten — history is changed.</p>




<h3>
  
  
  📖 The Final Map
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="kt">void</span> <span class="nf">printDistances</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="n">n</span><span class="p">;</span> <span class="n">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">j</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">j</span> <span class="o">&lt;</span> <span class="n">n</span><span class="p">;</span> <span class="n">j</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">if</span> <span class="p">(</span><span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="n">j</span><span class="p">]</span> <span class="o">==</span> <span class="n">INF</span><span class="p">)</span> <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"INF "</span><span class="p">;</span>
                <span class="k">else</span> <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="n">dist</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="n">j</span><span class="p">]</span> <span class="o">&lt;&lt;</span> <span class="s">" "</span><span class="p">;</span>
            <span class="p">}</span>
            <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"</span><span class="se">\n</span><span class="s">"</span><span class="p">;</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>At the end of the ritual, the scribes unroll the grand map:</p>

<ul>
<li>If it says a number, that’s the fastest possible journey known.</li>
<li>If it says <code>INF</code>, the kingdoms are separated by oceans of impossibility.</li>
</ul>




<h3>
  
  
  🧪 The Day of Truth
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">Graph</span> <span class="n">g</span><span class="p">(</span><span class="mi">4</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">5</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>
    <span class="n">g</span><span class="p">.</span><span class="n">addEdge</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>

    <span class="n">g</span><span class="p">.</span><span class="n">floydWarshall</span><span class="p">();</span>
    <span class="n">g</span><span class="p">.</span><span class="n">printDistances</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p>They begin with four kingdoms, connected by partial roads.</p>

<p>The ritual begins:</p>

<ul>
<li>First, open the gates of Kingdom 0 (<code>k=0</code>), test all paths.</li>
<li>Then open Kingdom 1 (<code>k=1</code>), test again.</li>
<li>Continue until <strong>every kingdom has been the “through point” once</strong>.</li>
</ul>

<p>By the end, no rumor remains unchecked.<br>
The shortest path from <em>everywhere to everywhere</em> is known, and the Age of Uncertainty ends.</p>




<h3>
  
  
  ⚡ Why You’ll Remember This
</h3>

<ul>
<li>
<strong>k outermost</strong> — because you pick one kingdom at a time to act as an allowed "middle stop" for all pairs.</li>
<li>
<strong>i middle</strong> — starting point.</li>
<li>
<strong>j innermost</strong> — ending point.</li>
<li>The process is <strong>like gradually lifting travel bans</strong> on kingdoms one by one and checking if that unlocks shorter routes.</li>
</ul>

<p>When you’re in an interview and they ask about Floyd-Warshall, picture the <strong>Conclave</strong>:<br>
the giant n×n hall of tables, the Threefold Ritual, the Gatekeeper k,<br>
and the slow unveiling of the true, final map of the world.</p>

