---
Title: The 7 Types of Time Every Developer Must Know
Description: 
Author: bwi
Date: 2026-01-22T21:55:31.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 2 of 8</strong> in the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">Time in Software, Done Right</a></p>
</blockquote>




<p>In the <a href="https://dev.to/bwi/why-a-date-is-not-a-point-in-time-ad8">previous article</a>, we established that a date is not a moment. But "date" and "moment" are just two of the many ways time shows up in software.</p>

<p>Most datetime bugs come from <strong>mixing up different types of time</strong>. Teams argue because one person says "timestamp" and means UTC, while another means local time. Someone stores an offset thinking it captures the timezone. Someone else treats a recurring rule like a single event.</p>

<p>Before we can write correct code, we need a shared vocabulary.</p>

<p>Here are the <strong>7 types of time</strong> you'll encounter in software — and when to use each one.</p>




<h2>
  
  
  1. Instant (Global Moment)
</h2>

<p>An <strong>Instant</strong> is a point in physical time that happens <strong>simultaneously everywhere on Earth</strong>.</p>

<p>When it's 14:00 in Vienna and 13:00 in London — that's the same instant, just displayed differently.</p>

<p><strong>Technical representations:</strong></p>

<ul>
<li>Unix timestamp (seconds/milliseconds since 1970-01-01 UTC)</li>
<li>ISO 8601 with <code>Z</code>: <code>2026-01-21T13:05:12Z</code>
</li>
<li>NodaTime: <code>Instant</code>
</li>
<li>JavaScript Temporal: <code>Temporal.Instant</code>
</li>
</ul>

<p><strong>Use it for:</strong></p>

<ul>
<li>Log entries ("request received at...")</li>
<li>Audit trails</li>
<li>Token expiry ("expires in 3600 seconds")</li>
<li>Event sourcing (<code>occurredAtUtc</code>)</li>
<li>Anything where you need to answer: "Did X happen before or after Y?"</li>
</ul>

<p><strong>Key property:</strong> An Instant has no timezone, no calendar — it's pure physics. To <em>display</em> it to a human, you must convert it to a local time in some timezone.</p>




<h2>
  
  
  2. Local Date (Calendar Day)
</h2>

<p>A <strong>Local Date</strong> is a day on the calendar — nothing more.</p>

<ul>
<li>December 25th</li>
<li>October 31st</li>
<li>Your birthday</li>
<li>A deadline written as "by June 5th"</li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>
<code>2026-06-05</code> (ISO 8601 date only)</li>
<li>NodaTime: <code>LocalDate</code>
</li>
<li>JavaScript Temporal: <code>Temporal.PlainDate</code>
</li>
</ul>

<p><strong>Use it for:</strong></p>

<ul>
<li>Holidays</li>
<li>Birthdays</li>
<li>Date-only fields ("date of birth", "release date")</li>
<li>Any business concept where time-of-day doesn't matter</li>
</ul>

<p><strong>Key property:</strong> A Local Date is <strong>not</strong> a moment. It has no timezone. Christmas in Sydney and Christmas in New York are the same <em>date</em> but different <em>instants</em>. You cannot convert a Local Date to UTC without adding a time and a timezone.</p>




<h2>
  
  
  3. Local Time (Time of Day)
</h2>

<p>A <strong>Local Time</strong> is a time without a date — just hours, minutes, seconds.</p>

<ul>
<li>"Daily standup at 09:00"</li>
<li>"Store opens at 10:00"</li>
<li>"Alarm at 07:30"</li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>
<code>10:00:00</code> (ISO 8601 time only)</li>
<li>NodaTime: <code>LocalTime</code>
</li>
<li>JavaScript Temporal: <code>Temporal.PlainTime</code>
</li>
</ul>

<p><strong>Use it for:</strong></p>

<ul>
<li>Opening hours</li>
<li>Recurring daily schedules</li>
<li>Alarm clocks</li>
<li>"Every day at X" patterns</li>
</ul>

