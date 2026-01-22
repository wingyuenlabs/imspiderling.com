---
Title: PostgreSQL – Storing Time Without Lying to Yourself
Description: 
Author: bwi
Date: 2026-01-22T22:07:23.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 7 of 8</strong> in the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">Time in Software, Done Right</a></p>
</blockquote>




<p>You've modeled time correctly in your application. You have <code>LocalDateTime</code>, <code>TimeZoneId</code>, and <code>Instant</code>. Now you need to persist it.</p>

<p>PostgreSQL has excellent time support — but its type names are misleading, and the default behaviors can surprise you. This article explains what PostgreSQL actually does, which types to use, and how to avoid the common traps.</p>




<h2>
  
  
  The Two Timestamp Types
</h2>

<p>PostgreSQL has two timestamp types:</p>

<ul>
<li>
<code>timestamp without time zone</code> (or just <code>timestamp</code>)</li>
<li>
<code>timestamp with time zone</code> (or <code>timestamptz</code>)</li>
</ul>

<p>The names suggest one stores a timezone and one doesn't. <strong>That's not quite right.</strong></p>

<h3>
  
  
  timestamp without time zone
</h3>

<p>This stores exactly what you give it — a date and time with no timezone context.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">test</span> <span class="p">(</span><span class="n">ts</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">);</span>
<span class="k">SELECT</span> <span class="n">ts</span> <span class="k">FROM</span> <span class="n">test</span><span class="p">;</span>
<span class="c1">-- Result: 2026-06-05 10:00:00</span>
</code></pre>

</div>



<p>No conversion happens. No timezone is stored. It's just a calendar value.</p>

<p><strong>Use for:</strong> <code>LocalDateTime</code> — when you're storing what the user said, not when it happened globally.</p>

<h3>
  
  
  timestamp with time zone
</h3>

<p>This is where the name lies. PostgreSQL does <strong>not</strong> store a timezone. It converts the input to UTC and stores UTC internally. On retrieval, it converts back to the session's timezone.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SET</span> <span class="n">timezone</span> <span class="o">=</span> <span class="s1">'Europe/Vienna'</span><span class="p">;</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">test</span> <span class="p">(</span><span class="n">tstz</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">);</span>

<span class="k">SET</span> <span class="n">timezone</span> <span class="o">=</span> <span class="s1">'Europe/London'</span><span class="p">;</span>
<span class="k">SELECT</span> <span class="n">tstz</span> <span class="k">FROM</span> <span class="n">test</span><span class="p">;</span>
<span class="c1">-- Result: 2026-06-05 04:00:00-04</span>
</code></pre>

</div>



<p>Same row, different display — because PostgreSQL stored UTC internally and converted on output.</p>

<p><strong>Use for:</strong> <code>Instant</code> — when you're storing a global moment.</p>




<h2>
  
  
  What PostgreSQL Actually Stores
</h2>

<p>Let's be precise:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>What's Stored</th>
<th>What Happens on Insert</th>
<th>What Happens on Select</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>timestamp</code></td>
<td>Raw datetime</td>
<td>Nothing</td>
<td>Nothing</td>
</tr>
<tr>
<td><code>timestamptz</code></td>
<td>Instant (normalized to UTC)</td>
<td>Converts input to UTC</td>
<td>Converts UTC to session timezone</td>
</tr>
</tbody>
</table></div>

<p>The critical insight: <strong><code>timestamptz</code> stores UTC, not a timezone.</strong> The "with time zone" means "timezone-aware" — it participates in timezone conversions. It doesn't mean "includes a timezone."</p>




<h2>
  
  
  The Session Timezone Trap
</h2>

<p>With <code>timestamptz</code>, PostgreSQL uses your session's timezone for conversions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SET</span> <span class="n">timezone</span> <span class="o">=</span> <span class="s1">'UTC'</span><span class="p">;</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">events</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">);</span>
<span class="c1">-- Stored as: 2026-06-05 10:00:00 UTC</span>

<span class="k">SET</span> <span class="n">timezone</span> <span class="o">=</span> <span class="s1">'Europe/Vienna'</span><span class="p">;</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">events</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">);</span>
<span class="c1">-- Stored as: 2026-06-05 08:00:00 UTC (Vienna is UTC+2 in summer)</span>
</code></pre>

</div>



<p>Same literal, different stored value — because PostgreSQL assumed the input was in the session timezone.</p>

