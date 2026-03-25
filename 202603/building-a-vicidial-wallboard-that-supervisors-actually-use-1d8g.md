---
Title: Building a VICIdial Wallboard That Supervisors Actually Use
Description: 
Author: Jason Shouldice
Date: 2026-03-25T22:05:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>VICIdial's default real-time report is the screen every call center supervisor stares at all day. It tells you who's talking, who's paused, who's waiting. At 10 agents, it works. At 25+, it falls apart — no at-a-glance KPI summary, no visual grouping by status, a page refresh that flickers and resets scroll position, and zero historical context.</p>

<p>Here's how to build something better without replacing VICIdial.</p>

<h2>
  
  
  What the Default Report Gets Wrong
</h2>

<p>Four specific problems that matter at scale:</p>

<p><strong>No aggregate summary.</strong> You can't glance at the screen and know your average talk time, calls per hour, or how many agents are productive right now. You have to mentally tally individual rows while the data refreshes underneath you.</p>

<p><strong>Flat layout.</strong> Every agent occupies one row regardless of status. INCALL agents (making money) are mixed with PAUSED agents (costing money). There's no color coding, no priority grouping, no visual separation between "healthy" and "needs attention."</p>

<p><strong>Full-page refresh.</strong> The meta-refresh tag reloads the entire page at a fixed interval. Flash. Scroll position reset. On a wallboard TV, it looks unprofessional and distracts everyone.</p>

<p><strong>No trending.</strong> It's purely instantaneous. You can't see whether an agent's calls per hour are declining through their shift or whether wait time has been climbing for 30 minutes.</p>

<h3>
  
  
  URL Parameters Most Admins Never Discover
</h3>

<p>Before building a custom dashboard, try the built-in parameters:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="o">/</span><span class="n">vicidial</span><span class="o">/</span><span class="n">realtime_report</span><span class="mf">.</span><span class="n">php</span><span class="o">?</span><span class="no">DB</span><span class="o">=</span><span class="mi">0</span><span class="o">&amp;</span><span class="n">group</span><span class="o">=</span><span class="no">SALESCAMP</span><span class="o">&amp;</span><span class="no">RR</span><span class="o">=</span><span class="mi">4</span><span class="o">&amp;</span><span class="nc">Session</span><span class="o">=</span><span class="n">your_session</span>
</code></pre>

</div>



<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Parameter</th>
<th>Purpose</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>group</code></td>
<td>Filter to a specific campaign</td>
<td><code>group=SALESCAMP</code></td>
</tr>
<tr>
<td><code>RR</code></td>
<td>Refresh interval in seconds</td>
<td><code>RR=4</code></td>
</tr>
<tr>
<td><code>show_parks</code></td>
<td>Show parked calls</td>
<td><code>show_parks=1</code></td>
</tr>
<tr>
<td><code>NOLOGin</code></td>
<td>Use without agent login</td>
<td><code>NOLOGin=Y</code></td>
</tr>
</tbody>
</table></div>

<p>These help, but they don't fix the fundamental layout and aggregation problems.</p>

<h2>
  
  
  The 5 Metrics Worth Wallboard Space
</h2>

<p>Not every metric deserves a spot on the TV. Focus on numbers that drive behavior:</p>

<p><strong>1. Calls Per Hour by agent and campaign average.</strong> The single most actionable outbound metric. Agents self-correct when they see how they compare to the group average.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="k">user</span><span class="p">,</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">AS</span> <span class="n">calls</span><span class="p">,</span>
    <span class="n">ROUND</span><span class="p">(</span><span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="o">/</span> <span class="p">(</span><span class="n">TIMESTAMPDIFF</span><span class="p">(</span><span class="k">MINUTE</span><span class="p">,</span> <span class="k">MIN</span><span class="p">(</span><span class="n">event_time</span><span class="p">),</span> <span class="n">NOW</span><span class="p">())</span> <span class="o">/</span> <span class="mi">60</span><span class="p">),</span> <span class="mi">1</span><span class="p">)</span> <span class="k">AS</span> <span class="n">cph</span>
