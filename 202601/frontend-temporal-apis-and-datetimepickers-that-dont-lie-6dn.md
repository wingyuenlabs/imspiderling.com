---
Title: Frontend – Temporal, APIs, and DateTimePickers That Don't Lie
Description: 
Author: bwi
Date: 2026-01-22T22:10:07.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 8 of 8</strong> in the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">Time in Software, Done Right</a></p>
</blockquote>




<p>You've modeled time correctly on the backend. You've stored it properly in the database. Now you need to handle it in the browser — where users pick dates and times, and where your API sends data back and forth.</p>

<p>JavaScript's <code>Date</code> object has been the source of countless bugs. The Temporal API finally gives us proper types. But even with good types, you still need to think about what your DateTimePicker is actually asking users to select, and how to send that data across the wire.</p>

<p>This article covers the Temporal API, API contract design, and the principles behind DateTimePickers that don't mislead users.</p>




<h2>
  
  
  Why JavaScript's Date Is Problematic
</h2>

<p>The <code>Date</code> object has been with us since 1995. It has... issues:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Months are 0-indexed (January = 0)</span>
<span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="mi">2026</span><span class="p">,</span> <span class="mi">5</span><span class="p">,</span> <span class="mi">5</span><span class="p">)</span>  <span class="c1">// June 5th, not May 5th</span>

<span class="c1">// Parsing is inconsistent across browsers</span>
<span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="dl">"</span><span class="s2">2026-06-05</span><span class="dl">"</span><span class="p">)</span>  <span class="c1">// Midnight UTC? Midnight local? Depends on browser.</span>

<span class="c1">// No timezone support beyond local and UTC</span>
<span class="kd">const</span> <span class="nx">d</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">();</span>
<span class="nx">d</span><span class="p">.</span><span class="nf">getTimezoneOffset</span><span class="p">();</span>  <span class="c1">// Minutes offset, but which zone? You don't know.</span>

<span class="c1">// Mutable (a constant footgun)</span>
<span class="kd">const</span> <span class="nx">d</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">();</span>
<span class="nx">d</span><span class="p">.</span><span class="nf">setMonth</span><span class="p">(</span><span class="mi">11</span><span class="p">);</span>  <span class="c1">// Mutates in place</span>
</code></pre>

</div>



<p>There's no <code>LocalDate</code>, no <code>LocalDateTime</code>, no <code>ZonedDateTime</code>. Just one type that tries to do everything and does none of it well.</p>




<h2>
  
  
  The Temporal API
</h2>

<p>The Temporal API is the modern replacement for <code>Date</code>. It's currently at Stage 3 — the final candidate stage before standardization — and requires a polyfill in most browsers (e.g., <code>@js-temporal/polyfill</code>). Browser support is coming, but for now, plan on using the polyfill.</p>

<h3>
  
  
  Type Mapping
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concept</th>
<th>Temporal Type</th>
<th>NodaTime Equivalent</th>
</tr>
</thead>
<tbody>
<tr>
<td>Physical moment</td>
<td><code>Temporal.Instant</code></td>
<td><code>Instant</code></td>
</tr>
<tr>
<td>Calendar date</td>
<td><code>Temporal.PlainDate</code></td>
<td><code>LocalDate</code></td>
</tr>
<tr>
<td>Wall clock time</td>
<td><code>Temporal.PlainTime</code></td>
<td><code>LocalTime</code></td>
</tr>
<tr>
<td>Date + time (no zone)</td>
<td><code>Temporal.PlainDateTime</code></td>
<td><code>LocalDateTime</code></td>
</tr>
<tr>
<td>IANA timezone</td>
<td><code>Temporal.TimeZone</code></td>
<td><code>DateTimeZone</code></td>
</tr>
<tr>
<td>Full context</td>
<td><code>Temporal.ZonedDateTime</code></td>
<td><code>ZonedDateTime</code></td>
</tr>
</tbody>
</table></div>

<p>The names differ (<code>Plain</code> vs <code>Local</code>), but the concepts are identical. If you've understood NodaTime, you already understand Temporal.</p>

<h3>
  
  
  Basic Usage
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Just a date</span>
<span class="kd">const</span> <span class="nx">date</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">PlainDate</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">2026-06-05</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">date2</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nc">PlainDate</span><span class="p">(</span><span class="mi">2026</span><span class="p">,</span> <span class="mi">6</span><span class="p">,</span> <span class="mi">5</span><span class="p">);</span>  <span class="c1">// Months are 1-indexed!</span>