<p><strong>Best practice:</strong> Always use explicit UTC or offsets when inserting into <code>timestamptz</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">events</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00+00'</span><span class="p">);</span>  <span class="c1">-- Explicit UTC</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">events</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00Z'</span><span class="p">);</span>    <span class="c1">-- Also UTC</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">events</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 12:00:00+02'</span><span class="p">);</span>  <span class="c1">-- Explicit offset</span>
</code></pre>

</div>



<p>Or set your application's session timezone to UTC and keep it there.</p>




<h2>
  
  
  The Recommended Schema
</h2>

<p>For human-scheduled events (meetings, deadlines, appointments), use the pattern from earlier articles:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">appointments</span> <span class="p">(</span>
    <span class="n">id</span>              <span class="n">uuid</span> <span class="k">PRIMARY</span> <span class="k">KEY</span> <span class="k">DEFAULT</span> <span class="n">gen_random_uuid</span><span class="p">(),</span>
    <span class="n">title</span>           <span class="nb">text</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>

    <span class="c1">-- Source of truth: what the user chose</span>
    <span class="n">local_start</span>     <span class="nb">timestamp</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">time_zone_id</span>    <span class="nb">text</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>

    <span class="c1">-- Derived: for global queries and sorting</span>
    <span class="n">instant_utc</span>     <span class="n">timestamptz</span> <span class="k">NOT</span> <span class="k">NULL</span>
<span class="p">);</span>
</code></pre>

</div>



<p><strong>Why three columns?</strong></p>

<ol>
<li>
<code>local_start</code> — The user said "10:00". That's the intent.</li>
<li>
<code>time_zone_id</code> — The user meant "Vienna". That's the context.</li>
<li>
<code>instant_utc</code> — For queries like "what's happening now?" and for sorting.</li>
</ol>

<p>If timezone rules change, you recalculate <code>instant_utc</code> from <code>local_start + time_zone_id</code>.</p>




<h2>
  
  
  Choosing the Right Type for Each Concept
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concept</th>
<th>PostgreSQL Type</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td>Instant / UTC moment</td>
<td><code>timestamptz</code></td>
<td>Log timestamp, <code>created_at</code>
</td>
</tr>
<tr>
<td>Local datetime (user intent)</td>
<td><code>timestamp</code></td>
<td>Meeting time, deadline</td>
</tr>
<tr>
<td>Date only</td>
<td><code>date</code></td>
<td>Birthday, holiday</td>
</tr>
<tr>
<td>Time only</td>
<td><code>time</code></td>
<td>Opening hours</td>
</tr>
<tr>
<td>IANA timezone ID</td>
<td><code>text</code></td>
<td><code>'Europe/Vienna'</code></td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Querying Patterns
</h2>

<h3>
  
  
  Pattern A: "What's on my calendar on June 5th in Vienna?"
</h3>

<p>Query by local date + timezone:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span>
<span class="k">FROM</span> <span class="n">appointments</span>
<span class="k">WHERE</span> <span class="n">time_zone_id</span> <span class="o">=</span> <span class="s1">'Europe/Vienna'</span>
  <span class="k">AND</span> <span class="n">local_start</span> <span class="o">&gt;=</span> <span class="s1">'2026-06-05'</span>
  <span class="k">AND</span> <span class="n">local_start</span> <span class="o">&lt;</span>  <span class="s1">'2026-06-06'</span><span class="p">;</span>
</code></pre>

</div>



<p>This finds all appointments that display as June 5th in Vienna, regardless of the global instant.</p>

<h3>
  
  
  Pattern B: "What's happening globally in the next hour?"
</h3>

<p>Query by instant:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span>
<span class="k">FROM</span> <span class="n">appointments</span>
<span class="k">WHERE</span> <span class="n">instant_utc</span> <span class="o">&gt;=</span> <span class="n">NOW</span><span class="p">()</span>
  <span class="k">AND</span> <span class="n">instant_utc</span> <span class="o">&lt;</span>  <span class="n">NOW</span><span class="p">()</span> <span class="o">+</span> <span class="n">INTERVAL</span> <span class="s1">'1 hour'</span><span class="p">;</span>
</code></pre>

</div>



<p>This finds all appointments happening in the next hour, regardless of their local calendars.</p>

<h3>
  
  
  Pattern C: "What's happening at 10:00 in any timezone?"
