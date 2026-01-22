---
Title: Instant vs Local – When UTC Helps and When It Hurts
Description: 
Author: bwi
Date: 2026-01-22T22:01:57.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 4 of 8</strong> in the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">Time in Software, Done Right</a></p>
</blockquote>




<p>"Just store everything in UTC."</p>

<p>You've probably heard this advice. It sounds simple, clean, universal. And for some use cases, it's exactly right.</p>

<p>But for others, it's a trap that will corrupt your data in ways you won't notice until it's too late.</p>

<p>This article explains when UTC is the right choice, when it destroys meaning, and what model actually works for human-facing times.</p>




<h2>
  
  
  Two Fundamentally Different Questions
</h2>

<p>When you're storing a time, you need to ask yourself which question you're answering:</p>

<p><strong>Question A:</strong> "What physical moment did this happen?"</p>

<p><strong>Question B:</strong> "What did the human mean when they said '10:00 in Vienna'?"</p>

<p>These are not the same question. They need different storage strategies.</p>




<h2>
  
  
  When UTC Is Perfect
</h2>

<p>UTC (or Instant) is the right choice when you're recording <strong>physical moments</strong> — things that happened at a specific point in time, independent of any human's calendar.</p>

<p><strong>Use UTC for:</strong></p>

<ul>
<li>
<strong>Log entries</strong> — "Request received at 2026-01-21T13:05:12Z"</li>
<li>
<strong>Audit trails</strong> — "User clicked 'Submit' at this instant"</li>
<li>
<strong>Token expiry</strong> — "This token is valid for 3600 seconds from now"</li>
<li>
<strong>Event sourcing</strong> — "This event occurred at this physical moment"</li>
<li>
<strong>Timestamps for ordering</strong> — "Did A happen before or after B?"</li>
<li>
<strong>Durations and intervals</strong> — "How long did this take?"</li>
</ul>

<p>In all these cases, you don't care about calendars, timezones, or what the clock on someone's wall showed. You care about <strong>the moment itself</strong>.</p>

<p>UTC is perfect for this. It's unambiguous, comparable, and never changes.</p>




<h2>
  
  
  When UTC Destroys Meaning
</h2>

<p>UTC is the <strong>wrong</strong> choice when you're storing <strong>human intent</strong> — times that exist because a person chose them, in a specific calendar context.</p>

<p><strong>Don't use UTC alone for:</strong></p>

<ul>
<li>
<strong>Meetings</strong> — "Team standup at 10:00 Vienna time"</li>
<li>
<strong>Appointments</strong> — "Doctor's appointment at 14:30"</li>
<li>
<strong>Deadlines</strong> — "Submit by June 5th, 23:59 Vienna"</li>
<li>
<strong>Opening hours</strong> — "Store opens at 09:00"</li>
<li>
<strong>Recurring events</strong> — "Every Monday at 10:00"</li>
<li>
<strong>Birthdays and anniversaries</strong> — "Party on March 15th at 19:00"</li>
</ul>

<p>Why? Because these times have <strong>meaning beyond the instant</strong>.</p>

<p>This isn't obvious at first — many systems start this way and run into problems later.</p>




<h2>
  
  
  The Problem with "Just Convert to UTC"
</h2>

<p>Let's say a user in Vienna schedules a meeting for June 5th at 10:00.</p>

<p>If you convert immediately to UTC and store only that:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>stored: 2026-06-05T08:00:00Z
</code></pre>

</div>



<p>You've lost information. You no longer know:</p>

<ul>
<li>That the user meant "10:00 in Vienna"</li>
<li>That Vienna was in CEST (Central European Summer Time) at that moment</li>
<li>What should happen if timezone rules change</li>
</ul>

<p>Now imagine: the EU abolishes Daylight Saving Time in 2027. Vienna stays on permanent standard time (UTC+1 instead of UTC+2 in summer).</p>

<p>Your stored UTC value <code>08:00Z</code> now displays as <strong>09:00 Vienna time</strong> — not 10:00!</p>

<p>The user said "10:00". The meeting should still be at 10:00. But you stored the math, not the meaning.</p>




<h2>
  
  
  Why Offsets Don't Help
</h2>

<p>"But I stored <code>2026-06-05T10:00:00+02:00</code> — that includes the offset!"</p>

<p>Yes, but an offset is a <strong>snapshot</strong>, not a <strong>meaning</strong>.</p>

<p><code>+02:00</code> could be:</p>

<ul>
<li>Vienna in summer</li>
<li>Berlin in summer</li>
<li>Cairo</li>
<li>Johannesburg</li>
<li>Kaliningrad</li>
</ul>

<p>You can't tell which one. And more importantly: if the rules change, you can't recalculate.</p>

<p>An offset tells you what the clock showed at the moment of storage. It doesn't tell you what the user <strong>meant</strong>, or what the correct time should be in the future.</p>