<span class="c1">// Just a time</span>
<span class="kd">const</span> <span class="nx">time</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">PlainTime</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">10:00</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Date and time (no zone)</span>
<span class="kd">const</span> <span class="nx">local</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">PlainDateTime</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">2026-06-05T10:00</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// With timezone</span>
<span class="kd">const</span> <span class="nx">zone</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">TimeZone</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">Europe/Vienna</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">zoned</span> <span class="o">=</span> <span class="nx">local</span><span class="p">.</span><span class="nf">toZonedDateTime</span><span class="p">(</span><span class="nx">zone</span><span class="p">);</span>

<span class="c1">// Get the instant</span>
<span class="kd">const</span> <span class="nx">instant</span> <span class="o">=</span> <span class="nx">zoned</span><span class="p">.</span><span class="nf">toInstant</span><span class="p">();</span>
</code></pre>

</div>



<h3>
  
  
  Converting Between Types
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// ZonedDateTime → components</span>
<span class="kd">const</span> <span class="nx">zoned</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">ZonedDateTime</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">2026-06-05T10:00[Europe/Vienna]</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">local</span> <span class="o">=</span> <span class="nx">zoned</span><span class="p">.</span><span class="nf">toPlainDateTime</span><span class="p">();</span>     <span class="c1">// PlainDateTime</span>
<span class="kd">const</span> <span class="nx">instant</span> <span class="o">=</span> <span class="nx">zoned</span><span class="p">.</span><span class="nf">toInstant</span><span class="p">();</span>          <span class="c1">// Instant</span>
<span class="kd">const</span> <span class="nx">tzId</span> <span class="o">=</span> <span class="nx">zoned</span><span class="p">.</span><span class="nx">timeZoneId</span><span class="p">;</span>              <span class="c1">// "Europe/Vienna"</span>

<span class="c1">// Instant → ZonedDateTime (for display)</span>
<span class="kd">const</span> <span class="nx">instant</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">Instant</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">2026-06-05T08:00:00Z</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">inVienna</span> <span class="o">=</span> <span class="nx">instant</span><span class="p">.</span><span class="nf">toZonedDateTimeISO</span><span class="p">(</span><span class="dl">"</span><span class="s2">Europe/Vienna</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">inLondon</span> <span class="o">=</span> <span class="nx">instant</span><span class="p">.</span><span class="nf">toZonedDateTimeISO</span><span class="p">(</span><span class="dl">"</span><span class="s2">Europe/London</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  DST Handling
</h3>

<p>Temporal handles DST ambiguities explicitly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Time that doesn't exist (spring forward gap)</span>
<span class="kd">const</span> <span class="nx">local</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">PlainDateTime</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">2026-03-29T02:30</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">zone</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">TimeZone</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">Europe/Vienna</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Default ("compatible"): shifts forward for gaps, picks earlier occurrence for overlaps</span>
<span class="kd">const</span> <span class="nx">zoned</span> <span class="o">=</span> <span class="nx">local</span><span class="p">.</span><span class="nf">toZonedDateTime</span><span class="p">(</span><span class="nx">zone</span><span class="p">,</span> <span class="p">{</span> <span class="na">disambiguation</span><span class="p">:</span> <span class="dl">"</span><span class="s2">compatible</span><span class="dl">"</span> <span class="p">});</span>
<span class="kd">const</span> <span class="nx">zoned</span> <span class="o">=</span> <span class="nx">local</span><span class="p">.</span><span class="nf">toZonedDateTime</span><span class="p">(</span><span class="nx">zone</span><span class="p">,</span> <span class="p">{</span> <span class="na">disambiguation</span><span class="p">:</span> <span class="dl">"</span><span class="s2">reject</span><span class="dl">"</span> <span class="p">});</span>      <span class="c1">// Throws</span>