</h3>

<p>Query by local time (rare but sometimes needed):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span>
<span class="k">FROM</span> <span class="n">appointments</span>
<span class="k">WHERE</span> <span class="n">local_start</span><span class="p">::</span><span class="nb">time</span> <span class="o">=</span> <span class="s1">'10:00:00'</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  Indexing Strategies
</h2>

<h3>
  
  
  For instant queries (most common)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_appointments_instant</span> <span class="k">ON</span> <span class="n">appointments</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">);</span>
</code></pre>

</div>



<p>This covers "what's happening now" and range queries across timezones.</p>

<h3>
  
  
  For local calendar queries
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_appointments_local</span> <span class="k">ON</span> <span class="n">appointments</span> <span class="p">(</span><span class="n">time_zone_id</span><span class="p">,</span> <span class="n">local_start</span><span class="p">);</span>
</code></pre>

</div>



<p>This covers "what's on the calendar for this timezone" queries. The timezone comes first because you'll almost always filter by it.</p>

<h3>
  
  
  For both
</h3>

<p>If you query both ways heavily, create both indexes. The storage cost is usually worth it.</p>




<h2>
  
  
  Common Mistakes
</h2>

<h3>
  
  
  Mistake 1: Using timestamptz for user intent
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- DON'T: Storing a meeting time as timestamptz</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">meetings</span> <span class="p">(</span><span class="n">starts_at</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">);</span>
</code></pre>

</div>



<p>You've lost the "10:00" intent. If timezone rules change, you can't recover it.</p>

<h3>
  
  
  Mistake 2: Using timestamp for log timestamps
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- DON'T: Storing a log timestamp without timezone</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">logs</span> <span class="p">(</span><span class="n">occurred_at</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">);</span>
</code></pre>

</div>



<p>Is that UTC? Server time? You don't know. Use <code>timestamptz</code> and be explicit.</p>

<h3>
  
  
  Mistake 3: Trusting session timezone
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- DON'T: Assuming session timezone is what you expect</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">events</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">);</span>
</code></pre>

</div>



<p>What timezone was the session in? Be explicit: <code>'2026-06-05 10:00:00+00'</code>.</p>

<h3>
  
  
  Mistake 4: Storing timezone names in timestamptz
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- DON'T: Thinking this stores "Vienna"</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">events</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">)</span> <span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00 Europe/Vienna'</span><span class="p">);</span>
</code></pre>

</div>



<p>PostgreSQL converts to UTC immediately. The string <code>'Europe/Vienna'</code> is gone. If you need the timezone, store it separately.</p>




<h2>
  
  
  AT TIME ZONE: The Conversion Operator
</h2>

<p>PostgreSQL's <code>AT TIME ZONE</code> converts between timestamps and timezones:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- timestamptz → timestamp in a specific zone</span>
<span class="k">SELECT</span> <span class="n">instant_utc</span> <span class="k">AT</span> <span class="nb">TIME</span> <span class="k">ZONE</span> <span class="s1">'Europe/Vienna'</span> <span class="k">AS</span> <span class="n">local_vienna</span>
<span class="k">FROM</span> <span class="n">appointments</span><span class="p">;</span>

<span class="c1">-- timestamp → timestamptz (interpreting as a specific zone)</span>
<span class="k">SELECT</span> <span class="n">local_start</span> <span class="k">AT</span> <span class="nb">TIME</span> <span class="k">ZONE</span> <span class="n">time_zone_id</span> <span class="k">AS</span> <span class="n">instant</span>
<span class="k">FROM</span> <span class="n">appointments</span><span class="p">;</span>
</code></pre>

</div>



<p>This is useful for display and for reconstructing the instant from stored local + timezone.</p>

<p><strong>Gotcha:</strong> The behavior differs based on the input type:</p>

<ul>
<li>
<code>timestamptz AT TIME ZONE 'X'</code> → returns <code>timestamp</code> (strips timezone, shows in X)</li>
<li>
<code>timestamp AT TIME ZONE 'X'</code> → returns <code>timestamptz</code> (interprets as X, converts to UTC)</li>
</ul>




<h2>
  
  
  Handling Timezone Rule Changes
</h2>

<p>When IANA updates timezone rules:</p>