<p><strong>Key property:</strong> A Local Time is a <strong>rule</strong>, not an event. "10:00" happens every day, in every timezone. It's not a moment — it's a pattern. And on Daylight Saving Time (DST) transition days, 02:30 might not exist at all (or exist twice).</p>




<h2>
  
  
  4. Local DateTime (Date + Time, No Zone)
</h2>

<p>A <strong>Local DateTime</strong> combines a date and a time — but without a timezone.</p>

<ul>
<li>"June 5th, 10:00"</li>
<li>"Meeting next Tuesday at 14:30"</li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>
<code>2026-06-05T10:00:00</code> (no <code>Z</code>, no offset)</li>
<li>NodaTime: <code>LocalDateTime</code>
</li>
<li>JavaScript Temporal: <code>Temporal.PlainDateTime</code>
</li>
</ul>

<p><strong>Use it for:</strong></p>

<ul>
<li>The "raw" input from a user before you know their timezone</li>
<li>Intermediate calculations</li>
<li>Display purposes (when the timezone is implied by context)</li>
</ul>

<p><strong>Key property:</strong> A Local DateTime is <strong>still not a moment</strong>. "June 5th 10:00" in Vienna and "June 5th 10:00" in London are different instants. To get a real moment, you need to pair it with a timezone.</p>

<p>This is where the magic happens:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>LocalDateTime + Timezone → Instant
</code></pre>

</div>






<h2>
  
  
  5. Zoned DateTime (The Complete Picture)
</h2>

<p>A <strong>Zoned DateTime</strong> is a Local DateTime paired with an IANA timezone.</p>

<ul>
<li>"June 5th, 10:00 in Europe/Vienna"</li>
<li>"Meeting at 14:30 Europe/London"</li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>NodaTime: <code>ZonedDateTime</code>
</li>
<li>JavaScript Temporal: <code>Temporal.ZonedDateTime</code>
</li>
</ul>

<p><strong>Use it for:</strong></p>

<ul>
<li>Meetings</li>
<li>Appointments</li>
<li>Deadlines with a clear location/context</li>
<li>Any human-facing time that needs to survive DST changes</li>
</ul>

<p><strong>Key property:</strong> This is what you should store for human-meaningful times. The IANA timezone (like <code>Europe/Vienna</code>) contains all the rules — historical DST, future DST, political changes. From this, you can always compute the correct Instant.</p>

<p><strong>This is the model:</strong></p>

<ul>
<li>Store: <code>LocalDateTime</code> + <code>TimeZoneId</code>
</li>
<li>Derive: <code>Instant</code> (for queries and sorting)</li>
</ul>

<p>If timezone rules change (DST abolished, offset shifted), you recalculate the Instant. The human intent ("10:00 in Vienna") stays correct.</p>




<h2>
  
  
  6. Offset DateTime (Snapshot, Not Intent)
</h2>

<p>An <strong>Offset DateTime</strong> is a date + time + UTC offset.</p>

<ul>
<li><code>2026-06-05T10:00:00+02:00</code></li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>ISO 8601 with offset</li>
<li>NodaTime: <code>OffsetDateTime</code>
</li>
<li>.NET: <code>DateTimeOffset</code>
</li>
<li>JavaScript Temporal: <code>Temporal.Instant</code> (formatted with offset)</li>
</ul>

<p><strong>Use it for:</strong></p>

<ul>
<li>Recording "what the clock showed at the moment of capture"</li>
<li>Serialization when you don't have full timezone info</li>
<li>Interoperability with systems that only support offsets</li>
</ul>

<p><strong>Key property:</strong> An offset is a <strong>snapshot</strong>, not a <strong>meaning</strong>. <code>+02:00</code> could be Vienna in summer, Berlin in summer, Cairo, Johannesburg, or Kaliningrad — completely ambiguous.</p>

<p>The offset tells you the math at <em>that</em> moment, but it doesn't tell you:</p>

<ul>
<li>Which region it was</li>
<li>Whether DST was in effect</li>
<li>What the offset will be next year</li>
</ul>

<p><strong>Don't use offsets as a substitute for timezones</strong> when storing human-facing times. If DST rules change, you can't recalculate — you've lost the intent.</p>