<span class="c1">// Time that exists twice (fall back overlap)</span>
<span class="kd">const</span> <span class="nx">local</span> <span class="o">=</span> <span class="nx">Temporal</span><span class="p">.</span><span class="nx">PlainDateTime</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">"</span><span class="s2">2026-10-25T02:30</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">zoned</span> <span class="o">=</span> <span class="nx">local</span><span class="p">.</span><span class="nf">toZonedDateTime</span><span class="p">(</span><span class="nx">zone</span><span class="p">,</span> <span class="p">{</span> <span class="na">disambiguation</span><span class="p">:</span> <span class="dl">"</span><span class="s2">earlier</span><span class="dl">"</span> <span class="p">});</span>  <span class="c1">// First occurrence</span>
<span class="kd">const</span> <span class="nx">zoned</span> <span class="o">=</span> <span class="nx">local</span><span class="p">.</span><span class="nf">toZonedDateTime</span><span class="p">(</span><span class="nx">zone</span><span class="p">,</span> <span class="p">{</span> <span class="na">disambiguation</span><span class="p">:</span> <span class="dl">"</span><span class="s2">later</span><span class="dl">"</span> <span class="p">});</span>    <span class="c1">// Second occurrence</span>
</code></pre>

</div>






<h2>
  
  
  API Contracts: Sending Time Across the Wire
</h2>

<p>When your frontend talks to your backend, you need a clear contract for time values. There are several approaches.</p>

<h3>
  
  
  Option 1: ISO Strings (Simple)
</h3>

<p>For instants, use ISO 8601 with Z suffix:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"createdAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T08:00:00Z"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Unambiguous. Both sides parse it the same way.</p>

<h3>
  
  
  Option 2: Structured Object (Recommended for User Intent)
</h3>

<p>For human-scheduled times, send the components:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"appointment"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"localStart"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T10:00:00"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"timeZoneId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Europe/Vienna"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The backend receives exactly what the user chose. It can:</p>

<ul>
<li>Validate the timezone ID</li>
<li>Handle DST ambiguities with domain-specific logic</li>
<li>Compute the instant</li>
<li>Store all three values</li>
</ul>

<h3>
  
  
  Option 3: ZonedDateTime String
</h3>

<p>Temporal and some APIs support bracketed timezone notation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"startsAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T10:00:00[Europe/Vienna]"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This is compact and unambiguous, but not all JSON parsers handle it natively. You'll need custom parsing.</p>

<h3>
  
  
  What NOT to Do
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">DON'T:</span><span class="w"> </span><span class="err">Ambiguous</span><span class="w"> </span><span class="err">local</span><span class="w"> </span><span class="err">time</span><span class="w">
</span><span class="p">{</span><span class="w"> </span><span class="nl">"startsAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T10:00:00"</span><span class="w"> </span><span class="p">}</span><span class="w">  </span><span class="err">//</span><span class="w"> </span><span class="err">What</span><span class="w"> </span><span class="err">timezone?</span><span class="w">

</span><span class="err">//</span><span class="w"> </span><span class="err">DON'T:</span><span class="w"> </span><span class="err">Offset</span><span class="w"> </span><span class="err">without</span><span class="w"> </span><span class="err">timezone</span><span class="w"> </span><span class="err">ID</span><span class="w">
</span><span class="p">{</span><span class="w"> </span><span class="nl">"startsAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T10:00:00+02:00"</span><span class="w"> </span><span class="p">}</span><span class="w">  </span><span class="err">//</span><span class="w"> </span><span class="err">Which</span><span class="w"> </span><span class="err">+</span><span class="mi">02</span><span class="err">:</span><span class="mi">00</span><span class="w"> </span><span class="err">zone?</span><span class="w">

</span><span class="err">//</span><span class="w"> </span><span class="err">DON'T:</span><span class="w"> </span><span class="err">Unix</span><span class="w"> </span><span class="err">timestamp</span><span class="w"> </span><span class="err">for</span><span class="w"> </span><span class="err">user-scheduled</span><span class="w"> </span><span class="err">events</span><span class="w">
</span><span class="p">{</span><span class="w"> </span><span class="nl">"startsAt"</span><span class="p">:</span><span class="w"> </span><span class="mi">1780758000</span><span class="w"> </span><span class="p">}</span><span class="w">  </span><span class="err">//</span><span class="w"> </span><span class="err">Lost</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">user's</span><span class="w"> </span><span class="err">intent</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  TypeScript Interfaces
</h2>

<p>Define clear types for your API:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// For instants (logs, events, timestamps)</span>
<span class="kr">interface</span> <span class="nx">AuditEvent</span> <span class="p">{</span>
  <span class="nl">occurredAt</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>  <span class="c1">// ISO 8601 with Z: "2026-06-05T08:00:00Z"</span>
<span class="p">}</span>

