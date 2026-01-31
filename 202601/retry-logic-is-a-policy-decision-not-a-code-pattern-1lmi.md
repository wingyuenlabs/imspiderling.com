---
Title: Retry Logic Is a Policy Decision, Not a Code Pattern
Description: 
Author: SecEngineerX
Date: 2026-01-31T21:54:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>I used to think retry logic was an implementation detail.</p>

<p>It isn’t.</p>

<p>Retries encode assumptions about failure, time, trust, and responsibility. When those assumptions are wrong, systems don’t crash. They lie quietly.</p>

<p>This post isn’t about elegance. It’s about being explicit.</p>

<p><strong><em>The mistake people make</em></strong></p>

<p>Most retry implementations answer the wrong question.</p>

<p>They ask: “How do I try again?”</p>

<p>The real question is: “Under what failures am I allowed to try again?”</p>

<p>Those are not the same thing.</p>

<p><strong><em>Retries are not resilience by default</em></strong></p>

<p>Blind retries are comforting. They make engineers feel proactive.</p>

<p>In reality, they often:</p>

<ul>
<li>Mask real outages</li>
<li>Amplify load during partial failures</li>
<li>Destroy observability</li>
<li>Delay alerts until damage is done</li>
</ul>

<p>A retry without a failure model is just noise with a sleep call.</p>

<p><strong><em>What I learned building a monitoring primitive</em></strong></p>

<p>While building an async endpoint checker, I was forced to confront a few uncomfortable truths.</p>




<ol>
<li>Parameters are contracts</li>
</ol>




<p>If a function exposes a parameter that is not used, it is lying.</p>

<p>Dead parameters rot APIs. They create false confidence and future bugs. Removing them is not cleanup. It’s honesty.</p>




<ol>
<li>Catching Exception inside retries is negligence</li>
</ol>




<p>Retrying on all exceptions means retrying on:</p>

<ul>
<li>Programmer errors</li>
<li>Logic bugs</li>
<li>Invalid states</li>
</ul>

<p>Those failures should terminate execution immediately.</p>

<p>Retries are for expected, transient failures only. Anything else must fail fast.</p>




<ol>
<li>HTTP retries without backoff are hostile behavior</li>
</ol>




<p>Retrying immediately on 500s or 429s is not resilience.</p>

<p>It’s pressure.</p>

<p>If your system retries aggressively during degradation, it becomes part of the outage. Good retry logic reduces harm. Bad retry logic accelerates it.</p>




<ol>
<li>Time must have a single owner</li>
</ol>




<p>If multiple layers measure “total time”, metrics become contradictory.</p>

<p>Time is a resource, not a side effect.</p>

<p>Only one layer should own it. Everything else should report partial truth or nothing at all.</p>




<ol>
<li>Helpers should not know semantics</li>
</ol>




<p>A retry helper that understands HTTP status codes is doing too much.</p>

<p>Helpers should be stupid and obedient. Policy belongs to the caller.</p>

<p>When helpers start making decisions, architecture leaks.</p>

<p><strong><em>The most dangerous bug</em></strong></p>

<p>On the final retry, it’s tempting to overwrite the result:</p>

<ul>
<li>Force failure</li>
<li>Normalize fields</li>
<li>“Clean things up”</li>
</ul>

<p><strong><em>That destroys information.</em></strong></p>

<p>The last attempt is still truth. Corrupting it poisons analytics, alerting, and postmortems. These bugs don’t show up in logs. They show up in lost trust.</p>

<p><strong><em>Why this matters in monitoring systems</em></strong></p>

<p>Some failures justify retries. <br>
Others demand immediate alerts.<br>
Some should be recorded but not acted on.</p>

<p>If a monitoring system cannot explain why something failed, it cannot be trusted when it claims something is broken.</p>

<p>Closing thought<br>
Retry logic is not a loop. It’s a statement about how you believe the world behaves under stress.</p>

<p>If that statement is vague, your system will be vague when it matters most.</p>

<p><strong><em>Explicit beats clever. Every time.</em></strong></p>