<h2>
  
  
  The Correct Model: Store Intent, Derive Math
</h2>

<p>Here's the model that actually works:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Source of truth:    local + timeZoneId
Derived value:      instantUtc
</code></pre>

</div>



<p>For a meeting at 10:00 Vienna:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>local:        2026-06-05T10:00
timeZoneId:   Europe/Vienna
instantUtc:   2026-06-05T08:00:00Z  (derived)
</code></pre>

</div>



<p><strong>What you store as truth:</strong></p>

<ul>
<li>The local datetime the user chose (10:00)</li>
<li>The IANA timezone (Europe/Vienna)</li>
</ul>

<p><strong>What you derive for queries:</strong></p>

<ul>
<li>The UTC instant (computed from local + zone)</li>
</ul>




<h2>
  
  
  Why IANA Timezones Matter
</h2>

<p>An IANA timezone like <code>Europe/Vienna</code> is not just an offset. It's a <strong>ruleset</strong>:</p>

<ul>
<li>Historical offsets (what was the offset in 1980?)</li>
<li>Current offset</li>
<li>DST transitions (when does the clock change?)</li>
<li>Future rules (as currently known)</li>
<li>Political changes (when a country changes its timezone policy)</li>
</ul>

<p>When you store <code>Europe/Vienna</code>, you're saying: "Apply whatever rules Vienna has — past, present, or future."</p>

<p>This is why the model survives rule changes.</p>




<h2>
  
  
  What Happens When Rules Change
</h2>

<p>Timezone rules change more often than you'd think:</p>

<ul>
<li>
<strong>Russia</strong> abolished DST in 2011</li>
<li>
<strong>Mexico</strong> abolished DST in 2022</li>
<li>
<strong>EU</strong> has been discussing abolishing DST for years</li>
<li>
<strong>Morocco</strong> changes its offset for Ramadan every year</li>
<li>
<strong>Egypt</strong> — the chaos champion: introduced DST in 1940, suspended it in 2011, reintroduced it in 2014, suspended it <em>during Ramadan</em> that same year, abolished it in 2015, restored it in 2023</li>
</ul>

<p>With the correct model:</p>

<p><strong>For past events:</strong></p>

<ul>
<li>The IANA database contains historical rules</li>
<li>Your <code>instantUtc</code> was correct at the time and stays correct</li>
<li>Recalculating gives the same result (safe, idempotent)</li>
</ul>

<p><strong>For future events:</strong></p>

<ul>
<li>When rules change, you <strong>recalculate</strong> <code>instantUtc</code>
</li>
<li>The human intent (10:00 Vienna) stays the same</li>
<li>The physical moment shifts to match the new rules</li>
</ul>




<h2>
  
  
  When Do You Actually Recalculate?
</h2>

<p>So <code>instantUtc</code> "can always be recalculated" — but when does that actually happen?</p>

<p><strong>Trigger: Your IANA timezone database gets updated.</strong></p>

<p>The IANA maintains the <a href="https://www.iana.org/time-zones" rel="noopener noreferrer">tz database</a>, which gets updated several times a year. When a country announces a DST change or offset shift, the database is updated, and eventually that update reaches your system:</p>

<ul>
<li>Your OS ships a tzdata update</li>
<li>NodaTime releases a new tzdb version</li>
<li>Your runtime (JVM, .NET, Node) updates its bundled data</li>
</ul>

<p><strong>What to do when this happens:</strong></p>

<ol>
<li><p><strong>Past events</strong>: Nothing. Recalculating them is safe — the result will be identical because historical rules don't change.</p></li>
<li><p><strong>Future events</strong>: Recalculate <code>instantUtc</code> for any event whose timezone was affected.</p></li>
</ol>

<p><strong>How to implement this:</strong></p>

<ul>
<li>
<strong>Batch job</strong>: When you update your tz data, run a migration that recalculates <code>instantUtc</code> for future events in affected zones.</li>
<li>
<strong>Lazy recalculation</strong>: Recompute <code>instantUtc</code> on read, and store the tz database version used. If it's stale, recalculate.</li>
<li>
<strong>Hybrid</strong>: Batch for known changes, lazy as a safety net.</li>
</ul>

<p>Most applications can get away with a simple batch job that runs after tz updates. If you're caching <code>instantUtc</code> in a search index or external system, remember to update those too.</p>

<p>The good news: this is rare (a few times a year, usually affecting only a handful of zones), and if you've stored <code>local + timeZoneId</code>, you have everything you need to recalculate correctly.</p>




<h2>
  
  
  The Two Query Patterns
</h2>

<p>This model supports both ways of querying time:</p>

<h3>
  
  
  Pattern A: "What's on my calendar on June 5th in Vienna?"
</h3>

