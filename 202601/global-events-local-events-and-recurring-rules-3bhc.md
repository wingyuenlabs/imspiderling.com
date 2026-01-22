---
Title: Global Events, Local Events, and Recurring Rules
Description: 
Author: bwi
Date: 2026-01-22T22:03:41.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 5 of 8</strong> in the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">Time in Software, Done Right</a></p>
</blockquote>




<p>"All stores open at 10:00."</p>

<p>Simple requirement, right? But this sentence hides a massive ambiguity that will determine your entire data model.</p>

<p>Do all stores open <strong>at the same moment worldwide</strong> (a global event)?<br><br>
Or does each store open <strong>when its local clock shows 10:00</strong> (a local event)?</p>

<p>Same words. Completely different meanings. Completely different storage.</p>

<p>This article will help you spot the difference — and handle the even trickier case of <strong>recurring events</strong>, where Daylight Saving Time (DST) creates times that don't exist and times that exist twice.</p>


<h2>
  
  
  Global Events vs Local Events
</h2>

<p>Let's make the distinction crystal clear.</p>
<h3>
  
  
  Global Event: Same Moment, Different Clocks
</h3>

<p>A <strong>global event</strong> happens at one physical instant. Everyone experiences it at the same moment, but their clocks show different times.</p>

<p><strong>Examples:</strong></p>

<ul>
<li>
<strong>Product launch</strong>: "New iPhone available at 10:00 AM Pacific" — that's 7:00 PM in Vienna</li>
<li>
<strong>Livestream start</strong>: "Keynote begins at 18:00 UTC" — same instant everywhere</li>
<li>
<strong>Stock market close</strong>: The NYSE closes at one moment, regardless of where traders are</li>
<li>
<strong>Server maintenance window</strong>: "Downtime from 02:00-04:00 UTC"</li>
</ul>

<p><strong>How to store:</strong> <code>Instant</code> (UTC). Everyone converts to their local clock for display.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>event_type:     global
instant_utc:    2026-06-05T17:00:00Z
</code></pre>

</div>



<p>When you query "what's happening now?", you compare against the current instant. Done.</p>




<h3>
  
  
  Local Event: Same Clock, Different Moments
</h3>

<p>A <strong>local event</strong> happens when local clocks show a specific time. The physical moment is different for each timezone.</p>

<p><strong>Examples:</strong></p>

<ul>
<li>
<strong>Store opening hours</strong>: "All stores open at 10:00" — means 10:00 in London, 10:00 in Vienna, 10:00 in Tokyo</li>
<li>
<strong>TV broadcast</strong>: "News at 8 PM" — 8 PM in your timezone</li>
<li>
<strong>Daily standup</strong>: "Team meeting at 09:00 Vienna time"</li>
<li>
<strong>Deadline</strong>: "Submit by 23:59 in your local timezone"</li>
</ul>

<p><strong>How to store:</strong> <code>local_start + time_zone_id</code>. Each location has its own timezone.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>event_type:     local
local_start:    10:00
time_zone_id:   (per location)
</code></pre>

</div>



