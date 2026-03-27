---
Title: PostgreSQL JSONB for Video Analytics Dashboards
Description: 
Author: ahmet gedik
Date: 2026-03-27T22:00:01.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Problem with Rigid Schemas for Analytics
</h2>

<p>Video analytics data is inherently messy. One day you are tracking view counts by region, the next you need device breakdowns, then watch-time percentiles. Adding a column for every new metric leads to a table with 50 nullable columns and constant migrations. At <a href="https://dailywatch.video" rel="noopener noreferrer">DailyWatch</a>, we solved this with PostgreSQL's JSONB column type.</p>

<h2>
  
  
  Schema Design
</h2>

<p>We keep structured data in normal columns and flexible analytics in JSONB:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">video_analytics</span> <span class="p">(</span>
    <span class="n">id</span> <span class="nb">SERIAL</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span>
    <span class="n">video_id</span> <span class="nb">TEXT</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">REFERENCES</span> <span class="n">videos</span><span class="p">(</span><span class="n">video_id</span><span class="p">),</span>
    <span class="n">captured_at</span> <span class="n">TIMESTAMPTZ</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="n">NOW</span><span class="p">(),</span>
    <span class="n">views</span> <span class="nb">INTEGER</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">likes</span> <span class="nb">INTEGER</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">metrics</span> <span class="n">JSONB</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="s1">'{}'</span>
<span class="p">);</span>
</code></pre>

</div>



<p>The <code>metrics</code> column stores everything that might change shape over time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"regions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"US"</span><span class="p">:</span><span class="w"> </span><span class="mi">45000</span><span class="p">,</span><span class="w"> </span><span class="nl">"GB"</span><span class="p">:</span><span class="w"> </span><span class="mi">12000</span><span class="p">,</span><span class="w"> </span><span class="nl">"DE"</span><span class="p">:</span><span class="w"> </span><span class="mi">8500</span><span class="p">,</span><span class="w"> </span><span class="nl">"FR"</span><span class="p">:</span><span class="w"> </span><span class="mi">6200</span><span class="p">},</span><span class="w">
  </span><span class="nl">"devices"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"mobile"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.62</span><span class="p">,</span><span class="w"> </span><span class="nl">"desktop"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.31</span><span class="p">,</span><span class="w"> </span><span class="nl">"tablet"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.07</span><span class="p">},</span><span class="w">
  </span><span class="nl">"watch_time"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"avg_seconds"</span><span class="p">:</span><span class="w"> </span><span class="mi">142</span><span class="p">,</span><span class="w"> </span><span class="nl">"p50"</span><span class="p">:</span><span class="w"> </span><span class="mi">98</span><span class="p">,</span><span class="w"> </span><span class="nl">"p95"</span><span class="p">:</span><span class="w"> </span><span class="mi">340</span><span class="p">},</span><span class="w">
  </span><span class="nl">"engagement"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"like_rate"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.034</span><span class="p">,</span><span class="w"> </span><span class="nl">"comment_rate"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.008</span><span class="p">},</span><span class="w">
  </span><span class="nl">"traffic_sources"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"search"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.35</span><span class="p">,</span><span class="w"> </span><span class="nl">"suggested"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.42</span><span class="p">,</span><span class="w"> </span><span class="nl">"external"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.23</span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Indexing JSONB for Fast Queries
</h2>

<p>A GIN index on the JSONB column lets PostgreSQL search inside the JSON efficiently:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- General-purpose GIN index</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_analytics_metrics</span> <span class="k">ON</span> <span class="n">video_analytics</span> <span class="k">USING</span> <span class="n">GIN</span><span class="p">(</span><span class="n">metrics</span><span class="p">);</span>

<span class="c1">-- Targeted index for a specific path (faster, smaller)</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_analytics_regions</span> <span class="k">ON</span> <span class="n">video_analytics</span>
    <span class="k">USING</span> <span class="n">GIN</span><span class="p">((</span><span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'regions'</span><span class="p">));</span>
</code></pre>

</div>



