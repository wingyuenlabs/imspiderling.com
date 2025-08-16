---
Title: Dijkstra's Algorithm C++: Story
Description: 
Author: Harsh Mishra
Date: 2025-08-16T20:58:49.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🏹 <em>The Beacon Riders of the Northern Realms</em> — The Dijkstra Saga
</h2>

<blockquote>
<p><em>"In the kingdom where roads shimmered with frost, the fastest rider was the one who chose his next step with care."</em><br>
— <em>Songs of the Winter Messenger</em></p>
</blockquote>




<p>The Northern Realms were a land of <strong>villages connected by winding frozen roads</strong>, each road taking a certain amount of time to travel.<br>
The High Council needed to send messengers from the <strong>capital</strong> to every other village, carrying news of an approaching winter storm.<br>
But there was a rule: <strong>messengers must always choose the currently closest unexplored village to visit next</strong>, so no time would be wasted wandering far before checking nearer places.</p>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">&lt;iostream&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;vector&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;queue&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;limits&gt;</span><span class="cp">
</span><span class="k">using</span> <span class="k">namespace</span> <span class="n">std</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">Dijkstra</span> <span class="p">{</span>
    <span class="kt">int</span> <span class="n">n</span><span class="p">;</span>
    <span class="n">vector</span><span class="o">&lt;</span><span class="n">vector</span><span class="o">&lt;</span><span class="n">pair</span><span class="o">&lt;</span><span class="kt">int</span><span class="p">,</span> <span class="kt">int</span><span class="o">&gt;&gt;&gt;</span> <span class="n">adj</span><span class="p">;</span>
    <span class="n">vector</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;</span> <span class="n">dist</span><span class="p">;</span>
    <span class="k">const</span> <span class="kt">int</span> <span class="n">INF</span> <span class="o">=</span> <span class="n">numeric_limits</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;::</span><span class="n">max</span><span class="p">();</span>