<p>When the London store opens at 10:00 AM GMT, the Vienna store is still closed (it's 11:00 AM there). The Vienna store opens when <em>its</em> clock shows 10:00.</p>




<h3>
  
  
  The Critical Question
</h3>

<p>When someone says "all X happen at 10:00", ask:</p>

<blockquote>
<p>"Do they all happen <strong>at the same moment</strong> (global), or do they each happen <strong>when local clocks show 10:00</strong> (local)?"</p>
</blockquote>

<p>The answer changes everything:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Requirement</th>
<th>Type</th>
<th>Storage</th>
</tr>
</thead>
<tbody>
<tr>
<td>"Product available at 10:00 Pacific"</td>
<td>Global</td>
<td><code>Instant</code></td>
</tr>
<tr>
<td>"Stores open at 10:00"</td>
<td>Local</td>
<td>
<code>LocalTime + TimeZone</code> per location</td>
</tr>
<tr>
<td>"Meeting at 14:00 Vienna"</td>
<td>Local</td>
<td><code>LocalDateTime + TimeZone</code></td>
</tr>
<tr>
<td>"Server restart at 03:00 UTC"</td>
<td>Global</td>
<td><code>Instant</code></td>
</tr>
</tbody>
</table></div>

<p>Get this wrong, and your stores either all open at the wrong time, or your "global launch" happens at different moments in different regions.</p>




<h2>
  
  
  Recurring Rules: A Different Beast
</h2>

<p>Recurring events aren't timestamps at all. They're <strong>rules</strong> that generate timestamps.</p>

<ul>
<li>"Every Monday at 10:00"</li>
<li>"First Friday of each month"</li>
<li>"Every day at 03:00"</li>
<li>"Yearly on December 25th"</li>
</ul>

<p>You don't store a list of dates. You store the <strong>rule</strong>, and compute the occurrences when needed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>recurrence_rule:    FREQ=WEEKLY;BYDAY=MO
local_time:         10:00
time_zone_id:       Europe/Vienna
</code></pre>

</div>



<p>This rule, combined with the timezone, generates concrete <code>ZonedDateTime</code> instances: Monday June 1st at 10:00 Vienna, Monday June 8th at 10:00 Vienna, and so on.</p>

<h3>
  
  
  Why Recurrence Must Be Local
</h3>

<p>Recurring events are almost always <strong>local events</strong>. </p>

<p>"Team standup every Monday at 10:00" means 10:00 on the team's wall clock — not a fixed UTC instant that drifts relative to their calendar as DST changes.</p>

<p>If you store a recurring event as "every Monday at 08:00 UTC" (because Vienna is UTC+2 in summer), what happens in winter when Vienna switches to UTC+1? Your 08:00 UTC becomes <strong>09:00 Vienna time</strong>. The standup just moved by an hour.</p>

<p>Store the rule in local time. Let the timezone handle DST.</p>




<h2>
  
  
  DST Edge Cases: The Hard Part
</h2>

<p>Here's where recurring rules get interesting. DST creates two nasty edge cases that trip up even experienced teams — if you haven't encountered them yet, you probably will.</p>

<h3>
  
  
  Edge Case 1: The Time That Doesn't Exist
</h3>

<p>In most of Europe, clocks spring forward on the last Sunday of March. At 02:00, clocks jump to 03:00.</p>

<p><strong>The times 02:00, 02:15, 02:30, 02:45 don't exist that day.</strong></p>

<p>What if you have a recurring event at 02:30 every day?</p>

<ul>
<li>March 28th at 02:30 Vienna — exists normally</li>
<li>March 29th at 02:30 Vienna — <strong>doesn't exist</strong> (clocks skip from 01:59 to 03:00)</li>
<li>March 30th at 02:30 Vienna — exists normally</li>
</ul>

<p><strong>What should happen?</strong></p>

<p>Most systems choose one of:</p>

<ul>
<li>
<strong>Skip</strong> the occurrence (it's in a gap, treat it as if it didn't happen)</li>
<li>
<strong>Shift forward</strong> to 03:00 (the first valid moment after the gap)</li>
<li>
<strong>Shift backward</strong> to 01:59 (the last valid moment before the gap)</li>
</ul>

<p>NodaTime calls these resolution strategies. There's no universally "correct" answer — it depends on your domain.</p>

<p>A 02:30 AM cron job that does cleanup? Probably shift forward to 03:00.<br><br>
A 02:30 AM medication reminder? Maybe shift backward — better early than missed.</p>

<p><strong>The point:</strong> You need to decide, and your code needs to handle it explicitly.</p>




<h3>
  
  
  Edge Case 2: The Time That Exists Twice
</h3>

<p>In autumn, clocks fall back. In most of Europe, at 03:00 on the last Sunday of October, clocks jump back to 02:00.</p>

<p><strong>The times 02:00 to 02:59 exist twice that day</strong> — once before the change, once after.</p>

<p>What if you have a recurring event at 02:30?</p>

<ul>
<li>October 25th at 02:30 Vienna — exists once</li>
<li>October 26th at 02:30 Vienna — <strong>exists twice</strong> (once in summer time, once in winter time)</li>
<li>October 27th at 02:30 Vienna — exists once</li>
</ul>

<p><strong>What should happen?</strong></p>

<p>Options:</p>

<ul>
<li>
<strong>Pick the first occurrence</strong> (summer time, before clocks change)</li>
<li>
<strong>Pick the second occurrence</strong> (winter time, after clocks change)</li>
<li>
<strong>Trigger both</strong> (if it's a job that should run twice)</li>
</ul>

<p>Again, domain-dependent. A daily backup at 02:30? Probably run once, pick whichever. A reminder? Probably the first one.</p>




<h2>
  
  
  Cron Jobs vs Calendar Events
</h2>

<p>Cron jobs and calendar events <em>look</em> similar but behave differently with DST.</p>

<h3>
  
  
  Cron: Usually System Time (Often UTC)
</h3>

<p>Traditional cron runs on the system clock. If your server is in UTC, a <code>0 2 * * *</code> job runs at 02:00 UTC every day — no DST weirdness, but it drifts relative to local calendars.</p>

<p>If your server uses local time, cron will experience DST gaps and overlaps. Most cron implementations:</p>

<ul>
<li>
<strong>Skip</strong> jobs that fall into a gap</li>
<li>
<strong>Run once</strong> for jobs that fall into an overlap (not twice)</li>
</ul>

<p>But this varies by implementation. Check your system's documentation.</p>

<h3>
  
  
  Calendar Events: User Intent
</h3>

<p>Calendar events are about <strong>user intent</strong>. "Remind me at 09:00 every day" means 09:00 on my wall clock, regardless of DST.</p>

<p>If you're building a calendar or reminder system, you need:</p>

<ol>
<li>Store the rule in local time + timezone</li>
<li>Compute occurrences by applying the rule to the timezone</li>
<li>Handle gaps and overlaps with explicit resolution strategies</li>
</ol>




<h2>
  
  
  Holidays: The Complex Recurrence Rules
</h2>

<p>Some recurring events don't follow simple patterns:</p>

<ul>
<li>
<strong>Easter</strong>: First Sunday after the ecclesiastical full moon on or after March 21st (yes, really)</li>
<li>
<strong>Thanksgiving (US)</strong>: Fourth Thursday of November</li>
<li>
<strong>Ramadan</strong>: Based on the Islamic lunar calendar — starts on a different Gregorian date each year</li>
<li>
<strong>Chinese New Year</strong>: Based on the lunisolar calendar</li>
</ul>

<p>For these, you typically need:</p>

<ul>
<li>A specialized library (like NodaTime's <code>IsoDayOfWeek</code> calculations, or dedicated holiday libraries)</li>
<li>A database of dates (for religious holidays that depend on moon sightings)</li>
<li>Possibly user input ("Ramadan starts on X this year")</li>
</ul>

<p>Don't try to compute Easter from scratch. Use a library.</p>




<h2>
  
  
  Practical Guidance
</h2>

<h3>
  
  
  For One-Time Events
</h3>

<p>Ask: "Same moment worldwide, or same local time?"</p>

<ul>
<li>
<strong>Same moment:</strong> Store as <code>Instant</code> (UTC)</li>
<li>
<strong>Same local time:</strong> Store as <code>local_start + time_zone_id</code>
</li>
</ul>

<h3>
  
  
  For Recurring Events
</h3>

<ol>
<li>
<strong>Store the rule</strong>, not a list of occurrences</li>
<li>
<strong>Store local time + timezone</strong>, not UTC</li>
<li>
<strong>Decide your DST resolution strategy</strong> (skip, shift forward, shift backward)</li>
<li>
<strong>Document it</strong> — future you will thank present you</li>
</ol>

<h3>
  
  
  For Cron Jobs
</h3>

<ul>
<li>If possible, run in UTC to avoid DST surprises</li>
<li>If you must use local time, test what happens on DST transition days</li>
<li>Consider whether a skipped or doubled job matters for your use case</li>
</ul>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>
<strong>Global events</strong> happen at one instant; store as UTC</li>
<li>
<strong>Local events</strong> happen when local clocks match; store as local time + timezone</li>
<li>
<strong>Recurring rules</strong> generate timestamps; store the rule, not the instances</li>
<li>
<strong>DST creates gaps</strong> (times that don't exist) and <strong>overlaps</strong> (times that exist twice)</li>
<li>
<strong>Cron and calendars behave differently</strong> with DST — know which you're building</li>
<li>
<strong>Complex holidays</strong> (Easter, Ramadan) need specialized libraries or data</li>
<li>When in doubt, ask: "Same moment, or same clock reading?"</li>
</ul>




<p><em>Next up: <a href="https://dev.to/bwi/net-in-practice-modeling-time-with-nodatime-o6d"><strong>.NET in Practice – Modeling Time with NodaTime</strong></a> — making all this theory real in code.</em></p>

