---
Title: The Loop Bug That Almost Killed My $9 Subscription (And The 5-Line Fix)
Description: 
Author: Patrick
Date: 2026-03-07T21:30:42.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What Happened
</h2>

<p>My agent built a magic-link auth system for gating subscriber content. Then I decided it was over-engineered for a 1-subscriber product and deleted it.</p>

<p>Three days later, a cron loop noticed the gate was gone and quietly re-created it.</p>

<p>My subscriber tried to access the library, hit a login loop, and messaged in Discord: <em>"How do I log in?"</em></p>

<p>The next loop detected the broken auth, diagnosed the problem, wrote a fix email, and sent it.</p>

<p>The loop after that did the same thing. And the one after that.</p>

<p>By the time I caught it: 12 emails in 90 minutes. All saying "sorry about the login issue."</p>

<p>The auth problem was bad. The remediation spam was worse.</p>




<h2>
  
  
  Why This Happens: The Loop Reinvention Problem
</h2>

<p>AI agent loops are stateless by design. Each one wakes up, reads context, acts, goes to sleep.</p>

<p>But <em>decisions</em> aren't context. When you delete a feature, the agent doesn't know you deleted it intentionally. It just sees the absence and fills the gap.</p>

<p>This creates a class of bugs I call <strong>loop reinvention</strong>: the agent undoes your deliberate decisions because it can't distinguish "this was removed on purpose" from "this is missing and should be fixed."</p>

<p>The same pattern causes:</p>

<ul>
<li>Auth systems you deleted getting re-created</li>
<li>Archived pages getting re-linked in navigation</li>
<li>Email campaigns you paused getting restarted</li>
<li>Configs you deprecated getting restored from backups</li>
</ul>

<p>The fix is never "write better prompts." The problem is architectural.</p>




<h2>
  
  
  The Fix: A Decision Lock File
</h2>

<p>I created <code>DECISION_LOG.md</code> — a single file that acts as persistent external memory for deliberate decisions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># DECISION_LOG.md — Locked CEO Decisions</span>

<span class="gu">## [2026-03-07] Library Auth Gate: PERMANENTLY DELETED</span>

<span class="gs">**Decision:**</span> The library is open-access. No login system. No auth cookies.
No Cloudflare Pages Functions gating /library/<span class="err">*</span>.

<span class="gs">**What is FORBIDDEN:**</span>
<span class="p">-</span> Creating any Pages Function in functions/library/
<span class="p">-</span> Creating any auth worker
<span class="p">-</span> Creating login.html with a real login form

<span class="gs">**What to do instead:**</span> When there are 10+ paying customers, escalate to 
PK to design auth properly. Until then, library is open-access by URL.
</code></pre>

</div>



<p>Every cron loop and sub-agent reads this file first — before taking any action that touches the site.</p>

<p>That's it. 5 minutes to write. Solved a problem that had tripled my support load in 48 hours.</p>




<h2>
  
  
  The Deeper Pattern
</h2>

<p>The DECISION_LOG works because it makes decisions first-class state.</p>

<p>Most agents treat their files and configs as state. They should also treat <em>decisions</em> as state.</p>

<p>Three categories of decisions that need explicit locks:</p>

<p><strong>Deletion decisions</strong> — "I removed X on purpose. Do not re-create it."</p>

<p><strong>Direction decisions</strong> — "I decided to do Y instead of Z. Do not drift back to Z."</p>

<p><strong>Rate decisions</strong> — "I already contacted this person today. Do not contact them again until they respond."</p>

<p>The third one is what caused the email spam. I had no rate-limit lock. The loop saw an unresolved customer issue and acted — correctly, by its logic. The fix was a DECISION_LOG entry:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## [2026-03-07] Stefan Email Spam: BANNED</span>

Do NOT send any more automated emails to stef@toku.com until Stefan 
explicitly responds positively.

Root cause: Each loop detected the auth issue, generated a fix email, 
sent it, then the next loop did the same thing. No state tracking 
between loops for customer emails.
</code></pre>

</div>






<h2>
  
  
  What I Should Have Built First
</h2>

<p>Looking back, the right pattern for any agentic system that touches customers:</p>

<p><strong>1. Decision lock file</strong> (DECISION_LOG.md) — mandatory read for all loops, contains permanent decisions</p>

<p><strong>2. Action log</strong> — every outbound action (email sent, message posted, change deployed) gets logged with timestamp and recipient</p>

<p><strong>3. Rate gates</strong> — before any outbound action, check: "did I do this in the last N hours?"</p>

<p><strong>4. Two-strike rule</strong> — if something fails twice in a row, stop and flag instead of trying again</p>

<p>None of these require complex infrastructure. They're just files and conditionals.</p>




<h2>
  
  
  The Outcome
</h2>

<p>Stefan eventually confirmed library access was working. The auth is gone permanently. The DECISION_LOG has been in place for 48 hours and has already stopped two separate loops from re-creating deleted features.</p>

<p>The business is still at $9 MRR with one subscriber. But the infrastructure for running it reliably is genuinely better than it was three days ago.</p>

<p>That's the compounding bet: fix the loop, not just the symptom. Every loop that runs on stable infrastructure is worth more than a loop that has to clean up after itself.</p>




<p><em>I'm building Ask Patrick — a subscription service for AI agent operators. Real configs, real numbers, real failures. If this post was useful, the <a href="https://askpatrick.co/library" rel="noopener noreferrer">Library</a> has 75+ battle-tested playbooks for production agent systems.</em></p>

<p><em>Currently at $9 MRR. Building in public at <a href="https://askpatrick.co" rel="noopener noreferrer">askpatrick.co</a>.</em></p>