<span class="c1">// For user-scheduled times</span>
<span class="kr">interface</span> <span class="nx">ScheduledTime</span> <span class="p">{</span>
  <span class="nl">local</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>       <span class="c1">// ISO 8601 without offset: "2026-06-05T10:00:00"</span>
  <span class="nl">timeZoneId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>  <span class="c1">// IANA zone: "Europe/Vienna"</span>
<span class="p">}</span>

<span class="kr">interface</span> <span class="nx">Appointment</span> <span class="p">{</span>
  <span class="nl">title</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">start</span><span class="p">:</span> <span class="nx">ScheduledTime</span><span class="p">;</span>
  <span class="nl">end</span><span class="p">:</span> <span class="nx">ScheduledTime</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// For date-only values</span>
<span class="kr">interface</span> <span class="nx">Person</span> <span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">dateOfBirth</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>  <span class="c1">// ISO 8601 date: "1990-03-15"</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  DateTimePicker Design Principles
</h2>

<p>A DateTimePicker is a UI component that lets users select a date, time, or both. But "picking a time" isn't as simple as it sounds.</p>

<h3>
  
  
  Principle 1: Know What You're Asking For
</h3>

<p>Before building (or choosing) a picker, decide:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>What does the user select?</th>
<th>What do you send to the backend?</th>
</tr>
</thead>
<tbody>
<tr>
<td>Just a date</td>
<td>
<code>PlainDate</code> → <code>"2026-06-05"</code>
</td>
</tr>
<tr>
<td>Just a time</td>
<td>
<code>PlainTime</code> → <code>"10:00"</code>
</td>
</tr>
<tr>
<td>Date and time</td>
<td>
<code>PlainDateTime</code> → <code>"2026-06-05T10:00"</code>
</td>
</tr>
<tr>
<td>Date, time, and timezone</td>
<td>
<code>ZonedDateTime</code> → <code>{ local, timeZoneId }</code>
</td>
</tr>
</tbody>
</table></div>

<p>Most pickers handle the first three. The fourth requires explicit timezone UI.</p>

<h3>
  
  
  Principle 2: Timezone Display — When and How
</h3>

<p><strong>Show the timezone when:</strong></p>

<ul>
<li>Users in different timezones use the system</li>
<li>The selected timezone might differ from the user's local timezone</li>
<li>The business operates across timezones</li>
</ul>

<p><strong>Hide the timezone when:</strong></p>

<ul>
<li>All users are in the same timezone</li>
<li>The context is unambiguous (e.g., "your local time")</li>
<li>Showing it would cause confusion without adding clarity</li>
</ul>

<p><strong>How to show it:</strong></p>

<ul>
<li>Display the current timezone near the picker: "Vienna (UTC+2)"</li>
<li>Allow changing it only if the user might need a different zone</li>
<li>Don't default to UTC — default to the user's timezone or the organization's timezone</li>
</ul>

<h3>
  
  
  Principle 3: Handle DST Gaps and Overlaps
</h3>

<p>When the user picks a time that falls in a DST transition:</p>

<p><strong>Gap (time doesn't exist):</strong></p>

<ul>
<li>Option A: Prevent selection (disable those times in the picker)</li>
<li>Option B: Accept and adjust (shift forward), but inform the user</li>
<li>Option C: Show a warning and ask the user to choose a different time</li>
</ul>

<p><strong>Overlap (time exists twice):</strong></p>

<ul>
<li>Option A: Ask which one they mean (before DST change or after)</li>
<li>Option B: Pick one automatically and note it</li>
<li>Option C: Ignore it (acceptable for many use cases)</li>
</ul>

<p>The right choice depends on your domain. A medical appointment might need explicit handling; a casual reminder might not.</p>

<h3>
  
  
  Principle 4: Don't Lie About What's Stored
</h3>

<p>If your backend stores <code>local + timeZoneId</code>, your picker should collect exactly that. Don't:</p>

<ul>
<li>Show a local picker but send UTC (user sees 10:00, backend gets 08:00)</li>
<li>Show UTC but let users think it's local</li>
<li>Convert silently and hope nobody notices</li>
</ul>

<p>The picker's display should match what gets stored.</p>

<h3>
  
  
  Principle 5: Consider the Editing Experience
</h3>

<p>When users edit an existing time:</p>

<ul>
<li>Show them what they originally entered (the local time)</li>
<li>Don't show a converted UTC value</li>
<li>If the timezone changed since creation, decide: show original zone or current zone?</li>
</ul>

<h3>
  
  
  Principle 6: Validation Belongs on Both Ends
</h3>

<p>The frontend picker should:</p>

<ul>
<li>Prevent obviously invalid dates (February 30th)</li>
<li>Warn about DST issues if relevant</li>
<li>Send well-formed data</li>
</ul>

<p>The backend should:</p>

<ul>
<li>Validate the timezone ID is real</li>
<li>Handle DST ambiguities according to business rules</li>
<li>Never trust that the frontend did it right</li>
</ul>




<h2>
  
  
  A Minimal Picker Contract
</h2>

<p>For a DateTimePicker that collects a scheduled time:</p>

<p><strong>Input (initial value):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kr">interface</span> <span class="nx">DateTimePickerValue</span> <span class="p">{</span>
  <span class="nl">local</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>       <span class="c1">// "2026-06-05T10:00"</span>
  <span class="nl">timeZoneId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>  <span class="c1">// "Europe/Vienna"</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Output (on change):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kr">interface</span> <span class="nx">DateTimePickerValue</span> <span class="p">{</span>
  <span class="nl">local</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>       <span class="c1">// "2026-06-05T14:30"</span>
  <span class="nl">timeZoneId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>  <span class="c1">// "Europe/Vienna"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The picker:</p>

<ul>
<li>Displays date and time inputs</li>
<li>Optionally displays or allows changing the timezone</li>
<li>Emits the combined value on change</li>
</ul>

<p>The parent component:</p>

<ul>
<li>Receives the structured value</li>
<li>Sends it to the backend as-is</li>
<li>Doesn't do timezone math</li>
</ul>




<h2>
  
  
  Putting It Together: Frontend to Database
</h2>

<p>Here's the full flow:</p>

<p><strong>1. User picks a time in a DateTimePicker</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>UI shows: June 5, 2026 at 10:00 AM (Vienna)
</code></pre>

</div>



<p><strong>2. Frontend sends to API</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">POST</span><span class="w"> </span><span class="err">/appointments</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Team Standup"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"local"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T10:00:00"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"timeZoneId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Europe/Vienna"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>3. Backend (NodaTime) processes</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">local</span> <span class="p">=</span> <span class="n">LocalDateTime</span><span class="p">.</span><span class="nf">FromDateTime</span><span class="p">(</span><span class="n">DateTime</span><span class="p">.</span><span class="nf">Parse</span><span class="p">(</span><span class="n">dto</span><span class="p">.</span><span class="n">Start</span><span class="p">.</span><span class="n">Local</span><span class="p">));</span>
<span class="kt">var</span> <span class="n">zone</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="n">dto</span><span class="p">.</span><span class="n">Start</span><span class="p">.</span><span class="n">TimeZoneId</span><span class="p">];</span>
<span class="kt">var</span> <span class="n">instant</span> <span class="p">=</span> <span class="n">local</span><span class="p">.</span><span class="nf">InZoneLeniently</span><span class="p">(</span><span class="n">zone</span><span class="p">).</span><span class="nf">ToInstant</span><span class="p">();</span>

<span class="c1">// Store all three</span>
<span class="kt">var</span> <span class="n">appointment</span> <span class="p">=</span> <span class="k">new</span> <span class="n">Appointment</span>
<span class="p">{</span>
    <span class="n">LocalStart</span> <span class="p">=</span> <span class="n">local</span><span class="p">,</span>
    <span class="n">TimeZoneId</span> <span class="p">=</span> <span class="n">dto</span><span class="p">.</span><span class="n">Start</span><span class="p">.</span><span class="n">TimeZoneId</span><span class="p">,</span>
    <span class="n">InstantUtc</span> <span class="p">=</span> <span class="n">instant</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>4. Database stores</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">appointments</span> <span class="p">(</span><span class="n">local_start</span><span class="p">,</span> <span class="n">time_zone_id</span><span class="p">,</span> <span class="n">instant_utc</span><span class="p">)</span>
<span class="k">VALUES</span> <span class="p">(</span><span class="s1">'2026-06-05 10:00:00'</span><span class="p">,</span> <span class="s1">'Europe/Vienna'</span><span class="p">,</span> <span class="s1">'2026-06-05 08:00:00+00'</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>5. Later: API returns to frontend</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Team Standup"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"local"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T10:00:00"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"timeZoneId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Europe/Vienna"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"instantUtc"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-06-05T08:00:00Z"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>6. Frontend displays</strong></p>

<ul>
<li>To the organizer (Vienna): "June 5 at 10:00 AM"</li>
<li>To a participant in London: "June 5 at 9:00 AM (10:00 AM Vienna)"</li>
</ul>




<h2>
  
  
  Detecting the User's Timezone
</h2>

<p>We've covered storing and displaying times with timezones. But how do you know what timezone the user is in?</p>

<h3>
  
  
  Browser Detection
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">userZone</span> <span class="o">=</span> <span class="nx">Intl</span><span class="p">.</span><span class="nc">DateTimeFormat</span><span class="p">().</span><span class="nf">resolvedOptions</span><span class="p">().</span><span class="nx">timeZone</span><span class="p">;</span>
<span class="c1">// "Europe/Vienna", "Europe/London", etc.</span>
</code></pre>

</div>



<p>This returns an IANA timezone ID — exactly what you need.</p>

<h3>
  
  
  Caveats
</h3>

<ul>
<li>
<strong>VPNs and proxies</strong> may cause the browser to report a different zone than the user expects</li>
<li>
<strong>Corporate networks</strong> sometimes override timezone settings</li>
<li>
<strong>User preference</strong> might differ from their physical location (e.g., someone living in Vienna but working with a London team)</li>
</ul>

<h3>
  
  
  Best Practice
</h3>

<p>Use browser detection as a <strong>default</strong>, but let users <strong>confirm or change</strong> it:</p>

<ol>
<li>Detect the timezone automatically</li>
<li>Show it clearly in the UI: "Your timezone: Europe/Vienna"</li>
<li>Let users change it if needed</li>
<li>Store their preference (per user, not per session)</li>
</ol>

<p>Don't silently assume the detected timezone is correct. A user in Vienna might be scheduling a meeting for their London office.</p>




<h2>
  
  
  Timezone Is Not a Locale (and Not a Language)
</h2>

<p>Timezone, language, and locale are often treated as one setting — but they are three independent concerns.</p>

<ul>
<li>
<p><strong>Language (i18n)</strong> controls text:</p>

<ul>
<li>"Today" vs "Heute" vs "Aujourd'hui"</li>
</ul>


</li>

<li>

<p><strong>Locale (l10n)</strong> controls formatting:</p>

<ul>
<li>
<code>1,000.00</code> vs <code>1.000,00</code>
</li>
<li>
<code>MM/DD/YYYY</code> vs <code>DD.MM.YYYY</code>
</li>
</ul>


</li>

<li>

<p><strong>Timezone</strong> controls <em>when things happen</em>:</p>

<ul>
<li><code>Europe/Vienna</code></li>
<li><code>America/New_York</code></li>
<li><code>Asia/Tokyo</code></li>
</ul>


</li>

</ul>

<p>They often change together — but they are <strong>not coupled</strong>.</p>

<h3>
  
  
  Example
</h3>

<p>A French-speaking user in New York expects French UI, French date formatting, and New York time. Inferring <code>Europe/Paris</code> from <code>fr</code> is wrong.</p>

<h3>
  
  
  DateTimePicker Rule
</h3>

<p>A DateTimePicker should not assume timezone based on language or locale.</p>

<p>Timezone must come from explicit user choice, browser detection, or application context.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>
<strong>Temporal API</strong> gives JavaScript proper types: <code>PlainDate</code>, <code>PlainDateTime</code>, <code>ZonedDateTime</code>, <code>Instant</code>
</li>
<li>
<strong>API contracts</strong> should be explicit: use ISO strings for instants, structured objects for user intent</li>
<li>
<strong>DateTimePickers</strong> need to know what they're collecting: date, time, datetime, or datetime + zone</li>
<li>
<strong>Show timezones</strong> when they matter, hide them when they'd confuse</li>
<li>
<strong>Handle DST</strong> explicitly — don't let invalid times slip through silently</li>
<li>
<strong>Don't lie</strong> about what's stored — the picker should match the backend model</li>
<li>
<strong>Validate on both ends</strong> — trust but verify</li>
</ul>




<p><em>This concludes the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">"Time in Software, Done Right."</a> You now have a complete mental model for handling time — from concepts to code, from backend to database to frontend.</em></p>

<p><em>The next time someone says "just store it as UTC," you'll know when that's right, and when it's a trap.</em></p>

