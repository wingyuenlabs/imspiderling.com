---
Title: .NET in Practice – Modeling Time with NodaTime
Description: 
Author: bwi
Date: 2026-01-22T22:05:54.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 6 of 8</strong> in the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">Time in Software, Done Right</a></p>
</blockquote>




<p>You've made it through the conceptual articles. You understand the difference between instants and local times, between global and local events, between storing intent and storing math.</p>

<p>Now let's make it real in .NET.</p>

<p>The BCL has improved significantly — <code>DateOnly</code> and <code>TimeOnly</code> (since .NET 6) are solid types for dates and times. But for <strong>timezone-aware scheduling</strong> — meetings, deadlines, appointments that need to survive DST changes — you'll want <strong>NodaTime</strong>. It gives you the types the BCL is still missing.</p>

<p>This article shows you how to use NodaTime to model time correctly, store it properly, and avoid the traps we've discussed throughout this series.</p>




<h2>
  
  
  Why DateTime Falls Short (and What the BCL Fixed)
</h2>

<p><code>DateTime</code> in .NET is a single type that tries to represent multiple concepts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">a</span> <span class="p">=</span> <span class="n">DateTime</span><span class="p">.</span><span class="n">Now</span><span class="p">;</span>           <span class="c1">// Local time on this machine</span>
<span class="kt">var</span> <span class="n">b</span> <span class="p">=</span> <span class="n">DateTime</span><span class="p">.</span><span class="n">UtcNow</span><span class="p">;</span>        <span class="c1">// UTC instant</span>
<span class="kt">var</span> <span class="n">c</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">DateTime</span><span class="p">(</span><span class="m">2026</span><span class="p">,</span> <span class="m">6</span><span class="p">,</span> <span class="m">5</span><span class="p">,</span> <span class="m">10</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>  <span class="c1">// Is this local? UTC? Unspecified?</span>
</code></pre>

</div>



<p>The problem is the <code>Kind</code> property:</p>

<ul>
<li>
<code>DateTimeKind.Local</code> — local to <em>this machine</em> (not a specific timezone)</li>
<li>
<code>DateTimeKind.Utc</code> — a UTC instant</li>
<li>
<code>DateTimeKind.Unspecified</code> — could be anything</li>
</ul>

<p>When you create <code>new DateTime(2026, 6, 5, 10, 0, 0)</code>, the <code>Kind</code> is <code>Unspecified</code>. Is that 10:00 in Vienna? 10:00 in London? 10:00 UTC? The type doesn't know, and neither does your code.</p>

<h3>
  
  
  The BCL Got Better: DateOnly and TimeOnly
</h3>

<p>.NET 6 added two types that address part of this problem:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">DateOnly</span> <span class="n">birthday</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">DateOnly</span><span class="p">(</span><span class="m">1990</span><span class="p">,</span> <span class="m">3</span><span class="p">,</span> <span class="m">15</span><span class="p">);</span>    <span class="c1">// Just a date, no time confusion</span>
<span class="n">TimeOnly</span> <span class="n">openingTime</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">TimeOnly</span><span class="p">(</span><span class="m">9</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>         <span class="c1">// Just a time, no date confusion</span>
</code></pre>

</div>



<p>These are great! If you just need a date or just a time, use them. They're in the BCL, well-supported by EF Core, and do exactly what they say.</p>

<h3>
  
  
  What the BCL Still Doesn't Have
</h3>

<p>But for the full picture — especially <strong>timezone-aware scheduling</strong> — the BCL still falls short:</p>

<ul>
<li>No <code>Instant</code> type (you use <code>DateTime</code> with <code>Kind.Utc</code> or <code>DateTimeOffset</code>)</li>
<li>No <code>LocalDateTime</code> with proper semantics (you use <code>DateTime</code> with <code>Kind.Unspecified</code>)</li>
<li>No <code>ZonedDateTime</code> that combines local time with a timezone</li>
<li>No first-class IANA timezone support (<code>TimeZoneInfo</code> uses Windows zones by default)</li>
</ul>

<p><code>DateTimeOffset</code> is better than <code>DateTime</code> — it includes an offset — but as we discussed in Article 4, an offset is a snapshot, not a meaning. <code>+02:00</code> could be Vienna in summer, Berlin in summer, Cairo, or Johannesburg. You can't tell.</p>

<p><strong>For simple cases</strong>: <code>DateOnly</code>, <code>TimeOnly</code>, <code>DateTime</code>, and <code>DateTimeOffset</code> are fine.</p>

