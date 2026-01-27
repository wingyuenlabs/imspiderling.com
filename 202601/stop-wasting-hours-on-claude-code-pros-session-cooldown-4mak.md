---
Title: Stop wasting hours on Claude Code Pro's session cooldown
Description: 
Author: Sleeyax
Date: 2026-01-27T21:47:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you're a Claude Code Pro subscriber, you know the pain: you burn through your session at 3pm, and then you're locked out until 8pm. That's a 5-hour cooldown with zero usage. Half your workday, gone.</p>

<p>I built a Telegram bot to fix this.</p>

<h2>
  
  
  The problem
</h2>

<p>Claude Code Pro gives you a 5-hour session window. Once it expires, you wait 5 hours before you can start a new one. There's no way to queue up your next session in advance, so you either waste prime working hours waiting, or you carefully plan your usage around the clock.</p>

<p>Neither option is great.</p>

<h2>
  
  
  My solution: automated warmups
</h2>

<p><a href="https://github.com/sleeyax/claude-code-session-bot" rel="noopener noreferrer">Claude Code Session Bot</a> is a self-hosted Telegram bot that starts Claude Code sessions on a schedule. The idea is simple: start the session while you sleep, work out or are away from your keyboard so it's partially used up but still active when you need it.</p>

<p>Example usage scenario: schedule <code>/schedule tomorrow 9am</code> before bed. The bot calculates that it needs to warm up at 6am. By the time 9am rolls around, you still have 2 hours of usage left. That's enough for a productive morning. By 11am the session expires, the cooldown starts, and after lunch you have a fresh session ready to go.</p>

<h2>
  
  
  How it works
</h2>

<p>The bot runs on a VPS (or any always-on machine) with the Claude CLI installed and authenticated. When a scheduled warmup fires, it runs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>claude <span class="nt">-p</span> <span class="s2">"ready"</span> <span class="nt">--output-format</span> json
</code></pre>

</div>



<p>This sends a minimal prompt to Claude, which starts the 5-hour session timer. The bot records the session in a local SQLite database and tracks time remaining.</p>

<h3>
  
  
  Commands
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/warmup</code></td>
<td>Start a session immediately</td>
</tr>
<tr>
<td><code>/session</code></td>
<td>Check active session status (time left, expiry)</td>
</tr>
<tr>
<td><code>/schedule &lt;datetime&gt; [hours]</code></td>
<td>Schedule a warmup so you have <code>[hours]</code> remaining at <code>&lt;datetime&gt;</code>
</td>
</tr>
<tr>
<td><code>/schedules</code></td>
<td>List pending scheduled warmups</td>
</tr>
<tr>
<td><code>/cancel &lt;id&gt;</code></td>
<td>Cancel a scheduled warmup</td>
</tr>
<tr>
<td><code>/history</code></td>
<td>View recent session history</td>
</tr>
</tbody>
</table></div>

<p>See the <a href="https://github.com/sleeyax/claude-code-session-bot" rel="noopener noreferrer">GitHub repo</a> for full instructions.</p>

<h3>
  
  
  Scheduling examples
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/schedule tomorrow 9am        # 2h remaining at 9am (warmup fires at 6am)
/schedule monday 14:00 3      # 3h remaining at 14:00 (warmup fires at 12:00)
/schedule jan 30 8:00 4h      # 4h remaining at 8:00 (warmup fires at 7:00)
</code></pre>

</div>



<p>The math behind this is simple: <code>warmup_time = target_time - (5h - desired_hours_remaining)</code>.</p>

<p>Date parsing is handled by <a href="https://github.com/wanasit/chrono" rel="noopener noreferrer">chrono-node</a>, so natural language like "tomorrow 9am" or "next monday 14:00" just works.</p>

<h2>
  
  
  Architecture
</h2>

<p>The stack is intentionally minimal:</p>

<ul>
<li>
<strong>TypeScript</strong> with strict mode</li>
<li>
<strong>SQLite</strong> (via better-sqlite3, WAL mode) for persistence</li>
<li>
<strong>node-telegram-bot-api</strong> for the Telegram interface</li>
<li>
<strong>In-memory <code>setTimeout</code></strong> timers restored from DB on restart</li>
</ul>

<p>No external services, no Redis, no message queues. Schedules survive restarts because they're persisted to SQLite and restored when the bot boots.</p>

<h2>
  
  
  Limitations
</h2>

<p>The bot only tracks sessions <strong>it</strong> starts. There's no Anthropic API to query your current session status, and reverse-engineering internal endpoints risks account bans. So if you start a session manually from your dev machine, the bot won't know about it.</p>

<p>For best results, use the bot as your sole session starter from a dedicated VPS.</p>

<h2>
  
  
  Final thoughts
</h2>

<p>This is a simple tool for a specific annoyance. If you're a Claude Code Pro user who's tired of session cooldowns eating into your productive hours, give it a try.</p>

<p>The project is <a href="https://github.com/sleeyax/claude-code-session-bot" rel="noopener noreferrer">open source</a> (MIT license) - contributions and feedback welcome.</p>