<blockquote>
<p>⚠️ <strong>Never use timezone abbreviations</strong> like "EST", "CST", or "GMT" — they're ambiguous. "EST" means Eastern Standard Time in both the US <em>and</em> Australia. "CST" is used in the US, Australia, China, and Cuba. Always use IANA timezone IDs like <code>Europe/London</code> or <code>Europe/Vienna</code>.</p>
</blockquote>




<h2>
  
  
  7. Recurring Rule (Pattern, Not Event)
</h2>

<p>A <strong>Recurring Rule</strong> is not a timestamp — it's a <strong>pattern</strong> that generates timestamps.</p>

<ul>
<li>"Every Monday at 09:00"</li>
<li>"First day of every month"</li>
<li>"Annually on December 25th"</li>
<li>"Every 15 minutes"</li>
<li>Cron: <code>0 9 * * 1</code>
</li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>iCalendar RRULE</li>
<li>Cron expressions</li>
<li>Custom domain models</li>
</ul>

<p><strong>Use it for:</strong></p>

<ul>
<li>Recurring meetings</li>
<li>Scheduled jobs</li>
<li>Subscription billing cycles</li>
<li>Holidays with complex rules (Easter, Ramadan)</li>
</ul>

<p><strong>Key property:</strong> A recurring rule produces <em>Local DateTimes</em>. To get actual Instants, you need to apply a timezone — and handle edge cases like DST gaps (what happens to "02:30 every day" when the clock skips from 02:00 to 03:00?).</p>




<h2>
  
  
  The Cheat Sheet
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>Has Date?</th>
<th>Has Time?</th>
<th>Has Zone?</th>
<th>Is a Moment?</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Instant</strong></td>
<td>No</td>
<td>No</td>
<td>No (UTC)</td>
<td>✅ Yes</td>
<td>Log timestamp</td>
</tr>
<tr>
<td><strong>Local Date</strong></td>
<td>✅ Yes</td>
<td>No</td>
<td>No</td>
<td>❌ No</td>
<td>Birthday</td>
</tr>
<tr>
<td><strong>Local Time</strong></td>
<td>No</td>
<td>✅ Yes</td>
<td>No</td>
<td>❌ No</td>
<td>"Opens at 10:00"</td>
</tr>
<tr>
<td><strong>Local DateTime</strong></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>No</td>
<td>❌ No</td>
<td>User input</td>
</tr>
<tr>
<td><strong>Zoned DateTime</strong></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>Meeting</td>
</tr>
<tr>
<td><strong>Offset DateTime</strong></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>Partial</td>
<td>✅ Yes</td>
<td>Snapshot</td>
</tr>
<tr>
<td><strong>Recurring Rule</strong></td>
<td>Pattern</td>
<td>Pattern</td>
<td>Needs one</td>
<td>❌ No</td>
<td>"Every Monday"</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p>📝 <strong>Other types you might need:</strong> Some domains use <code>YearMonth</code> (credit card expiry: "03/2027", financial periods) and <code>MonthDay</code> (annual events: "March 15th" without a year). NodaTime has <code>YearMonth</code>; Temporal has <code>Temporal.PlainYearMonth</code> and <code>Temporal.PlainMonthDay</code>.</p>
</blockquote>




<h2>
  
  
  Why This Matters
</h2>

<p>When you can name the type of time you're dealing with, you can:</p>

<ol>
<li>
<strong>Choose the right storage format</strong> — don't store a Local Date as a timestamp</li>
<li>
<strong>Avoid silent conversions</strong> — don't let the framework "helpfully" add a timezone</li>
<li>
<strong>Communicate clearly</strong> — say "we need a Zoned DateTime here" instead of arguing about "timestamps"</li>
<li>
<strong>Catch bugs early</strong> — if someone passes a Local DateTime where you need an Instant, that's a type error</li>
</ol>

<p>Most datetime bugs are <strong>category errors</strong> — treating one type of time as another. A shared vocabulary prevents that.</p>




<h2>
  
  
  Bonus: Two Types of Time Amounts
</h2>