<p><strong>For timezone-aware scheduling</strong>: NodaTime gives you the right types for the right concepts.</p>




<h2>
  
  
  The NodaTime Types You Need
</h2>

<p>Here's how NodaTime maps to the concepts we've covered (and their BCL equivalents where they exist):</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concept</th>
<th>NodaTime Type</th>
<th>BCL Equivalent</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td>Physical moment</td>
<td><code>Instant</code></td>
<td>
<code>DateTime</code> (UTC) / <code>DateTimeOffset</code>
</td>
<td>Log timestamp, token expiry</td>
</tr>
<tr>
<td>Calendar date</td>
<td><code>LocalDate</code></td>
<td>
<code>DateOnly</code> ✓</td>
<td>Birthday, holiday</td>
</tr>
<tr>
<td>Wall clock time</td>
<td><code>LocalTime</code></td>
<td>
<code>TimeOnly</code> ✓</td>
<td>"Opens at 09:00"</td>
</tr>
<tr>
<td>Date + time (no zone)</td>
<td><code>LocalDateTime</code></td>
<td>
<code>DateTime</code> (Unspecified)</td>
<td>User's chosen meeting time</td>
</tr>
<tr>
<td>IANA timezone</td>
<td><code>DateTimeZone</code></td>
<td>
<code>TimeZoneInfo</code> (partial)</td>
<td><code>Europe/Vienna</code></td>
</tr>
<tr>
<td>Full context</td>
<td><code>ZonedDateTime</code></td>
<td>❌ None</td>
<td>Meeting at 10:00 Vienna</td>
</tr>
<tr>
<td>Snapshot with offset</td>
<td><code>OffsetDateTime</code></td>
<td><code>DateTimeOffset</code></td>
<td>What the clock showed at a moment</td>
</tr>
</tbody>
</table></div>

<p>The ✓ marks where the BCL type is a good choice. For <code>DateOnly</code> and <code>TimeOnly</code>, you can often skip NodaTime entirely.</p>

<p>The gap is <code>ZonedDateTime</code> — the combination of a local time and an IANA timezone that lets you handle DST correctly. That's where NodaTime shines.</p>

<p>Let's see each in action.</p>




<h2>
  
  
  Instant: For Physical Moments
</h2>

<p>Use <code>Instant</code> when you're recording <em>when something happened</em> — independent of any human's calendar.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Current moment</span>
<span class="n">Instant</span> <span class="n">now</span> <span class="p">=</span> <span class="n">SystemClock</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="nf">GetCurrentInstant</span><span class="p">();</span>

<span class="c1">// From a Unix timestamp</span>
<span class="n">Instant</span> <span class="n">fromUnix</span> <span class="p">=</span> <span class="n">Instant</span><span class="p">.</span><span class="nf">FromUnixTimeSeconds</span><span class="p">(</span><span class="m">1735689600</span><span class="p">);</span>

<span class="c1">// For logs, audits, event sourcing</span>
<span class="k">public</span> <span class="k">class</span> <span class="nc">AuditEntry</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="n">Instant</span> <span class="n">OccurredAt</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Action</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><code>Instant</code> is unambiguous. There's no timezone to confuse, no <code>Kind</code> property to check. It's just a point on the timeline.</p>




<h2>
  
  
  LocalDate, LocalTime, LocalDateTime: For Human Concepts
</h2>

<p>These types represent calendar and clock values <strong>without</strong> a timezone attached.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Just a date (NodaTime)</span>
<span class="n">LocalDate</span> <span class="n">birthday</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">LocalDate</span><span class="p">(</span><span class="m">1990</span><span class="p">,</span> <span class="m">3</span><span class="p">,</span> <span class="m">15</span><span class="p">);</span>

<span class="c1">// Just a date (BCL - equally good!)</span>
<span class="n">DateOnly</span> <span class="n">birthdayBcl</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">DateOnly</span><span class="p">(</span><span class="m">1990</span><span class="p">,</span> <span class="m">3</span><span class="p">,</span> <span class="m">15</span><span class="p">);</span>

<span class="c1">// Just a time (NodaTime)</span>
<span class="n">LocalTime</span> <span class="n">openingTime</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">LocalTime</span><span class="p">(</span><span class="m">9</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>