<span class="k">FROM</span> <span class="n">vicidial_agent_log</span>
<span class="k">WHERE</span> <span class="n">event_time</span> <span class="o">&gt;=</span> <span class="n">CURDATE</span><span class="p">()</span>
  <span class="k">AND</span> <span class="n">campaign_id</span> <span class="o">=</span> <span class="s1">'SALESCAMP'</span> <span class="k">AND</span> <span class="n">talk_sec</span> <span class="o">&gt;</span> <span class="mi">0</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="k">user</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">cph</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>2. Talk/Pause ratio.</strong> An agent with 3 hours talk and 45 minutes pause is performing very differently from one with 1 hour talk and 2.5 hours pause. Show the ratio, not just the raw numbers.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">vla</span><span class="p">.</span><span class="k">user</span><span class="p">,</span> <span class="n">vu</span><span class="p">.</span><span class="n">full_name</span><span class="p">,</span>
    <span class="n">val_sum</span><span class="p">.</span><span class="n">talk_seconds</span><span class="p">,</span> <span class="n">val_sum</span><span class="p">.</span><span class="n">pause_seconds</span><span class="p">,</span>
    <span class="n">ROUND</span><span class="p">(</span><span class="n">val_sum</span><span class="p">.</span><span class="n">talk_seconds</span> <span class="o">/</span> <span class="k">NULLIF</span><span class="p">(</span><span class="n">val_sum</span><span class="p">.</span><span class="n">pause_seconds</span><span class="p">,</span> <span class="mi">0</span><span class="p">),</span> <span class="mi">2</span><span class="p">)</span> <span class="k">AS</span> <span class="n">talk_pause_ratio</span>
<span class="k">FROM</span> <span class="n">vicidial_live_agents</span> <span class="n">vla</span>
<span class="k">JOIN</span> <span class="n">vicidial_users</span> <span class="n">vu</span> <span class="k">ON</span> <span class="n">vla</span><span class="p">.</span><span class="k">user</span> <span class="o">=</span> <span class="n">vu</span><span class="p">.</span><span class="k">user</span>
<span class="k">LEFT</span> <span class="k">JOIN</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="k">user</span><span class="p">,</span> <span class="k">SUM</span><span class="p">(</span><span class="n">talk_sec</span><span class="p">)</span> <span class="k">AS</span> <span class="n">talk_seconds</span><span class="p">,</span> <span class="k">SUM</span><span class="p">(</span><span class="n">pause_sec</span><span class="p">)</span> <span class="k">AS</span> <span class="n">pause_seconds</span>
    <span class="k">FROM</span> <span class="n">vicidial_agent_log</span> <span class="k">WHERE</span> <span class="n">event_time</span> <span class="o">&gt;=</span> <span class="n">CURDATE</span><span class="p">()</span>
    <span class="k">GROUP</span> <span class="k">BY</span> <span class="k">user</span>
<span class="p">)</span> <span class="n">val_sum</span> <span class="k">ON</span> <span class="n">vla</span><span class="p">.</span><span class="k">user</span> <span class="o">=</span> <span class="n">val_sum</span><span class="p">.</span><span class="k">user</span>
<span class="k">WHERE</span> <span class="n">vla</span><span class="p">.</span><span class="n">campaign_id</span> <span class="o">=</span> <span class="s1">'SALESCAMP'</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">talk_pause_ratio</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>3. Active pause codes.</strong> Don't just show that agents are paused — show why. The <code>pause_code</code> field in <code>vicidial_live_agents</code> tells you whether someone is on a legitimate lunch or has been in BREAK for 45 minutes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">va</span><span class="p">.</span><span class="k">user</span><span class="p">,</span> <span class="n">vu</span><span class="p">.</span><span class="n">full_name</span><span class="p">,</span> <span class="n">va</span><span class="p">.</span><span class="n">pause_code</span><span class="p">,</span>
    <span class="n">UNIX_TIMESTAMP</span><span class="p">(</span><span class="n">NOW</span><span class="p">())</span> <span class="o">-</span> <span class="n">UNIX_TIMESTAMP</span><span class="p">(</span><span class="n">va</span><span class="p">.</span><span class="n">last_state_change</span><span class="p">)</span> <span class="k">AS</span> <span class="n">seconds_paused</span>
