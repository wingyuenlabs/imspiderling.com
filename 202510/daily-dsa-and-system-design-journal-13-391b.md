---
Title: Daily DSA and System Design Journal - 13
Description: 
Author: I.K
Date: 2025-10-28T21:44:36.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🧠 Day 13 — Bitmasking &amp; Background Job Communication ⚙️
</h2>




<h3>
  
  
  🧩 DSA Problems [1 hr]
</h3>

<p><strong>Problem:</strong> <a href="https://leetcode.com/problems/maximize-the-number-of-partitions-after-operations/" rel="noopener noreferrer">3003. Maximize the Number of Partitions After Operations</a></p>

<h4>
  
  
  💡 Approach: Bitwise Operations + Preprocessing + Enumeration
</h4>

<h5>
  
  
  🧠 Intuition
</h5>

<p>We can modify <strong>at most one character</strong> in the string.<br>
Without modifications, we can compute all valid partitions easily.<br>
When we introduce one modification, only its <strong>local segment</strong> changes — everything before and after it remains unaffected.</p>

<p>To analyze this efficiently:</p>

<ul>
<li>We <strong>split the problem</strong> into a <strong>left segment</strong> and <strong>right segment</strong> around the position of modification.</li>
<li>
<p>Each segment tracks:</p>

<ul>
<li>🧩 Number of splits</li>
<li>🧮 Character mask (bitwise representation)</li>
<li>🔢 Count of distinct characters</li>
</ul>


</li>

</ul>

<p>Using bit operations lets us quickly track which characters exist in each split.</p>

<p>We then simulate all modification positions and compute the best outcome.</p>




