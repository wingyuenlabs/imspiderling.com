---
Title: Why Your IoT Device State Is Probably Wrong
Description: 
Author: Tyler
Date: 2026-03-05T21:29:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>You built the connected device platform. The dashboard shows device status in real time. Your customers are happy — until a door that the system says is locked is standing open, or a sensor that shows "online" has been offline for six minutes.</p>

<p>The data was never missing. The events all arrived. The problem is subtler than that: <strong>your system resolved the wrong state from correct inputs.</strong></p>

<p>This happens to almost every IoT platform at scale, and it almost always comes down to one of three unhandled edge cases.</p>




<h2>
  
  
  The Late-Arriving Disconnect
</h2>

<p>Here is the scenario. A device drops its connection briefly — 800 milliseconds — then reconnects. Your broker receives two events:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>T+0ms    → RECONNECT  (arrives first, processed first)
T+340ms  → DISCONNECT (late-arriving, processed second)
</code></pre>

</div>



<p>Network variance inverted the delivery order. Your system processes them in arrival order, so the final state is <code>offline</code>. The device is online. Your customer gets an alert.</p>

<p>This is not a bug in your broker. It is a design gap in your state resolution layer. You have delivery infrastructure but no arbitration logic.</p>

<p>The naive fix is Last Write Wins on timestamp:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">resolve_state</span><span class="p">(</span><span class="n">events</span><span class="p">):</span>
    <span class="k">return</span> <span class="nf">sorted</span><span class="p">(</span><span class="n">events</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">e</span><span class="p">:</span> <span class="n">e</span><span class="p">[</span><span class="sh">'</span><span class="s">timestamp</span><span class="sh">'</span><span class="p">])[</span><span class="o">-</span><span class="mi">1</span><span class="p">][</span><span class="sh">'</span><span class="s">status</span><span class="sh">'</span><span class="p">]</span>
</code></pre>

</div>



<p>This works until your devices have clock drift. A device that woke from deep sleep with a stale RTC will send events with timestamps from six minutes ago. LWW on timestamp now resolves stale state as authoritative.</p>

<p>The correct fix is to treat delivery order and timestamp as two separate signals, weight them based on confidence, and make the arbitration decision explicit.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">resolve_state</span><span class="p">(</span><span class="n">events</span><span class="p">,</span> <span class="n">reconnect_window_seconds</span><span class="o">=</span><span class="mi">30</span><span class="p">):</span>
    <span class="n">sorted_by_arrival</span> <span class="o">=</span> <span class="nf">sorted</span><span class="p">(</span><span class="n">events</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">e</span><span class="p">:</span> <span class="n">e</span><span class="p">[</span><span class="sh">'</span><span class="s">arrival_time</span><span class="sh">'</span><span class="p">])</span>
    <span class="n">sorted_by_timestamp</span> <span class="o">=</span> <span class="nf">sorted</span><span class="p">(</span><span class="n">events</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">e</span><span class="p">:</span> <span class="n">e</span><span class="p">[</span><span class="sh">'</span><span class="s">timestamp</span><span class="sh">'</span><span class="p">])</span>

    <span class="n">last_arrival</span> <span class="o">=</span> <span class="n">sorted_by_arrival</span><span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span>
    <span class="n">last_timestamp</span> <span class="o">=</span> <span class="n">sorted_by_timestamp</span><span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span>

    <span class="c1"># Detect clock drift
</span>    <span class="n">clock_drift</span> <span class="o">=</span> <span class="nf">abs</span><span class="p">(</span><span class="n">last_timestamp</span><span class="p">[</span><span class="sh">'</span><span class="s">timestamp</span><span class="sh">'</span><span class="p">]</span> <span class="o">-</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">())</span>
    <span class="n">timestamp_trusted</span> <span class="o">=</span> <span class="n">clock_drift</span> <span class="o">&lt;</span> <span class="mi">3600</span>

    <span class="k">if</span> <span class="n">timestamp_trusted</span><span class="p">:</span>
        <span class="n">authoritative</span> <span class="o">=</span> <span class="n">last_timestamp</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">authoritative</span> <span class="o">=</span> <span class="n">last_arrival</span>

    <span class="c1"># Reconnect window: if the last confirmed reconnect was recent,
</span>    <span class="c1"># a late-arriving disconnect is probably a network artifact
</span>    <span class="n">last_reconnect</span> <span class="o">=</span> <span class="nf">next</span><span class="p">(</span>
        <span class="p">(</span><span class="n">e</span> <span class="k">for</span> <span class="n">e</span> <span class="ow">in</span> <span class="nf">reversed</span><span class="p">(</span><span class="n">sorted_by_arrival</span><span class="p">)</span> <span class="k">if</span> <span class="n">e</span><span class="p">[</span><span class="sh">'</span><span class="s">status</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="sh">'</span><span class="s">online</span><span class="sh">'</span><span class="p">),</span> 
        <span class="bp">None</span>
    <span class="p">)</span>
    <span class="nf">if </span><span class="p">(</span><span class="n">last_reconnect</span> <span class="ow">and</span> 
        <span class="n">authoritative</span><span class="p">[</span><span class="sh">'</span><span class="s">status</span><span class="sh">'</span><span class="p">]</span> <span class="o">==</span> <span class="sh">'</span><span class="s">offline</span><span class="sh">'</span> <span class="ow">and</span>
        <span class="p">(</span><span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span> <span class="o">-</span> <span class="n">last_reconnect</span><span class="p">[</span><span class="sh">'</span><span class="s">arrival_time</span><span class="sh">'</span><span class="p">])</span> <span class="o">&lt;</span> <span class="n">reconnect_window_seconds</span><span class="p">):</span>
        <span class="n">authoritative</span> <span class="o">=</span> <span class="n">last_reconnect</span>

    <span class="k">return</span> <span class="n">authoritative</span>
</code></pre>

</div>



<p>This is better, but it still has a critical gap: <strong>it makes the decision silently.</strong></p>




<h2>
  
  
  The Silent Resolution Problem
</h2>

<p>When your resolution logic overrides a disconnect because it falls inside the reconnect window, your application layer has no idea that happened. It just sees <code>status: online</code>. It cannot distinguish between:</p>

<ul>
<li>A clean online event with high confidence</li>
<li>An offline event that was suppressed because it looked like a network artifact</li>
<li>An offline event that was suppressed incorrectly because your reconnect window is too aggressive</li>
</ul>

<p>This matters enormously when the device controls something physical. A door lock. A valve. An actuator. The difference between "online with high confidence" and "online because we suppressed a suspicious disconnect" should produce different downstream behavior.</p>

<p>The resolution layer needs to return not just the authoritative state, but the basis for that decision:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">resolve_state_with_trace</span><span class="p">(</span><span class="n">events</span><span class="p">,</span> <span class="n">reconnect_window_seconds</span><span class="o">=</span><span class="mi">30</span><span class="p">):</span>
    <span class="c1"># ... resolution logic ...
</span>
    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">'</span><span class="s">authoritative_status</span><span class="sh">'</span><span class="p">:</span> <span class="n">authoritative</span><span class="p">[</span><span class="sh">'</span><span class="s">status</span><span class="sh">'</span><span class="p">],</span>
        <span class="sh">'</span><span class="s">confidence</span><span class="sh">'</span><span class="p">:</span> <span class="n">confidence_score</span><span class="p">,</span>
        <span class="sh">'</span><span class="s">resolution_method</span><span class="sh">'</span><span class="p">:</span> <span class="n">resolution_method</span><span class="p">,</span>  <span class="c1"># 'direct', 'reconnect_supersession', 'drift_compensated'
</span>        <span class="sh">'</span><span class="s">anomaly_signals</span><span class="sh">'</span><span class="p">:</span> <span class="n">anomaly_signals</span><span class="p">,</span>       <span class="c1"># ['clock_drift', 'weak_rf', 'late_disconnect']
</span>        <span class="sh">'</span><span class="s">conflicts_detected</span><span class="sh">'</span><span class="p">:</span> <span class="n">conflicts</span><span class="p">,</span>
        <span class="sh">'</span><span class="s">recommended_action</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">ACT</span><span class="sh">'</span> <span class="k">if</span> <span class="n">confidence</span> <span class="o">&gt;=</span> <span class="mf">0.85</span> <span class="k">else</span> <span class="sh">'</span><span class="s">CONFIRM</span><span class="sh">'</span> <span class="k">if</span> <span class="n">confidence</span> <span class="o">&gt;=</span> <span class="mf">0.65</span> <span class="k">else</span> <span class="sh">'</span><span class="s">LOG_ONLY</span><span class="sh">'</span>
    <span class="p">}</span>
</code></pre>

</div>



<p>Now your application layer can branch on <code>recommended_action</code> instead of implementing its own threshold logic. More importantly, it can implement <strong>hysteresis</strong> against the anomaly signals rather than the raw confidence float.</p>




<h2>
  
  
  Why Hysteresis Belongs in the Application Layer
</h2>

<p>A confidence score of 0.92 from weak RF signal carries different operational meaning than a confidence score of 0.92 from clock drift. The number is the same. The cause is different. The downstream policy should be different.</p>

<p>Weak RF increases the probability of duplicate events and late arrivals. The correct response is to widen your deduplication window and weight sequence numbers more heavily than timestamps.</p>

<p>Clock drift means your timestamps cannot be trusted as an ordering signal at all. The correct response is to fall back entirely to server-side arrival sequencing and flag the resolution as arrival-ordered rather than timestamp-ordered.</p>

<p>If your resolution layer collapses all degradation into a single confidence float, your application layer cannot differentiate these cases. The anomaly signals need to be explicit fields in the resolution response, not compressed into a number.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Don't do this in application logic
</span><span class="k">if</span> <span class="n">resolution</span><span class="p">[</span><span class="sh">'</span><span class="s">confidence</span><span class="sh">'</span><span class="p">]</span> <span class="o">&lt;</span> <span class="mf">0.85</span><span class="p">:</span>
    <span class="nf">hold_for_confirmation</span><span class="p">()</span>

<span class="c1"># Do this instead
</span><span class="k">if</span> <span class="sh">'</span><span class="s">clock_drift</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">resolution</span><span class="p">[</span><span class="sh">'</span><span class="s">anomaly_signals</span><span class="sh">'</span><span class="p">]:</span>
    <span class="c1"># Trust sequence number over timestamp
</span>    <span class="nf">use_arrival_order_policy</span><span class="p">()</span>
<span class="k">elif</span> <span class="sh">'</span><span class="s">weak_rf</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">resolution</span><span class="p">[</span><span class="sh">'</span><span class="s">anomaly_signals</span><span class="sh">'</span><span class="p">]:</span>
    <span class="c1"># Widen dedup window, watch for retry storm
</span>    <span class="nf">apply_rf_degraded_policy</span><span class="p">()</span>
<span class="k">elif</span> <span class="n">resolution</span><span class="p">[</span><span class="sh">'</span><span class="s">confidence</span><span class="sh">'</span><span class="p">]</span> <span class="o">&lt;</span> <span class="mf">0.85</span><span class="p">:</span>
    <span class="c1"># Generic low confidence — no specific signal
</span>    <span class="nf">hold_for_confirmation</span><span class="p">()</span>
</code></pre>

</div>



<p>This is the correct integration pattern. Write your hysteresis logic against the named anomaly signals, not the raw float.</p>




<h2>
  
  
  The Sequence Number Trap
</h2>

<p>One more edge case that breaks most platforms at scale: <strong>the sequence number reset.</strong></p>

<p>Devices restart. When they do, sequence numbers reset to zero or one. If your resolution layer uses sequence numbers to detect late arrivals, a restarted device will look like it is sending events from the past. Every event after restart will be flagged as potentially stale.</p>

<p>The correct handling is to detect the reset explicitly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">detect_sequence_reset</span><span class="p">(</span><span class="n">current_seq</span><span class="p">,</span> <span class="n">last_known_seq</span><span class="p">):</span>
    <span class="c1"># If sequence dropped significantly, assume restart not late arrival
</span>    <span class="k">if</span> <span class="n">last_known_seq</span> <span class="ow">and</span> <span class="n">current_seq</span> <span class="o">&lt;</span> <span class="n">last_known_seq</span><span class="p">:</span>
        <span class="n">drop</span> <span class="o">=</span> <span class="n">last_known_seq</span> <span class="o">-</span> <span class="n">current_seq</span>
        <span class="k">if</span> <span class="n">drop</span> <span class="o">&gt;</span> <span class="mi">100</span><span class="p">:</span>  <span class="c1"># Large drop = restart, not late arrival
</span>            <span class="k">return</span> <span class="bp">True</span>
    <span class="k">return</span> <span class="bp">False</span>
</code></pre>

</div>



<p>And surface it in the resolution trace as a named signal, not a confidence penalty. Your application layer can then treat post-restart events differently — perhaps holding them until the sequence stabilizes — rather than applying a generic low-confidence policy.</p>




<h2>
  
  
  What This Looks Like End to End
</h2>

<p>A well-designed resolution layer takes a raw event, evaluates it against multiple signals, and returns a structured resolution with full trace:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"authoritative_status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"online"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"confidence"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.88</span><span class="p">,</span><span class="w">
  </span><span class="nl">"resolution_method"</span><span class="p">:</span><span class="w"> </span><span class="s2">"reconnect_supersession"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"anomaly_signals"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"weak_rf_signal"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"conflicts_detected"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"Offline event arrived within reconnect window (4.2s) — superseded by confirmed reconnect"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"recommended_action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ACT"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"resolution_basis"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"timestamp_confidence"</span><span class="p">:</span><span class="w"> </span><span class="s2">"high"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"signal_quality"</span><span class="p">:</span><span class="w"> </span><span class="s2">"weak"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"conflicts_resolved"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Every decision is named. Every override is visible. The application layer has everything it needs to implement domain-specific policy without reverse-engineering the resolution logic.</p>




<h2>
  
  
  The Pattern in Three Rules
</h2>

<ol>
<li><p><strong>Separate delivery from resolution.</strong> Your broker delivers events. A separate layer resolves authoritative state from those events. These are different concerns and should be different code.</p></li>
<li><p><strong>Make every arbitration decision explicit.</strong> If you suppress a disconnect, say so. If you fell back to arrival order because timestamps drifted, say so. Silent decisions are invisible bugs.</p></li>
<li><p><strong>Return named signals, not compressed floats.</strong> Your application layer needs to know <em>why</em> confidence is 0.72, not just <em>that</em> it is 0.72. Hysteresis logic, provisional state, and actuation gates all depend on the cause, not the score.</p></li>
</ol>

<p>These three rules will not eliminate every edge case in a distributed IoT system. But they will make the ones that remain visible — which is the only way to fix them before your customer finds them first.</p>




<p><em>Building connected device infrastructure? The author works on device state resolution systems. Reach out on Dev.to or follow for more on distributed IoT patterns.</em></p>