<p>The general GIN index supports containment (<code>@&gt;</code>) and existence (<code>?</code>) operators. The path-specific index is smaller and faster if you know which keys you query most.</p>

<h2>
  
  
  Querying JSONB Data
</h2>

<p>Here is how we extract analytics for the admin dashboard at <a href="https://dailywatch.video" rel="noopener noreferrer">DailyWatch</a>:</p>

<p><strong>Get regional view breakdown:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">video_id</span><span class="p">,</span>
       <span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'regions'</span> <span class="o">-&gt;&gt;</span> <span class="s1">'US'</span> <span class="k">AS</span> <span class="n">us_views</span><span class="p">,</span>
       <span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'regions'</span> <span class="o">-&gt;&gt;</span> <span class="s1">'DE'</span> <span class="k">AS</span> <span class="n">de_views</span><span class="p">,</span>
       <span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'watch_time'</span> <span class="o">-&gt;&gt;</span> <span class="s1">'avg_seconds'</span> <span class="k">AS</span> <span class="n">avg_watch</span>
<span class="k">FROM</span> <span class="n">video_analytics</span>
<span class="k">WHERE</span> <span class="n">captured_at</span> <span class="o">&gt;</span> <span class="n">NOW</span><span class="p">()</span> <span class="o">-</span> <span class="n">INTERVAL</span> <span class="s1">'24 hours'</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">views</span> <span class="k">DESC</span>
<span class="k">LIMIT</span> <span class="mi">20</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Find videos where mobile traffic exceeds 70%:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">v</span><span class="p">.</span><span class="n">title</span><span class="p">,</span> <span class="n">a</span><span class="p">.</span><span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'devices'</span> <span class="o">-&gt;&gt;</span> <span class="s1">'mobile'</span> <span class="k">AS</span> <span class="n">mobile_share</span>
<span class="k">FROM</span> <span class="n">video_analytics</span> <span class="n">a</span>
<span class="k">JOIN</span> <span class="n">videos</span> <span class="n">v</span> <span class="k">ON</span> <span class="n">v</span><span class="p">.</span><span class="n">video_id</span> <span class="o">=</span> <span class="n">a</span><span class="p">.</span><span class="n">video_id</span>
<span class="k">WHERE</span> <span class="p">(</span><span class="n">a</span><span class="p">.</span><span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'devices'</span> <span class="o">-&gt;&gt;</span> <span class="s1">'mobile'</span><span class="p">)::</span><span class="nb">float</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">.</span><span class="mi">70</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">a</span><span class="p">.</span><span class="n">captured_at</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Aggregate regional views across all videos:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span>
    <span class="k">key</span> <span class="k">AS</span> <span class="n">region</span><span class="p">,</span>
    <span class="k">SUM</span><span class="p">(</span><span class="n">value</span><span class="p">::</span><span class="nb">integer</span><span class="p">)</span> <span class="k">AS</span> <span class="n">total_views</span>
<span class="k">FROM</span> <span class="n">video_analytics</span><span class="p">,</span>
     <span class="n">jsonb_each_text</span><span class="p">(</span><span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'regions'</span><span class="p">)</span>
<span class="k">WHERE</span> <span class="n">captured_at</span> <span class="o">&gt;</span> <span class="n">NOW</span><span class="p">()</span> <span class="o">-</span> <span class="n">INTERVAL</span> <span class="s1">'7 days'</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="k">key</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">total_views</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<p>The <code>jsonb_each_text</code> function unpacks a JSONB object into rows — incredibly useful for aggregation across dynamic keys.</p>

<h2>
  
  
  Appending to JSONB Without Full Replacement
</h2>

<p>PostgreSQL's <code>jsonb_set</code> function updates specific paths without replacing the entire object:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Add a new traffic source</span>
<span class="k">UPDATE</span> <span class="n">video_analytics</span>
<span class="k">SET</span> <span class="n">metrics</span> <span class="o">=</span> <span class="n">jsonb_set</span><span class="p">(</span><span class="n">metrics</span><span class="p">,</span> <span class="s1">'{traffic_sources,direct}'</span><span class="p">,</span> <span class="s1">'0.15'</span><span class="p">)</span>
<span class="k">WHERE</span> <span class="n">video_id</span> <span class="o">=</span> <span class="s1">'abc123'</span><span class="p">;</span>