<h4>
  
  
  💻 Code
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">Solution</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">maxPartitionsAfterOperations</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">s</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">k</span><span class="p">:</span> <span class="nb">int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">int</span><span class="p">:</span>
        <span class="n">n</span> <span class="o">=</span> <span class="nf">len</span><span class="p">(</span><span class="n">s</span><span class="p">)</span>
        <span class="n">left</span> <span class="o">=</span> <span class="p">[[</span><span class="mi">0</span><span class="p">]</span> <span class="o">*</span> <span class="mi">3</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">n</span><span class="p">)]</span>
        <span class="n">right</span> <span class="o">=</span> <span class="p">[[</span><span class="mi">0</span><span class="p">]</span> <span class="o">*</span> <span class="mi">3</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">n</span><span class="p">)]</span>

        <span class="n">num</span><span class="p">,</span> <span class="n">mask</span><span class="p">,</span> <span class="n">count</span> <span class="o">=</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span>
        <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">n</span> <span class="o">-</span> <span class="mi">1</span><span class="p">):</span>
            <span class="n">binary</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">&lt;&lt;</span> <span class="p">(</span><span class="nf">ord</span><span class="p">(</span><span class="n">s</span><span class="p">[</span><span class="n">i</span><span class="p">])</span> <span class="o">-</span> <span class="nf">ord</span><span class="p">(</span><span class="sh">"</span><span class="s">a</span><span class="sh">"</span><span class="p">))</span>
            <span class="k">if</span> <span class="ow">not</span> <span class="p">(</span><span class="n">mask</span> <span class="o">&amp;</span> <span class="n">binary</span><span class="p">):</span>
                <span class="n">count</span> <span class="o">+=</span> <span class="mi">1</span>
                <span class="k">if</span> <span class="n">count</span> <span class="o">&lt;=</span> <span class="n">k</span><span class="p">:</span>
                    <span class="n">mask</span> <span class="o">|=</span> <span class="n">binary</span>
                <span class="k">else</span><span class="p">:</span>
                    <span class="n">num</span> <span class="o">+=</span> <span class="mi">1</span>
                    <span class="n">mask</span> <span class="o">=</span> <span class="n">binary</span>
                    <span class="n">count</span> <span class="o">=</span> <span class="mi">1</span>
            <span class="n">left</span><span class="p">[</span><span class="n">i</span> <span class="o">+</span> <span class="mi">1</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">=</span> <span class="n">num</span>
            <span class="n">left</span><span class="p">[</span><span class="n">i</span> <span class="o">+</span> <span class="mi">1</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span> <span class="o">=</span> <span class="n">mask</span>
            <span class="n">left</span><span class="p">[</span><span class="n">i</span> <span class="o">+</span> <span class="mi">1</span><span class="p">][</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="n">count</span>

        <span class="n">num</span><span class="p">,</span> <span class="n">mask</span><span class="p">,</span> <span class="n">count</span> <span class="o">=</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">0</span>
        <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">n</span> <span class="o">-</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">):</span>
            <span class="n">binary</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">&lt;&lt;</span> <span class="p">(</span><span class="nf">ord</span><span class="p">(</span><span class="n">s</span><span class="p">[</span><span class="n">i</span><span class="p">])</span> <span class="o">-</span> <span class="nf">ord</span><span class="p">(</span><span class="sh">"</span><span class="s">a</span><span class="sh">"</span><span class="p">))</span>
            <span class="k">if</span> <span class="ow">not</span> <span class="p">(</span><span class="n">mask</span> <span class="o">&amp;</span> <span class="n">binary</span><span class="p">):</span>
                <span class="n">count</span> <span class="o">+=</span> <span class="mi">1</span>
                <span class="k">if</span> <span class="n">count</span> <span class="o">&lt;=</span> <span class="n">k</span><span class="p">:</span>
                    <span class="n">mask</span> <span class="o">|=</span> <span class="n">binary</span>
                <span class="k">else</span><span class="p">:</span>
                    <span class="n">num</span> <span class="o">+=</span> <span class="mi">1</span>
                    <span class="n">mask</span> <span class="o">=</span> <span class="n">binary</span>
                    <span class="n">count</span> <span class="o">=</span> <span class="mi">1</span>
            <span class="n">right</span><span class="p">[</span><span class="n">i</span> <span class="o">-</span> <span class="mi">1</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">=</span> <span class="n">num</span>
            <span class="n">right</span><span class="p">[</span><span class="n">i</span> <span class="o">-</span> <span class="mi">1</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span> <span class="o">=</span> <span class="n">mask</span>
            <span class="n">right</span><span class="p">[</span><span class="n">i</span> <span class="o">-</span> <span class="mi">1</span><span class="p">][</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="n">count</span>

        <span class="n">max_val</span> <span class="o">=</span> <span class="mi">0</span>
        <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">n</span><span class="p">):</span>
            <span class="n">seg</span> <span class="o">=</span> <span class="n">left</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">+</span> <span class="n">right</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span> <span class="o">+</span> <span class="mi">2</span>
            <span class="n">tot_mask</span> <span class="o">=</span> <span class="n">left</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span> <span class="o">|</span> <span class="n">right</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span>
            <span class="n">tot_count</span> <span class="o">=</span> <span class="nf">bin</span><span class="p">(</span><span class="n">tot_mask</span><span class="p">).</span><span class="nf">count</span><span class="p">(</span><span class="sh">"</span><span class="s">1</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">if</span> <span class="n">left</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="mi">2</span><span class="p">]</span> <span class="o">==</span> <span class="n">k</span> <span class="ow">and</span> <span class="n">right</span><span class="p">[</span><span class="n">i</span><span class="p">][</span><span class="mi">2</span><span class="p">]</span> <span class="o">==</span> <span class="n">k</span> <span class="ow">and</span> <span class="n">tot_count</span> <span class="o">&lt;</span> <span class="mi">26</span><span class="p">:</span>
                <span class="n">seg</span> <span class="o">+=</span> <span class="mi">1</span>
            <span class="k">elif</span> <span class="nf">min</span><span class="p">(</span><span class="n">tot_count</span> <span class="o">+</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">26</span><span class="p">)</span> <span class="o">&lt;=</span> <span class="n">k</span><span class="p">:</span>
                <span class="n">seg</span> <span class="o">-=</span> <span class="mi">1</span>
            <span class="n">max_val</span> <span class="o">=</span> <span class="nf">max</span><span class="p">(</span><span class="n">max_val</span><span class="p">,</span> <span class="n">seg</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">max_val</span>
</code></pre>

</div>






<h4>
  
  
  ⚙️ Key Learnings
</h4>

<ul>
<li>Bitmasking enables <strong>constant-time distinct character tracking</strong>.</li>
<li>Dividing problems into <strong>left and right contexts</strong> simplifies otherwise complex state management.</li>
<li>Enumerating possible modifications systematically ensures you don’t miss edge cases.</li>
</ul>




<h3>
  
  
  🏗 System Design — Roadmap.sh [1 hr]
</h3>

<h4>
  
  
  💬 Returning Results from Background Jobs
</h4>

<p>Background jobs often run <strong>asynchronously</strong> — decoupled from the main request-response cycle.<br>
But what happens when the <strong>caller</strong> needs to know if the background task:</p>

<ul>
<li>finished successfully? ✅</li>
<li>failed? ❌</li>
<li>or made progress? 🔄</li>
</ul>

<p>That’s where <strong>communication mechanisms</strong> come in.</p>




<h4>
  
  
  🧩 Common Strategies
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>🧾 <strong>Status Store</strong>
</td>
<td>The job writes its progress or results to a shared datastore (DB, Redis). The UI polls or queries it.</td>
</tr>
<tr>
<td>📬 <strong>Reply Queue</strong>
</td>
<td>A “reply-to” queue receives messages from the job — includes <code>ReplyTo</code> and <code>CorrelationId</code> for mapping results to requests.</td>
</tr>
<tr>
<td>🌐 <strong>API Endpoint</strong>
</td>
<td>The job exposes an API for clients to request status updates or results.</td>
</tr>
<tr>
<td>🔔 <strong>Callback / Webhook</strong>
</td>
<td>The job calls back to the client via API or publishes an event when it completes.</td>
</tr>
</tbody>
</table></div>




<h4>
  
  
  🧠 Example Scenarios
</h4>

<ul>
<li>🧮 <strong>Report Generation:</strong> The user triggers a report → background job runs → result stored in S3 → UI polls until ready.</li>
<li>🔄 <strong>Machine Learning Pipeline:</strong> Background task publishes progress to a queue; dashboard listens for updates.</li>
<li>📦 <strong>Order Processing:</strong> System publishes “order completed” event via a message bus; microservices subscribe to react accordingly.</li>
</ul>




<h4>
  
  
  ⚠️ Design Considerations
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concern</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>🧱 <strong>Reliability</strong>
</td>
<td>Messages or writes can fail — implement retries and acknowledgements.</td>
</tr>
<tr>
<td>🧩 <strong>Idempotence</strong>
</td>
<td>Duplicate completion messages shouldn’t cause inconsistencies.</td>
</tr>
<tr>
<td>⏳ <strong>Timeouts &amp; TTLs</strong>
</td>
<td>Define how long clients should wait for results.</td>
</tr>
<tr>
<td>📡 <strong>Scalability</strong>
</td>
<td>For large systems, prefer async event-driven feedback over polling.</td>
</tr>
</tbody>
</table></div>




<h4>
  
  
  🧠 Reflection
</h4>

<p>This day beautifully mirrors the DSA concept:</p>

<ul>
<li>In the algorithm, you used <strong>bit-level precision</strong> to track states efficiently.</li>
<li>In system design, you used <strong>communication-level precision</strong> to synchronize async processes.</li>
</ul>

<p>Both are about <strong>smart tracking and controlled feedback</strong> — small signals, big coordination.</p>




<p>✅ <strong>Day 13 Summary:</strong></p>

<blockquote>
<p>Progress isn’t just computation — it’s communication. Systems (and developers) that <em>report back effectively</em> scale gracefully. 🚀</p>
</blockquote>

