---
Title: Deadlines Are Hard – Why "Submit by June 5th" Is Broken
Description: 
Author: bwi
Date: 2026-01-22T21:57:19.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Part 3 of 8</strong> in the series <a href="https://dev.to/bwi/time-in-software-done-right-3mjb">Time in Software, Done Right</a></p>
</blockquote>




<p>"Submit by June 5th."</p>

<p>Sounds clear, right? Everyone knows what June 5th means.</p>

<p>Except... they don't. And if you build a system around this requirement, you're setting up your users to fail.</p>

<p>This article is about <strong>deadline fairness</strong> — and why most deadline definitions are technically broken.</p>




<h2>
  
  
  The Problem in One Sentence
</h2>

<blockquote>
<p>If a deadline is defined in a way that users cannot objectively act correctly, the problem is not the user — it's the deadline.</p>
</blockquote>

<p>A user shouldn't need to guess what timezone the server is in. They shouldn't need to submit "just to be safe" hours before they think the deadline is. They shouldn't lose because of ambiguity they can't control.</p>




<h2>
  
  
  What's Missing from "June 5th"?
</h2>

<p>When someone says "the deadline is June 5th", what do they actually mean?</p>

<ul>
<li>June 5th at 00:00? (start of day)</li>
<li>June 5th at 23:59? (end of day)</li>
<li>June 5th at 17:00? (end of business hours)</li>
<li>Some other time?</li>
</ul>

<p>And more importantly: <strong>in which timezone?</strong></p>

<ul>
<li>The user's timezone?</li>
<li>The server's timezone?</li>
<li>The organization's timezone?</li>
<li>UTC?</li>
</ul>

<p>"June 5th" contains none of this information. It's a <strong>Local Date</strong> — and as we established in <a href="https://dev.to/bwi/why-a-date-is-not-a-point-in-time-ad8">Article 1</a>, a Local Date is not a moment.</p>




<h2>
  
  
  The Three Timezones in Every Deadline
</h2>

<p>When a deadline involves a user and a server, there are (at least) three timezones in play:</p>

<h3>
  
  
  1. User Time
</h3>

<p>Where the user physically is. A user in Sydney sees their clock, their calendar, their "June 5th".</p>

<h3>
  
  
  2. Server Time
</h3>

<p>Where the server runs. AWS <code>us-east-1</code>, Azure <code>westeurope</code>, a data center somewhere. The server has its own clock.</p>

<h3>
  
  
  3. Organization Time
</h3>

<p>The business context. A company headquartered in Vienna might define all deadlines in <code>Europe/Vienna</code>, regardless of where users or servers are.</p>

<p>When someone says "June 5th", which of these three do they mean?</p>

<p>Usually: <strong>nobody knows</strong>. And that's the problem.</p>




<h2>
  
  
  A Real Scenario
</h2>

<p>Let's trace through what happens with an ambiguous deadline:</p>

<ol>
<li>
<strong>Business requirement:</strong> "Users must submit by June 5th"</li>
<li>
<strong>Developer interprets:</strong> "End of day UTC" → stores <code>2026-06-05T23:59:59Z</code>
</li>
<li>
<strong>Frontend displays:</strong> "Due: June 5th" (no time, no timezone shown)</li>
<li>
<strong>User in New York:</strong> Sees "June 5th", submits at 11:00 PM on June 5th local time</li>
<li>
<strong>Server receives:</strong> The request at <code>2026-06-06T03:00:00Z</code> (New York is UTC-4 in June)</li>
<li>
<strong>Result:</strong> Rejected — the UTC deadline has already passed</li>
</ol>

<p>The user submitted on June 5th in their timezone. They did everything right from their perspective. But they failed because "June 5th" meant something different to the server.</p>

<p>Now imagine this is:</p>

<ul>
<li>A tax filing</li>
<li>A university application</li>
<li>A legal submission</li>
<li>A grant proposal</li>
</ul>

<p>The stakes are real. The ambiguity is unacceptable.</p>




<h2>
  
  
  "End of Day" Doesn't Help
</h2>

<p>Sometimes people try to fix this by saying "end of day June 5th" or "by end of business June 5th".</p>

<p>This doesn't solve anything:</p>

<ul>
<li>
<strong>End of day</strong> — whose day? What time? 23:59? 23:59:59? 23:59:59.999?</li>
<li>
<strong>End of business</strong> — whose business? What timezone? What if it's a holiday?</li>
</ul>