<span class="c1">// Just a time (BCL - equally good!)</span>
<span class="n">TimeOnly</span> <span class="n">openingTimeBcl</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">TimeOnly</span><span class="p">(</span><span class="m">9</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>

<span class="c1">// Date and time together (NodaTime)</span>
<span class="n">LocalDateTime</span> <span class="n">meetingTime</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">LocalDateTime</span><span class="p">(</span><span class="m">2026</span><span class="p">,</span> <span class="m">6</span><span class="p">,</span> <span class="m">5</span><span class="p">,</span> <span class="m">10</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>For dates and times alone</strong>, use whichever you prefer — <code>DateOnly</code>/<code>TimeOnly</code> are in the BCL and work great with EF Core.</p>

<p><strong>For date+time combinations</strong> that you'll later combine with a timezone, NodaTime's <code>LocalDateTime</code> is clearer because it's part of a coherent type system that includes <code>ZonedDateTime</code>.</p>

<p>A <code>LocalDateTime</code> of <code>2026-06-05T10:00</code> means "June 5th at 10:00" — but it doesn't yet specify <em>where</em>. That's intentional. You'll combine it with a timezone to get the full picture.</p>




<h2>
  
  
  DateTimeZone: The Ruleset
</h2>

<p>A <code>DateTimeZone</code> represents an IANA timezone — not just an offset, but the complete ruleset including DST transitions and historical changes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Get a timezone by IANA ID</span>
<span class="n">DateTimeZone</span> <span class="n">vienna</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="s">"Europe/Vienna"</span><span class="p">];</span>
<span class="n">DateTimeZone</span> <span class="n">london</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="s">"Europe/London"</span><span class="p">];</span>

<span class="c1">// The provider gives you access to all IANA zones</span>
<span class="n">IDateTimeZoneProvider</span> <span class="n">tzdb</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">;</span>
</code></pre>

</div>



<p><code>DateTimeZoneProviders.Tzdb</code> uses the IANA tz database, which is updated regularly with new rules. When you update NodaTime's tzdb data, your code automatically handles new DST rules.</p>




<h2>
  
  
  ZonedDateTime: The Full Picture
</h2>

<p><code>ZonedDateTime</code> combines a <code>LocalDateTime</code> with a <code>DateTimeZone</code> — giving you everything you need.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">LocalDateTime</span> <span class="n">local</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">LocalDateTime</span><span class="p">(</span><span class="m">2026</span><span class="p">,</span> <span class="m">6</span><span class="p">,</span> <span class="m">5</span><span class="p">,</span> <span class="m">10</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>
<span class="n">DateTimeZone</span> <span class="n">zone</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="s">"Europe/Vienna"</span><span class="p">];</span>

<span class="c1">// Combine them</span>
<span class="n">ZonedDateTime</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">local</span><span class="p">.</span><span class="nf">InZoneLeniently</span><span class="p">(</span><span class="n">zone</span><span class="p">);</span>

<span class="c1">// Now you can get the instant</span>
<span class="n">Instant</span> <span class="n">instant</span> <span class="p">=</span> <span class="n">zoned</span><span class="p">.</span><span class="nf">ToInstant</span><span class="p">();</span>

<span class="c1">// Or display in different zones</span>
<span class="n">ZonedDateTime</span> <span class="n">inLondon</span> <span class="p">=</span> <span class="n">instant</span><span class="p">.</span><span class="nf">InZone</span><span class="p">(</span><span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="s">"Europe/London"</span><span class="p">]);</span>
<span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">inNewYork</span><span class="p">.</span><span class="nf">ToString</span><span class="p">(</span><span class="s">"uuuu-MM-dd HH:mm x"</span><span class="p">,</span> <span class="n">CultureInfo</span><span class="p">.</span><span class="n">InvariantCulture</span><span class="p">));</span>
<span class="c1">// Output: 2026-06-05 04:00 -04</span>
<span class="c1">// (Requires: using System.Globalization;)</span>
</code></pre>

</div>



<h3>
  
  
  Why "Leniently"?
</h3>

<p>The <code>InZoneLeniently</code> method handles DST edge cases automatically:</p>