<span class="nl">public:</span>
    <span class="n">Dijkstra</span><span class="p">(</span><span class="kt">int</span> <span class="n">n</span><span class="p">)</span> <span class="o">:</span> <span class="n">n</span><span class="p">(</span><span class="n">n</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">adj</span><span class="p">.</span><span class="n">resize</span><span class="p">(</span><span class="n">n</span><span class="p">);</span>
        <span class="n">dist</span><span class="p">.</span><span class="n">assign</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">INF</span><span class="p">);</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>The kingdom was drawn as <strong>n villages</strong> on a great frost-map.<br>
Beside each village lay a ledger of roads: <code>adj[u]</code> kept track of every road from that village, written as <code>(neighbor, time_cost)</code>.<br>
<code>dist[]</code> was the parchment where the <strong>shortest known times</strong> to reach each village would be recorded.</p>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="kt">void</span> <span class="nf">addRoad</span><span class="p">(</span><span class="kt">int</span> <span class="n">u</span><span class="p">,</span> <span class="kt">int</span> <span class="n">v</span><span class="p">,</span> <span class="kt">int</span> <span class="n">w</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">adj</span><span class="p">[</span><span class="n">u</span><span class="p">].</span><span class="n">push_back</span><span class="p">({</span><span class="n">v</span><span class="p">,</span> <span class="n">w</span><span class="p">});</span>
        <span class="n">adj</span><span class="p">[</span><span class="n">v</span><span class="p">].</span><span class="n">push_back</span><span class="p">({</span><span class="n">u</span><span class="p">,</span> <span class="n">w</span><span class="p">});</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Every road was carved into the map:<br>
<em>"From U to V, the ride takes W hours."</em><br>
And because roads in the Northern Realms could be traveled in both directions, the same note was made for the reverse path.</p>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>    <span class="kt">void</span> <span class="nf">shortestPaths</span><span class="p">(</span><span class="kt">int</span> <span class="n">src</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">dist</span><span class="p">[</span><span class="n">src</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
        <span class="n">priority_queue</span><span class="o">&lt;</span><span class="n">pair</span><span class="o">&lt;</span><span class="kt">int</span><span class="p">,</span> <span class="kt">int</span><span class="o">&gt;</span><span class="p">,</span> <span class="n">vector</span><span class="o">&lt;</span><span class="n">pair</span><span class="o">&lt;</span><span class="kt">int</span><span class="p">,</span> <span class="kt">int</span><span class="o">&gt;&gt;</span><span class="p">,</span> <span class="n">greater</span><span class="o">&lt;</span><span class="n">pair</span><span class="o">&lt;</span><span class="kt">int</span><span class="p">,</span> <span class="kt">int</span><span class="o">&gt;&gt;&gt;</span> <span class="n">pq</span><span class="p">;</span>
        <span class="n">pq</span><span class="p">.</span><span class="n">push</span><span class="p">({</span><span class="mi">0</span><span class="p">,</span> <span class="n">src</span><span class="p">});</span>
</code></pre>

</div>



<p>The messengers began at the <strong>capital</strong> (<code>src</code>).<br>
The time to reach the capital itself was <strong>0</strong>.<br>
A magical beacon-fire, represented by a <strong>priority queue</strong>, was lit — its flames always showing <strong>the next closest village yet to be visited</strong>.</p>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>        <span class="k">while</span> <span class="p">(</span><span class="o">!</span><span class="n">pq</span><span class="p">.</span><span class="n">empty</span><span class="p">())</span> <span class="p">{</span>
            <span class="kt">int</span> <span class="n">d</span> <span class="o">=</span> <span class="n">pq</span><span class="p">.</span><span class="n">top</span><span class="p">().</span><span class="n">first</span><span class="p">;</span>
            <span class="kt">int</span> <span class="n">u</span> <span class="o">=</span> <span class="n">pq</span><span class="p">.</span><span class="n">top</span><span class="p">().</span><span class="n">second</span><span class="p">;</span>
            <span class="n">pq</span><span class="p">.</span><span class="n">pop</span><span class="p">();</span>
</code></pre>

</div>



<p>At each step, the beacon’s flame called forth the <strong>nearest unexplored village</strong>.<br>
The chosen village’s name was read, and the messenger rode there next.</p>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>            <span class="k">if</span> <span class="p">(</span><span class="n">d</span> <span class="o">&gt;</span> <span class="n">dist</span><span class="p">[</span><span class="n">u</span><span class="p">])</span> <span class="k">continue</span><span class="p">;</span>
</code></pre>

</div>



<p>If a messenger arrived and found that <strong>a faster messenger had already reached this village before</strong>, they turned around without delay — no need to waste effort on a place already warned sooner.</p>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code>            <span class="k">for</span> <span class="p">(</span><span class="k">auto</span> <span class="o">&amp;</span><span class="n">edge</span> <span class="o">:</span> <span class="n">adj</span><span class="p">[</span><span class="n">u</span><span class="p">])</span> <span class="p">{</span>
                <span class="kt">int</span> <span class="n">v</span> <span class="o">=</span> <span class="n">edge</span><span class="p">.</span><span class="n">first</span><span class="p">;</span>
                <span class="kt">int</span> <span class="n">w</span> <span class="o">=</span> <span class="n">edge</span><span class="p">.</span><span class="n">second</span><span class="p">;</span>
                <span class="k">if</span> <span class="p">(</span><span class="n">dist</span><span class="p">[</span><span class="n">u</span><span class="p">]</span> <span class="o">+</span> <span class="n">w</span> <span class="o">&lt;</span> <span class="n">dist</span><span class="p">[</span><span class="n">v</span><span class="p">])</span> <span class="p">{</span>
                    <span class="n">dist</span><span class="p">[</span><span class="n">v</span><span class="p">]</span> <span class="o">=</span> <span class="n">dist</span><span class="p">[</span><span class="n">u</span><span class="p">]</span> <span class="o">+</span> <span class="n">w</span><span class="p">;</span>
                    <span class="n">pq</span><span class="p">.</span><span class="n">push</span><span class="p">({</span><span class="n">dist</span><span class="p">[</span><span class="n">v</span><span class="p">],</span> <span class="n">v</span><span class="p">});</span>
                <span class="p">}</span>
            <span class="p">}</span>
        <span class="p">}</span>
</code></pre>

</div>



<p>From each newly reached village, the messenger looked at <strong>every road leading outward</strong>.<br>
If traveling through this village gave a <strong>shorter path</strong> to another village than previously known, the ledger was updated.<br>
The beacon-fire was fed with this news, so those closer villages could be visited next.</p>

<p>And so, like ripples in ice, the shortest paths spread from the capital outward.</p>






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



<p>When the last flame of the beacon-fire faded, the ledger was unrolled before the High Council:</p>

<ul>
<li>If a number was written, it was the <strong>fastest possible travel time</strong> from the capital to that village.</li>
<li>If <code>INF</code>, it meant <strong>the snow had blocked every road to that place</strong> — unreachable until the spring thaw.</li>
</ul>






<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">Dijkstra</span> <span class="n">realm</span><span class="p">(</span><span class="mi">5</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">9</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">4</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">4</span><span class="p">);</span>
    <span class="n">realm</span><span class="p">.</span><span class="n">addRoad</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">6</span><span class="p">);</span>

    <span class="n">realm</span><span class="p">.</span><span class="n">shortestPaths</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The messengers rode, flames guiding them village by village, until the whole kingdom had been warned — in the <strong>fastest way possible</strong>.</p>