<span class="c1">-- Merge new data into existing object</span>
<span class="k">UPDATE</span> <span class="n">video_analytics</span>
<span class="k">SET</span> <span class="n">metrics</span> <span class="o">=</span> <span class="n">metrics</span> <span class="o">||</span> <span class="s1">'{"cdn_cost": 0.0023}'</span><span class="p">::</span><span class="n">jsonb</span>
<span class="k">WHERE</span> <span class="n">video_id</span> <span class="o">=</span> <span class="s1">'abc123'</span><span class="p">;</span>
</code></pre>

</div>



<p>The <code>||</code> operator merges objects. Existing keys are overwritten, new keys are added. This is how we incrementally enrich analytics records as new data arrives.</p>

<h2>
  
  
  Building a Time-Series View
</h2>

<p>For dashboard charts, we need metrics over time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span>
    <span class="n">date_trunc</span><span class="p">(</span><span class="s1">'hour'</span><span class="p">,</span> <span class="n">captured_at</span><span class="p">)</span> <span class="k">AS</span> <span class="n">hour</span><span class="p">,</span>
    <span class="k">AVG</span><span class="p">(</span><span class="n">views</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_views</span><span class="p">,</span>
    <span class="k">AVG</span><span class="p">((</span><span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'watch_time'</span> <span class="o">-&gt;&gt;</span> <span class="s1">'avg_seconds'</span><span class="p">)::</span><span class="nb">float</span><span class="p">)</span> <span class="k">AS</span> <span class="n">avg_watch_time</span><span class="p">,</span>
    <span class="k">AVG</span><span class="p">((</span><span class="n">metrics</span> <span class="o">-&gt;</span> <span class="s1">'devices'</span> <span class="o">-&gt;&gt;</span> <span class="s1">'mobile'</span><span class="p">)::</span><span class="nb">float</span><span class="p">)</span> <span class="k">AS</span> <span class="n">mobile_share</span>
<span class="k">FROM</span> <span class="n">video_analytics</span>
<span class="k">WHERE</span> <span class="n">video_id</span> <span class="o">=</span> <span class="s1">'abc123'</span>
  <span class="k">AND</span> <span class="n">captured_at</span> <span class="o">&gt;</span> <span class="n">NOW</span><span class="p">()</span> <span class="o">-</span> <span class="n">INTERVAL</span> <span class="s1">'7 days'</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">hour</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">hour</span><span class="p">;</span>
</code></pre>

</div>



<p>This gives us hourly data points for charting view trends, watch time, and device mix — all from a single JSONB column.</p>

<h2>
  
  
  When NOT to Use JSONB
</h2>

<p>JSONB is not a replacement for proper schema design. We use normal columns for data we always query (video_id, views, likes, captured_at) and JSONB for data that is optional, variable-shape, or infrequently filtered.</p>

<p>Bad use of JSONB: storing video_id or title inside JSON when you need to JOIN on them.<br>
Good use of JSONB: storing analytics breakdowns that vary by source and evolve over time.</p>

<h2>
  
  
  Performance Notes
</h2>

<p>On our dataset of ~50,000 analytics records with GIN indexes:</p>

<ul>
<li>Path extraction (<code>-&gt;&gt;</code>) queries: 1-3ms</li>
<li>Containment (<code>@&gt;</code>) queries: 2-5ms</li>
<li>
<code>jsonb_each_text</code> aggregation: 10-20ms</li>
<li>Full-table JSONB aggregation: 40-80ms</li>
</ul>

<p>The flexibility of JSONB let us iterate on our analytics dashboard without a single migration after the initial table creation.</p>




<p><em>This article is part of the Building DailyWatch series. Check out <a href="https://dailywatch.video" rel="noopener noreferrer">DailyWatch</a> to see these techniques in action.</em></p>