<ul>
<li>If the local time falls in a <strong>gap</strong> (doesn't exist), it shifts forward</li>
<li>If the local time falls in an <strong>overlap</strong> (exists twice), it picks the earlier occurrence</li>
</ul>

<p>For explicit control, NodaTime offers several options:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Called on LocalDateTime</span>
<span class="n">ZonedDateTime</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">local</span><span class="p">.</span><span class="nf">InZoneLeniently</span><span class="p">(</span><span class="n">zone</span><span class="p">);</span>   <span class="c1">// Auto-resolve gaps/overlaps</span>
<span class="n">ZonedDateTime</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">local</span><span class="p">.</span><span class="nf">InZoneStrictly</span><span class="p">(</span><span class="n">zone</span><span class="p">);</span>    <span class="c1">// Throws if ambiguous</span>

<span class="c1">// Called on DateTimeZone (same behavior, different syntax)</span>
<span class="n">ZonedDateTime</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">zone</span><span class="p">.</span><span class="nf">AtLeniently</span><span class="p">(</span><span class="n">local</span><span class="p">);</span>
<span class="n">ZonedDateTime</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">zone</span><span class="p">.</span><span class="nf">AtStrictly</span><span class="p">(</span><span class="n">local</span><span class="p">);</span>

<span class="c1">// With custom resolver</span>
<span class="n">ZonedDateTime</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">local</span><span class="p">.</span><span class="nf">InZone</span><span class="p">(</span><span class="n">zone</span><span class="p">,</span> <span class="n">Resolvers</span><span class="p">.</span><span class="n">LenientResolver</span><span class="p">);</span>
</code></pre>

</div>






<h2>
  
  
  The Pattern: Store Intent, Derive Instant
</h2>

<p>Here's the core pattern from Article 4, implemented in NodaTime:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Appointment</span>
<span class="p">{</span>
    <span class="c1">// Source of truth: what the user chose</span>
    <span class="k">public</span> <span class="n">LocalDateTime</span> <span class="n">LocalStart</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">TimeZoneId</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="c1">// Derived: for queries and sorting</span>
    <span class="k">public</span> <span class="n">Instant</span> <span class="n">InstantUtc</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">private</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="nf">RecalculateInstant</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">zone</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="n">TimeZoneId</span><span class="p">];</span>
        <span class="kt">var</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">LocalStart</span><span class="p">.</span><span class="nf">InZoneLeniently</span><span class="p">(</span><span class="n">zone</span><span class="p">);</span>
        <span class="n">InstantUtc</span> <span class="p">=</span> <span class="n">zoned</span><span class="p">.</span><span class="nf">ToInstant</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>When timezone rules change, you call <code>RecalculateInstant()</code> on future appointments. Past appointments stay correct because IANA contains historical rules.</p>




<h2>
  
  
  Real-World Examples
</h2>

<h3>
  
  
  Example 1: Logging (Use Instant)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">LogEntry</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="n">Instant</span> <span class="n">Timestamp</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Level</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Message</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="k">static</span> <span class="n">LogEntry</span> <span class="nf">Create</span><span class="p">(</span><span class="kt">string</span> <span class="n">level</span><span class="p">,</span> <span class="kt">string</span> <span class="n">message</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="n">LogEntry</span>
        <span class="p">{</span>
            <span class="n">Timestamp</span> <span class="p">=</span> <span class="n">SystemClock</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="nf">GetCurrentInstant</span><span class="p">(),</span>
            <span class="n">Level</span> <span class="p">=</span> <span class="n">level</span><span class="p">,</span>
            <span class="n">Message</span> <span class="p">=</span> <span class="n">message</span>
        <span class="p">};</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Example 2: Birthday (Use LocalDate)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Person</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Name</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">LocalDate</span> <span class="n">DateOfBirth</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kt">int</span> <span class="nf">GetAge</span><span class="p">(</span><span class="n">LocalDate</span> <span class="n">today</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="n">Period</span><span class="p">.</span><span class="nf">Between</span><span class="p">(</span><span class="n">DateOfBirth</span><span class="p">,</span> <span class="n">today</span><span class="p">,</span> <span class="n">PeriodUnits</span><span class="p">.</span><span class="n">Years</span><span class="p">).</span><span class="n">Years</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>No timezone needed — birthdays are calendar concepts.</p>

<h3>
  
  
  Example 3: Meeting (Use LocalDateTime + TimeZone)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Meeting</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Title</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">LocalDateTime</span> <span class="n">LocalStart</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">TimeZoneId</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">Instant</span> <span class="n">InstantUtc</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="k">static</span> <span class="n">Meeting</span> <span class="nf">Create</span><span class="p">(</span><span class="kt">string</span> <span class="n">title</span><span class="p">,</span> <span class="n">LocalDateTime</span> <span class="n">localStart</span><span class="p">,</span> <span class="kt">string</span> <span class="n">timeZoneId</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">zone</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="n">timeZoneId</span><span class="p">];</span>
        <span class="kt">var</span> <span class="n">instant</span> <span class="p">=</span> <span class="n">localStart</span><span class="p">.</span><span class="nf">InZoneLeniently</span><span class="p">(</span><span class="n">zone</span><span class="p">).</span><span class="nf">ToInstant</span><span class="p">();</span>

        <span class="k">return</span> <span class="k">new</span> <span class="n">Meeting</span>
        <span class="p">{</span>
            <span class="n">Title</span> <span class="p">=</span> <span class="n">title</span><span class="p">,</span>
            <span class="n">LocalStart</span> <span class="p">=</span> <span class="n">localStart</span><span class="p">,</span>
            <span class="n">TimeZoneId</span> <span class="p">=</span> <span class="n">timeZoneId</span><span class="p">,</span>
            <span class="n">InstantUtc</span> <span class="p">=</span> <span class="n">instant</span>
        <span class="p">};</span>
    <span class="p">}</span>

    <span class="c1">// Display in any timezone</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="nf">GetDisplayTime</span><span class="p">(</span><span class="n">DateTimeZone</span> <span class="n">viewerZone</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">inViewerZone</span> <span class="p">=</span> <span class="n">InstantUtc</span><span class="p">.</span><span class="nf">InZone</span><span class="p">(</span><span class="n">viewerZone</span><span class="p">);</span>
        <span class="c1">// Note: uuuu is NodaTime's recommended year specifier (absolute year)</span>
        <span class="k">return</span> <span class="n">inViewerZone</span><span class="p">.</span><span class="nf">ToString</span><span class="p">(</span><span class="s">"uuuu-MM-dd HH:mm"</span><span class="p">,</span> <span class="n">CultureInfo</span><span class="p">.</span><span class="n">InvariantCulture</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Example 4: Deadline (Use LocalDateTime + TimeZone)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Deadline</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="n">LocalDateTime</span> <span class="n">LocalDeadline</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">TimeZoneId</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="n">Instant</span> <span class="n">InstantUtc</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="kt">bool</span> <span class="nf">IsPastDeadline</span><span class="p">(</span><span class="n">Instant</span> <span class="n">now</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="n">now</span> <span class="p">&gt;</span> <span class="n">InstantUtc</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="n">Duration</span> <span class="nf">TimeRemaining</span><span class="p">(</span><span class="n">Instant</span> <span class="n">now</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="n">InstantUtc</span> <span class="p">-</span> <span class="n">now</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  EF Core Integration
</h2>

<p>NodaTime doesn't map to SQL types out of the box, but there are excellent packages for this.</p>

<h3>
  
  
  For PostgreSQL: Npgsql.EntityFrameworkCore.PostgreSQL.NodaTime
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// In your DbContext configuration</span>
<span class="k">protected</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">OnConfiguring</span><span class="p">(</span><span class="n">DbContextOptionsBuilder</span> <span class="n">optionsBuilder</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">optionsBuilder</span><span class="p">.</span><span class="nf">UseNpgsql</span><span class="p">(</span><span class="n">connectionString</span><span class="p">,</span> <span class="n">o</span> <span class="p">=&gt;</span> <span class="n">o</span><span class="p">.</span><span class="nf">UseNodaTime</span><span class="p">());</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This maps:</p>

<ul>
<li>
<code>Instant</code> → <code>timestamp with time zone</code>
</li>
<li>
<code>LocalDateTime</code> → <code>timestamp without time zone</code>
</li>
<li>
<code>LocalDate</code> → <code>date</code>
</li>
<li>
<code>LocalTime</code> → <code>time</code>
</li>
</ul>

<p><strong>What about <code>ZonedDateTime</code>?</strong> There's no single PostgreSQL type for it — that's the whole point of our pattern. You decompose it into separate columns:</p>

<ul>
<li>
<code>LocalDateTime</code> → <code>timestamp without time zone</code>
</li>
<li>
<code>TimeZoneId</code> → <code>text</code>
</li>
<li>Optionally: <code>Instant</code> → <code>timestamp with time zone</code> (for queries)</li>
</ul>

<p>Here's how to extract the parts from a <code>ZonedDateTime</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">ZonedDateTime</span> <span class="n">zoned</span> <span class="p">=</span> <span class="n">local</span><span class="p">.</span><span class="nf">InZoneLeniently</span><span class="p">(</span><span class="n">zone</span><span class="p">);</span>

<span class="c1">// Decompose for storage</span>
<span class="n">LocalDateTime</span> <span class="n">localPart</span> <span class="p">=</span> <span class="n">zoned</span><span class="p">.</span><span class="n">LocalDateTime</span><span class="p">;</span>
<span class="kt">string</span> <span class="n">timeZoneId</span> <span class="p">=</span> <span class="n">zoned</span><span class="p">.</span><span class="n">Zone</span><span class="p">.</span><span class="n">Id</span><span class="p">;</span>           <span class="c1">// e.g. "Europe/Vienna"</span>
<span class="n">Instant</span> <span class="n">instantPart</span> <span class="p">=</span> <span class="n">zoned</span><span class="p">.</span><span class="nf">ToInstant</span><span class="p">();</span>
</code></pre>

</div>



<h3>
  
  
  For SQL Server: Consider Value Converters
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">AppointmentConfiguration</span> <span class="p">:</span> <span class="n">IEntityTypeConfiguration</span><span class="p">&lt;</span><span class="n">Appointment</span><span class="p">&gt;</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">void</span> <span class="nf">Configure</span><span class="p">(</span><span class="n">EntityTypeBuilder</span><span class="p">&lt;</span><span class="n">Appointment</span><span class="p">&gt;</span> <span class="n">builder</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">builder</span><span class="p">.</span><span class="nf">Property</span><span class="p">(</span><span class="n">a</span> <span class="p">=&gt;</span> <span class="n">a</span><span class="p">.</span><span class="n">LocalStart</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">HasConversion</span><span class="p">(</span>
                <span class="n">v</span> <span class="p">=&gt;</span> <span class="n">v</span><span class="p">.</span><span class="nf">ToDateTimeUnspecified</span><span class="p">(),</span>
                <span class="n">v</span> <span class="p">=&gt;</span> <span class="n">LocalDateTime</span><span class="p">.</span><span class="nf">FromDateTime</span><span class="p">(</span><span class="n">v</span><span class="p">));</span>

        <span class="n">builder</span><span class="p">.</span><span class="nf">Property</span><span class="p">(</span><span class="n">a</span> <span class="p">=&gt;</span> <span class="n">a</span><span class="p">.</span><span class="n">InstantUtc</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">HasConversion</span><span class="p">(</span>
                <span class="n">v</span> <span class="p">=&gt;</span> <span class="n">v</span><span class="p">.</span><span class="nf">ToDateTimeUtc</span><span class="p">(),</span>
                <span class="n">v</span> <span class="p">=&gt;</span> <span class="n">Instant</span><span class="p">.</span><span class="nf">FromDateTimeUtc</span><span class="p">(</span><span class="n">v</span><span class="p">));</span>

        <span class="n">builder</span><span class="p">.</span><span class="nf">Property</span><span class="p">(</span><span class="n">a</span> <span class="p">=&gt;</span> <span class="n">a</span><span class="p">.</span><span class="n">TimeZoneId</span><span class="p">)</span>
            <span class="p">.</span><span class="nf">HasMaxLength</span><span class="p">(</span><span class="m">64</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Sample Entity
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Appointment</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="n">Guid</span> <span class="n">Id</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Title</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="c1">// Stored as timestamp without time zone</span>
    <span class="k">public</span> <span class="n">LocalDateTime</span> <span class="n">LocalStart</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="c1">// Stored as text/varchar</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">TimeZoneId</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>

    <span class="c1">// Stored as timestamp with time zone (for queries)</span>
    <span class="k">public</span> <span class="n">Instant</span> <span class="n">InstantUtc</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">init</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Handling DST Transitions
</h2>

<p>When creating appointments that might fall in DST gaps or overlaps, be explicit:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">AppointmentService</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="n">ZonedDateTime</span> <span class="nf">ResolveLocalTime</span><span class="p">(</span><span class="n">LocalDateTime</span> <span class="n">local</span><span class="p">,</span> <span class="kt">string</span> <span class="n">timeZoneId</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">zone</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="n">timeZoneId</span><span class="p">];</span>
        <span class="kt">var</span> <span class="n">mapping</span> <span class="p">=</span> <span class="n">zone</span><span class="p">.</span><span class="nf">MapLocal</span><span class="p">(</span><span class="n">local</span><span class="p">);</span>

        <span class="k">return</span> <span class="n">mapping</span><span class="p">.</span><span class="n">Count</span> <span class="k">switch</span>
        <span class="p">{</span>
            <span class="m">0</span> <span class="p">=&gt;</span> <span class="n">zone</span><span class="p">.</span><span class="nf">AtLeniently</span><span class="p">(</span><span class="n">local</span><span class="p">),</span>        <span class="c1">// Gap: shift forward to valid time</span>
            <span class="m">1</span> <span class="p">=&gt;</span> <span class="n">mapping</span><span class="p">.</span><span class="nf">Single</span><span class="p">(),</span>                <span class="c1">// Normal: exactly one mapping</span>
            <span class="m">2</span> <span class="p">=&gt;</span> <span class="n">mapping</span><span class="p">.</span><span class="nf">First</span><span class="p">(),</span>                 <span class="c1">// Overlap: pick earlier occurrence</span>
            <span class="n">_</span> <span class="p">=&gt;</span> <span class="k">throw</span> <span class="k">new</span> <span class="nf">InvalidOperationException</span><span class="p">()</span>
        <span class="p">};</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>For more control (e.g., asking the user to choose during overlaps):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="n">ZonedDateTime</span> <span class="nf">ResolveWithUserChoice</span><span class="p">(</span>
    <span class="n">LocalDateTime</span> <span class="n">local</span><span class="p">,</span> 
    <span class="kt">string</span> <span class="n">timeZoneId</span><span class="p">,</span>
    <span class="n">Func</span><span class="p">&lt;</span><span class="n">ZonedDateTime</span><span class="p">,</span> <span class="n">ZonedDateTime</span><span class="p">,</span> <span class="n">ZonedDateTime</span><span class="p">&gt;</span> <span class="n">overlapResolver</span><span class="p">)</span>
<span class="p">{</span>
    <span class="kt">var</span> <span class="n">zone</span> <span class="p">=</span> <span class="n">DateTimeZoneProviders</span><span class="p">.</span><span class="n">Tzdb</span><span class="p">[</span><span class="n">timeZoneId</span><span class="p">];</span>
    <span class="kt">var</span> <span class="n">mapping</span> <span class="p">=</span> <span class="n">zone</span><span class="p">.</span><span class="nf">MapLocal</span><span class="p">(</span><span class="n">local</span><span class="p">);</span>

    <span class="k">return</span> <span class="n">mapping</span><span class="p">.</span><span class="n">Count</span> <span class="k">switch</span>
    <span class="p">{</span>
        <span class="m">0</span> <span class="p">=&gt;</span> <span class="n">zone</span><span class="p">.</span><span class="nf">AtLeniently</span><span class="p">(</span><span class="n">local</span><span class="p">),</span>
        <span class="m">1</span> <span class="p">=&gt;</span> <span class="n">mapping</span><span class="p">.</span><span class="nf">Single</span><span class="p">(),</span>
        <span class="m">2</span> <span class="p">=&gt;</span> <span class="nf">overlapResolver</span><span class="p">(</span><span class="n">mapping</span><span class="p">.</span><span class="nf">First</span><span class="p">(),</span> <span class="n">mapping</span><span class="p">.</span><span class="nf">Last</span><span class="p">()),</span>
        <span class="n">_</span> <span class="p">=&gt;</span> <span class="k">throw</span> <span class="k">new</span> <span class="nf">InvalidOperationException</span><span class="p">()</span>
    <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Converting from DateTime
</h2>

<p>If you have existing code using <code>DateTime</code>, here's how to convert:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// DateTime (UTC) to Instant</span>
<span class="n">DateTime</span> <span class="n">dtUtc</span> <span class="p">=</span> <span class="n">DateTime</span><span class="p">.</span><span class="n">UtcNow</span><span class="p">;</span>
<span class="n">Instant</span> <span class="n">instant</span> <span class="p">=</span> <span class="n">Instant</span><span class="p">.</span><span class="nf">FromDateTimeUtc</span><span class="p">(</span><span class="n">dtUtc</span><span class="p">);</span>

<span class="c1">// DateTime (unspecified) to LocalDateTime</span>
<span class="n">DateTime</span> <span class="n">dt</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">DateTime</span><span class="p">(</span><span class="m">2026</span><span class="p">,</span> <span class="m">6</span><span class="p">,</span> <span class="m">5</span><span class="p">,</span> <span class="m">10</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>
<span class="n">LocalDateTime</span> <span class="n">local</span> <span class="p">=</span> <span class="n">LocalDateTime</span><span class="p">.</span><span class="nf">FromDateTime</span><span class="p">(</span><span class="n">dt</span><span class="p">);</span>

<span class="c1">// Instant to DateTime (UTC)</span>
<span class="n">DateTime</span> <span class="n">backToUtc</span> <span class="p">=</span> <span class="n">instant</span><span class="p">.</span><span class="nf">ToDateTimeUtc</span><span class="p">();</span>

<span class="c1">// LocalDateTime to DateTime (unspecified)</span>
<span class="n">DateTime</span> <span class="n">backToUnspecified</span> <span class="p">=</span> <span class="n">local</span><span class="p">.</span><span class="nf">ToDateTimeUnspecified</span><span class="p">();</span>
</code></pre>

</div>






<h2>
  
  
  Testing Time-Dependent Code
</h2>

<p>Code that calls <code>SystemClock.Instance.GetCurrentInstant()</code> directly is hard to test. You can't control "now".</p>

<p>NodaTime solves this with <code>IClock</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// Production: inject the real clock</span>
<span class="k">public</span> <span class="k">class</span> <span class="nc">AppointmentService</span><span class="p">(</span><span class="n">IClock</span> <span class="n">clock</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">bool</span> <span class="nf">IsUpcoming</span><span class="p">(</span><span class="n">Appointment</span> <span class="n">appointment</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">now</span> <span class="p">=</span> <span class="n">clock</span><span class="p">.</span><span class="nf">GetCurrentInstant</span><span class="p">();</span>
        <span class="k">return</span> <span class="n">appointment</span><span class="p">.</span><span class="n">InstantUtc</span> <span class="p">&gt;</span> <span class="n">now</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// In production</span>
<span class="kt">var</span> <span class="n">service</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">AppointmentService</span><span class="p">(</span><span class="n">SystemClock</span><span class="p">.</span><span class="n">Instance</span><span class="p">);</span>

<span class="c1">// In tests: use a fake clock</span>
<span class="kt">var</span> <span class="n">fakeNow</span> <span class="p">=</span> <span class="n">Instant</span><span class="p">.</span><span class="nf">FromUtc</span><span class="p">(</span><span class="m">2026</span><span class="p">,</span> <span class="m">6</span><span class="p">,</span> <span class="m">5</span><span class="p">,</span> <span class="m">8</span><span class="p">,</span> <span class="m">0</span><span class="p">);</span>
<span class="kt">var</span> <span class="n">fakeClock</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">FakeClock</span><span class="p">(</span><span class="n">fakeNow</span><span class="p">);</span>
<span class="kt">var</span> <span class="n">service</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">AppointmentService</span><span class="p">(</span><span class="n">fakeClock</span><span class="p">);</span>

<span class="c1">// Now you can test time-dependent logic deterministically</span>
</code></pre>

</div>



<p><strong>Rule:</strong> Never call <code>SystemClock.Instance</code> directly in business logic. Inject <code>IClock</code> instead. Your tests will thank you.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>
<strong>Use NodaTime</strong> for anything beyond simple logging — it gives you the right types for the right concepts</li>
<li>
<strong><code>Instant</code></strong> for physical moments (logs, events, tokens)</li>
<li>
<strong><code>LocalDate</code></strong> for calendar dates (birthdays, holidays)</li>
<li>
<strong><code>LocalDateTime</code> + <code>DateTimeZone</code></strong> for human-scheduled times (meetings, deadlines)</li>
<li>
<strong>Store intent</strong>: <code>LocalDateTime</code> + <code>TimeZoneId</code> as your source of truth</li>
<li>
<strong>Derive instant</strong>: compute <code>InstantUtc</code> for queries and sorting</li>
<li>
<strong>Handle DST explicitly</strong>: use <code>InZoneLeniently</code> or check <code>MapLocal</code> for edge cases</li>
<li>
<strong>EF Core</strong>: use <code>Npgsql.EntityFrameworkCore.PostgreSQL.NodaTime</code> for PostgreSQL, or value converters for other databases</li>
</ul>




<p><em>Next up: <a href="https://dev.to/bwi/postgresql-storing-time-without-lying-to-yourself-jb1"><strong>PostgreSQL – Storing Time Without Lying to Yourself</strong></a> — the database side of the equation.</em></p>