<p>You've just replaced one ambiguity with another.</p>




<h2>
  
  
  The Only Fix: Be Explicit
</h2>

<p>A technically correct deadline has three components:</p>

<ol>
<li>
<strong>Date</strong> — June 5th</li>
<li>
<strong>Time</strong> — 23:59 (or 17:00, or whatever makes sense)</li>
<li>
<strong>Timezone</strong> — Europe/Vienna (or the user's timezone, explicitly stated)</li>
</ol>

<p>Example of a clear deadline:</p>

<blockquote>
<p>"Submit by June 5th, 2026 at 23:59 Europe/Vienna"</p>
</blockquote>

<p>Now there's no ambiguity. Users in any timezone can convert this to their local time. The system can enforce it precisely. Everyone knows exactly when the deadline is.</p>




<h2>
  
  
  But What If We <em>Want</em> User-Local Deadlines?
</h2>

<p>Sometimes the business intent really is: "Each user has until the end of June 5th in their own timezone."</p>

<p>That's a valid requirement! But it's a <strong>different</strong> requirement — and it needs different handling:</p>

<ol>
<li>
<strong>Capture the user's timezone</strong> (or let them choose)</li>
<li>
<strong>Store the deadline as:</strong> Local Date + User's Timezone</li>
<li>
<strong>Evaluate per user:</strong> Convert their timezone's "end of June 5th" to an Instant, compare to now</li>
</ol>

<p>This is more complex, but it's honest. You're explicitly saying "the deadline is local to each user" rather than pretending a global deadline is local.</p>




<h2>
  
  
  What Systems Must Enforce
</h2>

<p>If you're building a system that handles deadlines, here's what you need:</p>

<h3>
  
  
  At Definition Time (when creating the deadline)
</h3>

<ul>
<li>Require a <strong>time</strong>, not just a date</li>
<li>Require a <strong>timezone</strong> (or default to a clearly documented one)</li>
<li>Reject incomplete definitions — don't guess</li>
</ul>

<h3>
  
  
  At Display Time (when showing the deadline)
</h3>

<ul>
<li>Show the <strong>full datetime with timezone</strong> at least once</li>
<li>Optionally show converted to user's local time ("June 5th 23:59 Vienna = June 6th 07:59 your time")</li>
</ul>

<h3>
  
  
  At Evaluation Time (when checking if deadline passed)
</h3>

<ul>
<li>Convert everything to Instant (UTC) for comparison</li>
<li>Use the <strong>stored timezone</strong>, not the server's timezone</li>
<li>Log the exact Instant of submission for disputes</li>
</ul>




<h2>
  
  
  When Developers Must Push Back
</h2>

<p>Sometimes you receive a requirement like "deadline is June 5th" and you know it's incomplete.</p>

<p>Push back. Ask:</p>

<ul>
<li>"What time on June 5th?"</li>
<li>"In which timezone?"</li>
<li>"Should users in other timezones get the same absolute deadline, or the same local experience?"</li>
</ul>

<p>If the business can't answer these questions, the deadline isn't defined yet — and you shouldn't build it.</p>

<p>Implementing an ambiguous deadline doesn't make you fast. It makes you responsible for the bugs.</p>




<h2>
  
  
  The Fairness Test
</h2>

<p>Here's a simple test for any deadline in your system:</p>

<blockquote>
<p>Can a user, acting in good faith, always determine whether they've met the deadline?</p>
</blockquote>

<p>If the answer is "no" — if a user could submit on what they believe is the correct day and still fail because of timezone confusion — the deadline is broken.</p>

<p>Fix the definition, not the user's expectations.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>"June 5th" is not a deadline — it's an incomplete specification</li>
<li>Every deadline needs: <strong>date + time + timezone</strong>
</li>
<li>There are (at least) three timezones in play: user, server, organization</li>
<li>"End of day" and "end of business" are just as ambiguous as bare dates</li>
<li>User-local deadlines are valid but need explicit handling</li>
<li>If users can't determine whether they've met the deadline, the deadline is broken — not the user</li>
</ul>




<p><em>Next up: <a href="https://dev.to/bwi/instant-vs-local-when-utc-helps-and-when-it-hurts-5d7p"><strong>Instant vs Local — When UTC Helps and When It Hurts</strong></a> — the core model for storing time correctly.</em></p>