<ol>
<li><p><strong>Past events</strong>: Nothing to do. PostgreSQL's <code>timestamptz</code> already stores UTC. Historical conversions use historical rules (if your system's tzdata is updated).</p></li>
<li><p><strong>Future events</strong>: Recalculate <code>instant_utc</code> from <code>local_start + time_zone_id</code>.<br>
</p></li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Recalculate instant_utc for future Vienna appointments</span>
<span class="k">UPDATE</span> <span class="n">appointments</span>
<span class="k">SET</span> <span class="n">instant_utc</span> <span class="o">=</span> <span class="n">local_start</span> <span class="k">AT</span> <span class="nb">TIME</span> <span class="k">ZONE</span> <span class="n">time_zone_id</span>
<span class="k">WHERE</span> <span class="n">time_zone_id</span> <span class="o">=</span> <span class="s1">'Europe/Vienna'</span>
  <span class="k">AND</span> <span class="n">instant_utc</span> <span class="o">&gt;</span> <span class="n">NOW</span><span class="p">();</span>
</code></pre>

</div>



<p>This is why storing <code>local_start + time_zone_id</code> matters — you have everything needed to recalculate.</p>




<h2>
  
  
  Working with EF Core and Npgsql
</h2>

<p>If you're using .NET with Npgsql, the mapping is straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// In your DbContext</span>
<span class="k">protected</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">OnConfiguring</span><span class="p">(</span><span class="n">DbContextOptionsBuilder</span> <span class="n">optionsBuilder</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">optionsBuilder</span><span class="p">.</span><span class="nf">UseNpgsql</span><span class="p">(</span><span class="n">connectionString</span><span class="p">,</span> <span class="n">o</span> <span class="p">=&gt;</span> <span class="n">o</span><span class="p">.</span><span class="nf">UseNodaTime</span><span class="p">());</span>
<span class="p">}</span>
</code></pre>

</div>



<p>With <code>UseNodaTime()</code>:</p>

<ul>
<li>
<code>Instant</code> ↔ <code>timestamptz</code>
</li>
<li>
<code>LocalDateTime</code> ↔ <code>timestamp</code>
</li>
<li>
<code>LocalDate</code> ↔ <code>date</code>
</li>
<li>
<code>LocalTime</code> ↔ <code>time</code>
</li>
</ul>

<p>The types align naturally with our model.</p>




<h2>
  
  
  Full Example: Creating and Querying Appointments
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- gen_random_uuid() requires this extension</span>
<span class="k">CREATE</span> <span class="n">EXTENSION</span> <span class="n">IF</span> <span class="k">NOT</span> <span class="k">EXISTS</span> <span class="n">pgcrypto</span><span class="p">;</span>

<span class="c1">-- Create table</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">appointments</span> <span class="p">(</span>
    <span class="n">id</span>              <span class="n">uuid</span> <span class="k">PRIMARY</span> <span class="k">KEY</span> <span class="k">DEFAULT</span> <span class="n">gen_random_uuid</span><span class="p">(),</span>
    <span class="n">title</span>           <span class="nb">text</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">local_start</span>     <span class="nb">timestamp</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">time_zone_id</span>    <span class="nb">text</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">instant_utc</span>     <span class="n">timestamptz</span> <span class="k">NOT</span> <span class="k">NULL</span>
<span class="p">);</span>

<span class="c1">-- Create indexes</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_appointments_instant</span> <span class="k">ON</span> <span class="n">appointments</span> <span class="p">(</span><span class="n">instant_utc</span><span class="p">);</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_appointments_local</span> <span class="k">ON</span> <span class="n">appointments</span> <span class="p">(</span><span class="n">time_zone_id</span><span class="p">,</span> <span class="n">local_start</span><span class="p">);</span>

<span class="c1">-- Insert an appointment (10:00 Vienna = 08:00 UTC in summer)</span>
<span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">appointments</span> <span class="p">(</span><span class="n">title</span><span class="p">,</span> <span class="n">local_start</span><span class="p">,</span> <span class="n">time_zone_id</span><span class="p">,</span> <span class="n">instant_utc</span><span class="p">)</span>
<span class="k">VALUES</span> <span class="p">(</span>
    <span class="s1">'Team Standup'</span><span class="p">,</span>
    <span class="s1">'2026-06-05 10:00:00'</span><span class="p">,</span>
    <span class="s1">'Europe/Vienna'</span><span class="p">,</span>
    <span class="s1">'2026-06-05 10:00:00'</span> <span class="k">AT</span> <span class="nb">TIME</span> <span class="k">ZONE</span> <span class="s1">'Europe/Vienna'</span>
<span class="p">);</span>