<p>The 7 types above describe <em>when</em> something happens. But you'll also need to express <em>how long</em> — and there are two very different ways:</p>

<h3>
  
  
  Duration (Physical Elapsed Time)
</h3>

<p>A <strong>Duration</strong> is an exact, fixed amount of time — measured in hours, minutes, seconds, or smaller units.</p>

<ul>
<li>90 minutes</li>
<li>3600 seconds</li>
<li>2.5 hours</li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>NodaTime: <code>Duration</code>
</li>
<li>JavaScript Temporal: <code>Temporal.Duration</code> (with time units)</li>
<li>ISO 8601: <code>PT1H30M</code> (1 hour 30 minutes)</li>
</ul>

<p><strong>Key property:</strong> A Duration is always the same length, no matter when you apply it. 90 minutes is 90 minutes — in January, in June, in any timezone.</p>

<h3>
  
  
  Period (Calendrical Amount)
</h3>

<p>A <strong>Period</strong> is a calendrical amount — measured in days, months, or years.</p>

<ul>
<li>1 month</li>
<li>2 weeks</li>
<li>1 year</li>
</ul>

<p><strong>Technical representations:</strong></p>

<ul>
<li>NodaTime: <code>Period</code>
</li>
<li>JavaScript Temporal: <code>Temporal.Duration</code> (with date units)</li>
<li>ISO 8601: <code>P1M</code> (1 month), <code>P1Y</code> (1 year)</li>
</ul>

<p><strong>Key property:</strong> A Period's actual length <strong>depends on when you apply it</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>January 31 + 1 month = February 28 (or 29)  → 28-29 days
March 31 + 1 month = April 30              → 30 days
July 31 + 1 month = August 31              → 31 days
</code></pre>

</div>



<p>"Add 1 month" and "add 30 days" are <strong>not the same operation</strong>.</p>

<h3>
  
  
  Why This Distinction Matters
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// These look similar but behave differently:</span>
<span class="kt">var</span> <span class="n">in30Days</span> <span class="p">=</span> <span class="n">today</span><span class="p">.</span><span class="nf">PlusDays</span><span class="p">(</span><span class="m">30</span><span class="p">);</span>           <span class="c1">// Always exactly 30 days</span>
<span class="kt">var</span> <span class="n">nextMonth</span> <span class="p">=</span> <span class="n">today</span><span class="p">.</span><span class="nf">Plus</span><span class="p">(</span><span class="n">Period</span><span class="p">.</span><span class="nf">FromMonths</span><span class="p">(</span><span class="m">1</span><span class="p">));</span>  <span class="c1">// Depends on the month</span>

<span class="c1">// "Renew subscription in 1 month" vs "Renew in 30 days"</span>
<span class="c1">// — different business meanings, different results</span>
</code></pre>

</div>



<p>When someone says "add a month," ask: do they mean a calendar month (Period) or 30 days (Duration)? The answer matters for billing cycles, contract renewals, and any date arithmetic involving months or years.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>There are (at least) 7 distinct types of time in software</li>
<li>
<strong>Instant</strong>, <strong>Zoned DateTime</strong>, and <strong>Offset DateTime</strong> represent actual moments — but only Zoned DateTime preserves the human intent</li>
<li>
<strong>Offsets</strong> are snapshots, not intent — prefer IANA timezones for human-facing times</li>
<li>
<strong>Recurring rules</strong> generate times, they aren't times themselves</li>
<li>If you can't name the type of time you're working with, you're probably about to introduce a bug</li>
</ul>

<blockquote>
<p>💡 <strong>What about leap seconds?</strong> We're ignoring them — intentionally. Unix time ignores them, NodaTime ignores them by default, and databases ignore them. For 99.9% of applications, this is correct. If you're building satellite navigation or scientific instrumentation, you already know who you are.</p>
</blockquote>




<p><em>Next up: <a href="https://dev.to/bwi/deadlines-are-hard-why-submit-by-june-5th-is-broken-1gl4"><strong>Deadlines Are Hard</strong></a> — why "Submit by June 5th" is a broken requirement, and what to do about it.</em></p>