<p>Query by local datetime + timezone:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WHERE</span> <span class="n">time_zone_id</span> <span class="o">=</span> <span class="s1">'Europe/Vienna'</span>
  <span class="k">AND</span> <span class="n">local_start</span> <span class="o">&gt;=</span> <span class="s1">'2026-06-05'</span>
  <span class="k">AND</span> <span class="n">local_start</span> <span class="o">&lt;</span>  <span class="s1">'2026-06-06'</span>
</code></pre>

</div>



<p>This finds everything on that calendar day, regardless of the global instant.</p>

<h3>
  
  
  Pattern B: "What's happening globally at this moment?"
</h3>

<p>Query by instant:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">WHERE</span> <span class="n">instant_utc</span> <span class="o">=</span> <span class="s1">'2026-06-05T08:00:00Z'</span>
</code></pre>

</div>



<p>This finds everything happening at that physical moment, regardless of local calendars.</p>

<p>Both queries are valid. Both are useful. The model supports both.</p>




<h2>
  
  
  How Many Columns Do You Actually Need?
</h2>

<p>Not every time field needs the same treatment. It depends on what you're storing:</p>

<p><strong>Instant-only fields</strong> (logs, audits, <code>occurredAt</code>):<br>
→ <strong>1 column</strong> — <code>timestamp with time zone</code> or <code>Instant</code></p>

<p><strong>Date-only fields</strong> (birthday, holiday):<br>
→ <strong>1 column</strong> — <code>date</code> or <code>LocalDate</code></p>

<p><strong>Human-scheduled time</strong> (appointment, meeting, deadline):<br>
→ <strong>2 columns minimum</strong>:</p>

<ul>
<li>
<code>local_start</code> (<code>timestamp without time zone</code>)</li>
<li>
<code>time_zone_id</code> (<code>text</code> — IANA zone)</li>
</ul>

<p><strong>Optional 3rd column</strong>: <code>instant_utc</code> as a cached/derived value — only if you need efficient global queries.</p>


<h2>
  
  
  A Practical Example
</h2>

<p>For one human-facing timestamp (like an appointment start), the robust representation is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Example using PostgreSQL types (concepts apply to any database)</span>
<span class="n">local_start</span>    <span class="nb">timestamp</span> <span class="k">without</span> <span class="nb">time</span> <span class="k">zone</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
<span class="n">time_zone_id</span>   <span class="nb">text</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>  <span class="c1">-- IANA zone, e.g. 'Europe/Vienna'</span>

<span class="c1">-- Optional: cached for fast global queries/sorting</span>
<span class="n">instant_utc</span>    <span class="nb">timestamp</span> <span class="k">with</span> <span class="nb">time</span> <span class="k">zone</span>
</code></pre>

</div>



<p>The exact types vary by database (<code>datetime</code> in SQL Server, <code>DATETIME</code> in MySQL, etc.), but the pattern is the same: <strong>local datetime + timezone ID + optional UTC instant</strong>.</p>

<p><strong>When do you need <code>instant_utc</code>?</strong></p>

<ul>
<li>Indexing and range queries across timezones</li>
<li>Sorting events globally</li>
<li>"What's happening right now?" queries</li>
<li>Avoiding recalculation in hot paths</li>
</ul>

<p>If you never query globally, you can skip it and compute on read.</p>

<p><strong>In practice</strong>, many teams wrap these two (or three) values into:</p>

<ul>
<li>An app-level value object (<code>ZonedDateTime</code>, a custom DTO)</li>
<li>A composite database type</li>
<li>A domain type with constraints</li>
</ul>

<p>The logical model stays the same — you're just packaging it.</p>




<h2>
  
  
  The Mantra
</h2>

<blockquote>
<p><strong>Store intent, not math.</strong></p>
</blockquote>

<ul>
<li>
<code>local + timeZoneId</code> = what the human meant</li>
<li>
<code>instantUtc</code> = what the computer needs for calculations</li>
</ul>

<p>The human intent is the source of truth. The UTC instant is derived, cached, and can always be recalculated.</p>

<p>If you store only UTC, you've thrown away the intent. You can never get it back.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>UTC is perfect for <strong>physical moments</strong> (logs, audits, tokens)</li>
<li>UTC alone is wrong for <strong>human intent</strong> (meetings, deadlines, appointments)</li>
<li>Offsets are snapshots, not meaning — they can't survive rule changes</li>
<li>The correct model: <strong>local + IANA timezone</strong> as truth, <strong>UTC instant</strong> as derived</li>
<li>IANA timezones contain rules, not just offsets — they handle past and future</li>
<li>When rules change: past events stay correct, future events get recalculated</li>
<li>Store intent, not math</li>
</ul>




<p><em>Next up: <a href="https://dev.to/bwi/global-events-local-events-and-recurring-rules-3bhc"><strong>Global Events, Local Events, and Recurring Rules</strong></a> — when "everyone at 10:00" means completely different things.</em></p>