<span class="c1">-- Query: What's on the Vienna calendar for June 5th?</span>
<span class="k">SELECT</span> <span class="n">title</span><span class="p">,</span> <span class="n">local_start</span>
<span class="k">FROM</span> <span class="n">appointments</span>
<span class="k">WHERE</span> <span class="n">time_zone_id</span> <span class="o">=</span> <span class="s1">'Europe/Vienna'</span>
  <span class="k">AND</span> <span class="n">local_start</span> <span class="o">&gt;=</span> <span class="s1">'2026-06-05'</span>
  <span class="k">AND</span> <span class="n">local_start</span> <span class="o">&lt;</span>  <span class="s1">'2026-06-06'</span><span class="p">;</span>

<span class="c1">-- Query: What's happening globally in the next 2 hours?</span>
<span class="k">SELECT</span> <span class="n">title</span><span class="p">,</span> <span class="n">instant_utc</span><span class="p">,</span> <span class="n">time_zone_id</span>
<span class="k">FROM</span> <span class="n">appointments</span>
<span class="k">WHERE</span> <span class="n">instant_utc</span> <span class="o">&gt;=</span> <span class="n">NOW</span><span class="p">()</span>
  <span class="k">AND</span> <span class="n">instant_utc</span> <span class="o">&lt;</span>  <span class="n">NOW</span><span class="p">()</span> <span class="o">+</span> <span class="n">INTERVAL</span> <span class="s1">'2 hours'</span><span class="p">;</span>

<span class="c1">-- Display in viewer's timezone</span>
<span class="k">SELECT</span> 
    <span class="n">title</span><span class="p">,</span>
    <span class="n">instant_utc</span> <span class="k">AT</span> <span class="nb">TIME</span> <span class="k">ZONE</span> <span class="s1">'Europe/London'</span> <span class="k">AS</span> <span class="n">starts_at_london</span>
<span class="k">FROM</span> <span class="n">appointments</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  A Note on ORMs, Query Builders, and Event Stores
</h2>

<p>The PostgreSQL model described here — storing <code>local_start</code>, <code>time_zone_id</code>, and a derived <code>instant_utc</code> — is independent of how you access the database.</p>

<ul>
<li>
<strong>EF Core / Npgsql</strong>: Works well with explicit mappings (see Article 6 for full NodaTime integration).</li>
<li>
<strong>Dapper</strong>: Maps naturally to simple columns; you compute instants in application code.</li>
<li>
<strong>Marten / Event Sourcing</strong>: Events typically store an <code>Instant</code> (<code>occurred_at</code>) plus domain-specific local values when needed.</li>
<li>
<strong>Raw SQL</strong>: The same rules apply — PostgreSQL doesn't care how the data got there.</li>
</ul>

<p>The key idea is not the ORM — it's the data model.</p>

<p>If you store human intent (<code>local + timezone</code>) separately from physical moments (<code>instant</code>), the approach works across tools, frameworks, and architectural styles.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>
<strong><code>timestamp</code></strong> stores raw datetime — use for <code>LocalDateTime</code> (user intent)</li>
<li>
<strong><code>timestamptz</code></strong> converts to/from UTC — use for <code>Instant</code> (global moments)</li>
<li>
<strong><code>timestamptz</code> does not store a timezone</strong> — it stores UTC and converts on read</li>
<li>
<strong>For human-scheduled events</strong>: store <code>local_start</code> + <code>time_zone_id</code> + <code>instant_utc</code>
</li>
<li>
<strong>Be explicit</strong> with timezones on insert — don't trust session settings</li>
<li>
<strong>Index <code>instant_utc</code></strong> for global queries, <strong>index <code>(time_zone_id, local_start)</code></strong> for calendar queries</li>
<li>
<strong>When rules change</strong>: recalculate <code>instant_utc</code> from the stored local + timezone</li>
</ul>




<p><em>Next up: <a href="https://dev.to/bwi/frontend-temporal-apis-and-datetimepickers-that-dont-lie-6dn"><strong>Frontend – Temporal, APIs, and DateTimePickers That Don't Lie</strong></a> — bringing it all together in the browser.</em></p>