<span class="k">FROM</span> <span class="n">vicidial_live_agents</span> <span class="n">va</span>
<span class="k">JOIN</span> <span class="n">vicidial_users</span> <span class="n">vu</span> <span class="k">ON</span> <span class="n">va</span><span class="p">.</span><span class="k">user</span> <span class="o">=</span> <span class="n">vu</span><span class="p">.</span><span class="k">user</span>
<span class="k">WHERE</span> <span class="n">va</span><span class="p">.</span><span class="n">status</span> <span class="o">=</span> <span class="s1">'PAUSED'</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">seconds_paused</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>4. Wait time.</strong> High wait time means agents are idle waiting for calls — a list exhaustion problem, a dial level problem, or a trunk capacity problem. Show it prominently so supervisors investigate before it gets worse.</p>

<p><strong>5. Drop count.</strong> For outbound, show drops in the last 30 minutes. This is a compliance number that should always be visible.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">campaign_id</span><span class="p">,</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">AS</span> <span class="n">drops_30min</span>
<span class="k">FROM</span> <span class="n">vicidial_log</span>
<span class="k">WHERE</span> <span class="n">call_date</span> <span class="o">&gt;=</span> <span class="n">DATE_SUB</span><span class="p">(</span><span class="n">NOW</span><span class="p">(),</span> <span class="n">INTERVAL</span> <span class="mi">30</span> <span class="k">MINUTE</span><span class="p">)</span>
  <span class="k">AND</span> <span class="n">status</span> <span class="o">=</span> <span class="s1">'DROP'</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">campaign_id</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  Replace the Flicker With AJAX
</h2>

<p>The biggest usability win is replacing the full-page refresh with AJAX polling. Create a PHP endpoint that returns agent data as JSON, then fetch it every 4 seconds and update the DOM without a page reload.</p>

<h3>
  
  
  Backend: JSON Data Endpoint
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>
<span class="c1">// wallboard_data.php — place in /var/www/html/vicidial/</span>
<span class="nb">header</span><span class="p">(</span><span class="s1">'Content-Type: application/json'</span><span class="p">);</span>
<span class="k">require_once</span><span class="p">(</span><span class="s2">"dbconnect_mysqli.php"</span><span class="p">);</span>

<span class="nv">$campaign</span> <span class="o">=</span> <span class="nf">mysqli_real_escape_string</span><span class="p">(</span><span class="nv">$link</span><span class="p">,</span> <span class="nv">$_GET</span><span class="p">[</span><span class="s1">'campaign'</span><span class="p">]</span> <span class="o">??</span> <span class="s1">'SALESCAMP'</span><span class="p">);</span>

<span class="nv">$result</span> <span class="o">=</span> <span class="nf">mysqli_query</span><span class="p">(</span><span class="nv">$link</span><span class="p">,</span> <span class="s2">"SELECT va.user, vu.full_name, va.status,
    va.pause_code AS sub_status, va.calls_today,
    UNIX_TIMESTAMP(NOW()) - UNIX_TIMESTAMP(va.last_state_change) AS seconds_in_state
FROM vicidial_live_agents va
JOIN vicidial_users vu ON va.user = vu.user
WHERE va.campaign_id = '</span><span class="nv">$campaign</span><span class="s2">'
ORDER BY va.status, vu.full_name"</span><span class="p">);</span>

<span class="nv">$agents</span> <span class="o">=</span> <span class="p">[];</span>
<span class="k">while</span> <span class="p">(</span><span class="nv">$row</span> <span class="o">=</span> <span class="nf">mysqli_fetch_assoc</span><span class="p">(</span><span class="nv">$result</span><span class="p">))</span> <span class="nv">$agents</span><span class="p">[]</span> <span class="o">=</span> <span class="nv">$row</span><span class="p">;</span>

<span class="nv">$summary_result</span> <span class="o">=</span> <span class="nf">mysqli_query</span><span class="p">(</span><span class="nv">$link</span><span class="p">,</span> <span class="s2">"SELECT
    COUNT(*) AS total_agents,
    SUM(CASE WHEN status='INCALL' THEN 1 ELSE 0 END) AS agents_incall,
    SUM(CASE WHEN status='READY' THEN 1 ELSE 0 END) AS agents_ready,
    SUM(CASE WHEN status='PAUSED' THEN 1 ELSE 0 END) AS agents_paused,
    ROUND(AVG(calls_today), 1) AS avg_calls
FROM vicidial_live_agents WHERE campaign_id = '</span><span class="nv">$campaign</span><span class="s2">'"</span><span class="p">);</span>
<span class="nv">$summary</span> <span class="o">=</span> <span class="nf">mysqli_fetch_assoc</span><span class="p">(</span><span class="nv">$summary_result</span><span class="p">);</span>

<span class="k">echo</span> <span class="nb">json_encode</span><span class="p">([</span>
    <span class="s1">'timestamp'</span> <span class="o">=&gt;</span> <span class="nb">time</span><span class="p">(),</span>
    <span class="s1">'campaign'</span> <span class="o">=&gt;</span> <span class="nv">$campaign</span><span class="p">,</span>
    <span class="s1">'summary'</span> <span class="o">=&gt;</span> <span class="nv">$summary</span><span class="p">,</span>
    <span class="s1">'agents'</span> <span class="o">=&gt;</span> <span class="nv">$agents</span>
<span class="p">]);</span>
<span class="cp">?&gt;</span>
</code></pre>

</div>



<h3>
  
  
  Frontend: AJAX Polling With Color-Coded Cards
</h3>

<p>The key is rendering agents as cards grouped by status, with smooth transitions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">CAMPAIGN</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">SALESCAMP</span><span class="dl">'</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">REFRESH_MS</span> <span class="o">=</span> <span class="mi">4000</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">fetchData</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">resp</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="s2">`wallboard_data.php?campaign=</span><span class="p">${</span><span class="nx">CAMPAIGN</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
        <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">resp</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
        <span class="nf">renderSummary</span><span class="p">(</span><span class="nx">data</span><span class="p">.</span><span class="nx">summary</span><span class="p">);</span>
        <span class="nf">renderAgents</span><span class="p">(</span><span class="nx">data</span><span class="p">.</span><span class="nx">agents</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Fetch failed:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">e</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">renderAgents</span><span class="p">(</span><span class="nx">agents</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">html</span> <span class="o">=</span> <span class="nx">agents</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">a</span> <span class="o">=&gt;</span> <span class="s2">`
        &lt;div class="agent-card </span><span class="p">${</span><span class="nx">a</span><span class="p">.</span><span class="nx">status</span><span class="p">}</span><span class="s2">"&gt;
            &lt;strong&gt;</span><span class="p">${</span><span class="nx">a</span><span class="p">.</span><span class="nx">full_name</span><span class="p">}</span><span class="s2">&lt;/strong&gt;&lt;br&gt;
            </span><span class="p">${</span><span class="nx">a</span><span class="p">.</span><span class="nx">status</span><span class="p">}${</span><span class="nx">a</span><span class="p">.</span><span class="nx">sub_status</span> <span class="p">?</span> <span class="dl">'</span><span class="s1"> (</span><span class="dl">'</span> <span class="o">+</span> <span class="nx">a</span><span class="p">.</span><span class="nx">sub_status</span> <span class="o">+</span> <span class="dl">'</span><span class="s1">)</span><span class="dl">'</span> <span class="p">:</span> <span class="dl">''</span><span class="p">}</span><span class="s2">&lt;br&gt;
            In state: </span><span class="p">${</span><span class="nf">formatTime</span><span class="p">(</span><span class="nf">parseInt</span><span class="p">(</span><span class="nx">a</span><span class="p">.</span><span class="nx">seconds_in_state</span><span class="p">))}</span><span class="s2">&lt;br&gt;
            Calls: </span><span class="p">${</span><span class="nx">a</span><span class="p">.</span><span class="nx">calls_today</span><span class="p">}</span><span class="s2">
        &lt;/div&gt;
    `</span><span class="p">).</span><span class="nf">join</span><span class="p">(</span><span class="dl">''</span><span class="p">);</span>
    <span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">'</span><span class="s1">agents</span><span class="dl">'</span><span class="p">).</span><span class="nx">innerHTML</span> <span class="o">=</span> <span class="nx">html</span><span class="p">;</span>
<span class="p">}</span>

<span class="nf">fetchData</span><span class="p">();</span>
<span class="nf">setInterval</span><span class="p">(</span><span class="nx">fetchData</span><span class="p">,</span> <span class="nx">REFRESH_MS</span><span class="p">);</span>
</code></pre>

</div>



<p>Use CSS: green background for INCALL, blue for READY, red for PAUSED. Set the card border color to match. Large text visible from across the room. No flicker, no scroll reset, smooth transitions when agents change state.</p>

<p>Set <code>REFRESH_MS</code> to 3000-5000ms. Faster than 3 seconds adds database load for minimal benefit. Slower than 5 seconds makes the display feel stale.</p>

<h2>
  
  
  Advanced Queries for Live Dashboards
</h2>

<h3>
  
  
  Campaign Performance Summary
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">vla</span><span class="p">.</span><span class="n">campaign_id</span><span class="p">,</span>
    <span class="k">COUNT</span><span class="p">(</span><span class="k">DISTINCT</span> <span class="n">vla</span><span class="p">.</span><span class="k">user</span><span class="p">)</span> <span class="k">AS</span> <span class="n">logged_in_agents</span><span class="p">,</span>
    <span class="k">SUM</span><span class="p">(</span><span class="k">CASE</span> <span class="k">WHEN</span> <span class="n">vla</span><span class="p">.</span><span class="n">status</span> <span class="o">=</span> <span class="s1">'INCALL'</span> <span class="k">THEN</span> <span class="mi">1</span> <span class="k">ELSE</span> <span class="mi">0</span> <span class="k">END</span><span class="p">)</span> <span class="k">AS</span> <span class="n">on_call</span><span class="p">,</span>
    <span class="p">(</span><span class="k">SELECT</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">vicidial_log</span> <span class="n">vl</span>
     <span class="k">WHERE</span> <span class="n">vl</span><span class="p">.</span><span class="n">campaign_id</span> <span class="o">=</span> <span class="n">vla</span><span class="p">.</span><span class="n">campaign_id</span> <span class="k">AND</span> <span class="n">vl</span><span class="p">.</span><span class="n">call_date</span> <span class="o">&gt;=</span> <span class="n">CURDATE</span><span class="p">())</span> <span class="k">AS</span> <span class="n">calls_today</span><span class="p">,</span>
    <span class="p">(</span><span class="k">SELECT</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">FROM</span> <span class="n">vicidial_log</span> <span class="n">vl</span>
     <span class="k">WHERE</span> <span class="n">vl</span><span class="p">.</span><span class="n">campaign_id</span> <span class="o">=</span> <span class="n">vla</span><span class="p">.</span><span class="n">campaign_id</span>
       <span class="k">AND</span> <span class="n">vl</span><span class="p">.</span><span class="n">call_date</span> <span class="o">&gt;=</span> <span class="n">DATE_SUB</span><span class="p">(</span><span class="n">NOW</span><span class="p">(),</span> <span class="n">INTERVAL</span> <span class="mi">1</span> <span class="n">HOUR</span><span class="p">))</span> <span class="k">AS</span> <span class="n">calls_last_hour</span>
<span class="k">FROM</span> <span class="n">vicidial_live_agents</span> <span class="n">vla</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">vla</span><span class="p">.</span><span class="n">campaign_id</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  Hopper Health
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">campaign_id</span><span class="p">,</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">AS</span> <span class="n">leads_in_hopper</span>
<span class="k">FROM</span> <span class="n">vicidial_hopper</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">campaign_id</span><span class="p">;</span>
</code></pre>

</div>



<p>A hopper count below 2x your agent count is a warning. Below 1x and the dialer is starving. Color-code the threshold on your wallboard.</p>

<h2>
  
  
  Grafana for Historical Trending
</h2>

<p>For operations that want to see trends over time, Grafana with a MySQL datasource is the way to go. Install on a separate box. Create a read-only MySQL user:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">USER</span> <span class="s1">'grafana_ro'</span><span class="o">@</span><span class="s1">'10.0.1.%'</span> <span class="n">IDENTIFIED</span> <span class="k">BY</span> <span class="s1">'strong_password'</span><span class="p">;</span>
<span class="k">GRANT</span> <span class="k">SELECT</span> <span class="k">ON</span> <span class="n">asterisk</span><span class="p">.</span><span class="n">vicidial_live_agents</span> <span class="k">TO</span> <span class="s1">'grafana_ro'</span><span class="o">@</span><span class="s1">'10.0.1.%'</span><span class="p">;</span>
<span class="k">GRANT</span> <span class="k">SELECT</span> <span class="k">ON</span> <span class="n">asterisk</span><span class="p">.</span><span class="n">vicidial_agent_log</span> <span class="k">TO</span> <span class="s1">'grafana_ro'</span><span class="o">@</span><span class="s1">'10.0.1.%'</span><span class="p">;</span>
<span class="k">GRANT</span> <span class="k">SELECT</span> <span class="k">ON</span> <span class="n">asterisk</span><span class="p">.</span><span class="n">vicidial_log</span> <span class="k">TO</span> <span class="s1">'grafana_ro'</span><span class="o">@</span><span class="s1">'10.0.1.%'</span><span class="p">;</span>
<span class="k">GRANT</span> <span class="k">SELECT</span> <span class="k">ON</span> <span class="n">asterisk</span><span class="p">.</span><span class="n">vicidial_hopper</span> <span class="k">TO</span> <span class="s1">'grafana_ro'</span><span class="o">@</span><span class="s1">'10.0.1.%'</span><span class="p">;</span>
<span class="n">FLUSH</span> <span class="k">PRIVILEGES</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  Most Valuable Panels
</h3>

<p><strong>Calls Per Hour Trend:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="err">$</span><span class="n">__timeGroup</span><span class="p">(</span><span class="n">call_date</span><span class="p">,</span> <span class="s1">'1h'</span><span class="p">)</span> <span class="k">AS</span> <span class="nb">time</span><span class="p">,</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">AS</span> <span class="n">calls</span>
<span class="k">FROM</span> <span class="n">vicidial_log</span>
<span class="k">WHERE</span> <span class="err">$</span><span class="n">__timeFilter</span><span class="p">(</span><span class="n">call_date</span><span class="p">)</span> <span class="k">AND</span> <span class="n">campaign_id</span> <span class="o">=</span> <span class="s1">'SALESCAMP'</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="nb">time</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="nb">time</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Drop Rate Gauge:</strong> Set thresholds at green below 2%, yellow 2-3%, red above 3% (the FTC safe harbor limit).</p>

<p><strong>Grafana Alerts:</strong> Configure notifications for drop rate exceeding 2.5%, hopper below threshold, wait time over 30 seconds, more than 50% of agents paused simultaneously.</p>

<h2>
  
  
  Physical Wallboard Layout
</h2>

<p>For a 25+ agent room with multiple screens:</p>

<ul>
<li>
<strong>Screen 1</strong> — Agent status grid (the AJAX card layout). Color coded, large text, visible from across the room.</li>
<li>
<strong>Screen 2</strong> — Leaderboard: top 10 agents by conversions and CPH.</li>
<li>
<strong>Screen 3</strong> — Grafana: CPH trend, drop rate gauge, hopper health.</li>
</ul>

<p>Use a Raspberry Pi per display running Chromium in kiosk mode:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>chromium-browser <span class="nt">--kiosk</span> <span class="nt">--disable-restore-session-state</span> <span class="se">\</span>
    <span class="s2">"http://10.0.1.10/vicidial/custom_wallboard.php?campaign=SALESCAMP"</span>
</code></pre>

</div>



<p>The complete guide with full HTML/CSS/JS for the AJAX wallboard, all Grafana queries, and alerting YAML is at <a href="https://vicistack.com/blog/vicidial-realtime-agent-dashboard/" rel="noopener noreferrer">ViciStack's real-time dashboard guide</a>.</p>




<p>Originally published at <a href="https://vicistack.com/blog/vicidial-realtime-agent-dashboard/" rel="noopener noreferrer">https://vicistack.com/blog/vicidial-realtime-agent-dashboard/</a></p>

