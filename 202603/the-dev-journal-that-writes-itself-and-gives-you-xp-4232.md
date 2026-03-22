---
Title: The Dev Journal That Writes Itself (And Gives You XP)
Description: 
Author: austin amento
Date: 2026-03-22T21:40:50.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/notion-2026-03-04">Notion MCP Challenge</a></em></p>




<p>Every developer has a graveyard of half-remembered side projects.</p>

<p>You know the feeling. Someone asks "what have you been building?" during a job search and you blank. Or you're writing a performance review and can't reconstruct what you shipped in Q1. Or you're three months into learning to code and can't tell if you're getting better, which honestly stings more than the other two.</p>

<p>The problem isn't that you aren't building. It's that the work disappears into git history the moment it's committed.</p>

<p>I built <strong>DevPulse</strong> to fix that. And then I turned your git history into a game.</p>




<h2>
  
  
  What I Built
</h2>

<p><strong>DevPulse</strong> is a global <a href="https://claude.ai/code" rel="noopener noreferrer">Claude Code</a> agent that automatically logs your coding sessions to Notion after every commit. It tracks your growth with XP and levels, maintains daily streaks, runs a 25-quest achievement system, and unlocks 30 achievement badges — all triggered by a single slash command, from any project on your machine.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🎮 Async Queue System for Ad Insertion Engine

⚡ 100 base × 1.5 (7-day streak) × 1.25 (diff run) + 10 (daily spark) + 50 (Ship It quest) = 247 XP
📊 Total: 1,847 XP

🏆 Level: Senior Dev → Staff Engineer 🎉

🔥 Streak: 8 days

🏅 Badges Unlocked:
  - Week Warrior ⚡ (7-day streak!)
  - Challenge Accepted ⚔️

⚔️ Quest Completed:
  - Challenge Accepted (50 XP) — 2 sessions this week at Difficulty 4+

• Built async-safe queue with retry logic and exponential backoff — no dropped frames under load
• Logged to Master Dev Log · Profile updated: 14 sessions, 1,847 XP, Staff Engineer
• Quests DB updated: Challenge Accepted → Completed
</code></pre>

</div>



<p>No server. No backend to maintain. No manual journaling. You committed your code — DevPulse reads the diff and handles everything else.</p>

<h3>
  
  
  Who It's Actually For
</h3>

<p>I designed this for two people simultaneously, because I think they have the same core problem.</p>

<p><strong>If you're learning to code:</strong> DevPulse makes the invisible visible. Every session logged is proof of work — proof that you showed up and built something. The difficulty ratings go from 1 (config tweaks) to 5 (complex algorithms). You watch them climb as you grow. Your first month might be all 2s and 3s. Six months later you're cracking 4s and 5s and the XP reflects it. The streak system is the most important feature here: building the habit of shipping daily, even small things, is what separates developers who grow fast from those who plateau. And when you can't articulate what you've been working on? Pull up your weekly recap.</p>

<p><strong>If you're an experienced developer:</strong> You already know how fast projects blur together. DevPulse gives you a structured, searchable record across every project on your machine — filtered by stack, sorted by difficulty, summarized weekly. The weekly recap generates a <strong>Growth Narrative</strong> written from your actual data, not generic encouragement. It's also genuinely useful for performance reviews, resume updates, and the "so what have you been working on?" questions you get in every technical conversation.</p>

<h3>
  
  
  The Growth Narrative
</h3>

<p>This is the feature I'm most proud of. Every <code>/weekly-recap</code> generates a Claude-written analysis of your actual week — specific to your data:</p>

<blockquote>
<p><em>"Backend-heavy week — 4 of 5 sessions touched the database layer and your average difficulty climbed to 3.8, up from 2.6 last week. TypeScript was your most-used technology for the third week running. You logged sessions on 5 different days, your most consistent week yet. Next week: your streak hits 14 days — push for Streak Legend."</em></p>
</blockquote>

<p>That's not templated. That's Claude reading your Master Dev Log, spotting the pattern, and writing it back to you. It's a mirror that shows you your own growth in a way raw git logs never could.</p>

<h3>
  
  
  The Architecture
</h3>

<p>The system runs as a <strong>global Claude Code configuration</strong> — installed once to <code>~/.claude/</code>, active in every project on your machine:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>~/.claude/
├── CLAUDE.md                    ← DevPulse identity + XP formula + all 30 badge definitions
└── commands/
    ├── pulse-check.md           ← /pulse-check workflow <span class="o">(</span>11 sequential steps<span class="o">)</span>
    ├── weekly-recap.md          ← /weekly-recap workflow
    └── monthly-recap.md         ← /monthly-recap workflow
</code></pre>

</div>



<p>When you type <code>/pulse-check</code>, Claude loads the agent identity from <code>CLAUDE.md</code> and executes the 11-step workflow in <code>pulse-check.md</code>. Every operation that touches persistent state goes through Notion MCP — there is no other storage layer.</p>

<h3>
  
  
  The Four Notion Databases
</h3>

<p>DevPulse manages four databases entirely through MCP:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Database</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Master Dev Log</strong></td>
<td>One row per session — title, stack, difficulty, XP, commit hash, badges</td>
</tr>
<tr>
<td><strong>DevPulse Profile</strong></td>
<td>Single-row state machine: streak, total XP, level, all badges earned</td>
</tr>
<tr>
<td><strong>DevPulse Quests</strong></td>
<td>25 quests (13 weekly rotating + 12 permanent milestones)</td>
</tr>
<tr>
<td><strong>DevPulse Recaps</strong></td>
<td>Weekly and monthly recap pages with tech breakdown + growth narrative</td>
</tr>
</tbody>
</table></div>

<p>Three of these are <strong>auto-created on first run</strong>. The one you set up manually (Master Dev Log) is the source of truth — DevPulse never deletes or modifies rows in it after they're written.</p>

<h3>
  
  
  The Full Gamification Engine (V2)
</h3>

<p>I started with V1 (basic logging + 7 badges) and iterated to a full gamification system.</p>

<p><strong>XP Multiplier Formula:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight mathematica"><code><span class="nv">Session</span><span class="w"> </span><span class="nv">XP</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="nv">floor</span><span class="p">((</span><span class="nv">Difficulty</span><span class="w"> </span><span class="err">×</span><span class="w"> </span><span class="m">20</span><span class="p">)</span><span class="w"> </span><span class="err">×</span><span class="w"> </span><span class="nv">streak</span><span class="o">_</span><span class="nv">mult</span><span class="w"> </span><span class="err">×</span><span class="w"> </span><span class="nv">diff</span><span class="o">_</span><span class="nv">run</span><span class="o">_</span><span class="nv">mult</span><span class="p">)</span><span class="w"> </span><span class="o">+</span><span class="w"> </span><span class="nv">daily</span><span class="o">_</span><span class="nv">spark</span><span class="w"> </span><span class="o">+</span><span class="w"> </span><span class="nv">quest</span><span class="o">_</span><span class="nv">bonus</span><span class="w">
</span></code></pre>

</div>



<ul>
<li>
<code>streak_mult</code>: 1.5× at 7-day streak, 2× at 30 days — consistency is rewarded</li>
<li>
<code>diff_run_mult</code>: 1.25× when two back-to-back sessions hit Difficulty ≥ 4 — keeps you pushing hard</li>
<li>
<code>daily_spark</code>: flat +10 XP on your first session of the day</li>
<li>
<code>quest_bonus</code>: bonus XP from any quests completed this session</li>
<li>Combined multiplier cap: 3.0× — no runaway math</li>
</ul>

<p><strong>25-Quest System:</strong></p>

<ul>
<li>13 <strong>weekly quests</strong> rotate every Monday: Ship It, Deep Work, Consistency Check, Stack Master, Challenge Accepted, Bug Hunter, Early Bird, Polyglot Week, and more</li>
<li>12 <strong>permanent milestone quests</strong>: Century Club, Perfectionist, Streak Legend, Tech Specialist, Versatile, and more</li>
<li>Quest completion writes to the Quests DB and adds XP to the current session's total</li>
</ul>

<p><strong>30 Achievement Badges</strong> across three groups — from First Flame (session 1) through Diamond Coder (30-day streak), Deep Diver (three Difficulty-5 sessions in 7 days), Full Stack (frontend + backend in one session), and Achievement Hunter (15+ badges earned).</p>




<h2>
  
  
  Show Us the Code
</h2>

<p>🔗 <strong>GitHub:</strong> <a href="https://github.com/puginator/devlog" rel="noopener noreferrer">github.com/puginator/devlog</a></p>

<p>The distributable config lives in <code>claude-config/</code>. Here's what makes the system tick:</p>

<h3>
  
  
  <code>claude-config/CLAUDE.md</code> — The Agent Identity
</h3>

<p>This file ships the XP formula, difficulty rubric, level thresholds, and all 30 badge definitions into Claude's context on every session. The badge definitions are the most interesting part — they reference in-memory values computed earlier in the same session:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Badge Definitions</span>
Badge checks use post-Step-8 streak value, Total Sessions = Profile Total Sessions + 1,
and provisional Total XP.

| Label | Condition |
|---|---|
| Multiplier | Session where streak_mult ≥ 1.5 AND diff_run_mult = 1.25 AND daily_spark was applied |
| Deep Diver | 3+ sessions with Difficulty = 5 in any rolling 7-day window |
| Full Stack | Session Stack contains ≥ 1 frontend AND ≥ 1 backend technology |
| Achievement Hunter | 15+ badges in Profile Badges Earned |
</code></pre>

</div>



<p>The hard constraints section is what makes the system reliable across sessions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Hard Constraints</span>
<span class="p">-</span> Never hallucinate Notion properties — stop and report errors exactly
<span class="p">-</span> Never log empty sessions (no meaningful changes = decline and explain)
<span class="p">-</span> Badge checks are idempotent — compare against Badges Earned in Profile before announcing
<span class="p">-</span> Quest evaluation is idempotent — check Completed On before awarding quest XP; if set, skip
<span class="p">-</span> Stack always written as an array of strings — never a comma-separated string
<span class="p">-</span> Difficulty run query always executes before Step 6 to read the previous session only
</code></pre>

</div>



<h3>
  
  
  <code>claude-config/commands/pulse-check.md</code> — The Core Workflow
</h3>

<p>11 steps executed in sequence. A few highlights:</p>

<p><strong>Step 1b — Duplicate check (prevents double-logging the same commit):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>If the commit hash is NOT "uncommitted": search Master Dev Log for an existing row
where Commit Hash matches this hash. If found: stop and say "Already logged commit
[hash] — make a new commit before running Pulse Check again!"
</code></pre>

</div>



<p><strong>Step 4b — XP multiplier computation (pure in-memory, no Notion reads):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>streak_mult:
<span class="p">-</span> 7–29 days → 1.5
<span class="p">-</span> 30+ days → 2.0
<span class="p">-</span> Otherwise → 1.0

diff_run_mult:
Query most recent Master Dev Log row (sort Date descending, limit 1). Read Difficulty.
<span class="p">-</span> If previous Difficulty ≥ 4 AND current Difficulty ≥ 4 → 1.25
<span class="p">-</span> Otherwise → 1.0

combined_mult = min(streak_mult × diff_run_mult, 3.0)
daily_spark = +10 XP if Last Active Date ≠ today
provisional_xp = floor((Difficulty × 20) × combined_mult) + daily_spark_xp
</code></pre>

</div>



<p><strong>Step 9b — Quest evaluation (runs after the session row is written):</strong></p>

<p>Each active quest has a specific condition evaluated against the session and Master Dev Log:</p>

<ul>
<li>
<code>Ship It (50 XP)</code>: 3+ sessions logged this ISO week</li>
<li>
<code>Deep Work (50 XP)</code>: current session Difficulty = 5</li>
<li>
<code>Challenge Accepted (50 XP)</code>: 2+ sessions this week at Difficulty ≥ 4</li>
<li>
<code>Bug Hunter (50 XP)</code>: 2+ sessions this week where Commit Message contains "fix" or "bug"</li>
<li>
<code>Consistency Check (50 XP)</code>: sessions logged on 3+ different days this week</li>
</ul>

<p>Completed quests update in the Quests DB, add to <code>quest_bonus</code>, and the session row's <code>XP Earned</code> is patched with the final total.</p>

<p><strong>Step 6 — Automatic V1→V2 schema migration:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>Before writing the row, verify Master Dev Log has the <span class="sb">`XP Earned`</span> (Number) and
<span class="sb">`Commit Message`</span> (Rich Text) properties. If either is missing, add it using
notion_update_database before proceeding.
</code></pre>

</div>



<p>Zero manual Notion work to upgrade. DevPulse detects missing columns and adds them on the first V2 run.</p>

<h3>
  
  
  <code>claude-config/commands/weekly-recap.md</code> — The Recap Generator
</h3>

<p>The weekly recap queries 7 days of Master Dev Log rows and writes a Notion page. The Growth Narrative step is where it gets interesting:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Step 6 — Growth Narrative</span>
Write a 3–4 sentence analysis of the week. Must reference actual data (most-used stack,
difficulty trend, project focus, streak status). Must include one specific pattern
observation (e.g. "Backend-heavy week — 3 of 5 sessions touched the database layer").
Must end with a forward-looking callout for next week.
</code></pre>

</div>



<p>Claude isn't filling in a template — it's reading your actual session data, identifying the pattern, and writing it back as analysis.</p>




<h2>
  
  
  How I Used Notion MCP
</h2>

<p>Notion MCP is the entire backend of DevPulse. Every piece of state lives in Notion. Claude reads and writes through <code>@notionhq/notion-mcp-server</code> — there's no local database, no config file, no server of any kind.</p>

<p>Here's every MCP operation the system uses:</p>

<h3>
  
  
  <code>notion_search</code> — Database discovery
</h3>

<p>Every workflow starts by finding its databases by name:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">notion_search</span><span class="p">(</span><span class="sh">"</span><span class="s">Master Dev Log</span><span class="sh">"</span><span class="p">)</span>     <span class="err">→</span> <span class="n">session</span> <span class="n">logging</span>
<span class="nf">notion_search</span><span class="p">(</span><span class="sh">"</span><span class="s">DevPulse Profile</span><span class="sh">"</span><span class="p">)</span>   <span class="err">→</span> <span class="n">XP</span><span class="p">,</span> <span class="n">streak</span><span class="p">,</span> <span class="n">badges</span>
<span class="nf">notion_search</span><span class="p">(</span><span class="sh">"</span><span class="s">DevPulse Quests</span><span class="sh">"</span><span class="p">)</span>    <span class="err">→</span> <span class="n">active</span> <span class="n">quests</span>
<span class="nf">notion_search</span><span class="p">(</span><span class="sh">"</span><span class="s">DevPulse Recaps</span><span class="sh">"</span><span class="p">)</span>    <span class="err">→</span> <span class="n">weekly</span><span class="o">/</span><span class="n">monthly</span> <span class="n">summaries</span>
</code></pre>

</div>



<p>This is what makes DevPulse portable. The databases can live anywhere in your Notion workspace.</p>

<h3>
  
  
  <code>notion_query_database</code> — Precise filtered reads
</h3>

<p>The quest system and badge checks require querying Master Dev Log with specific filters:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Sessions this ISO week (quest evaluation)</span>
<span class="na">filter</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">Date</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">on_or_after</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2026-03-16"</span><span class="pi">,</span> <span class="nv">on_or_before</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2026-03-22"</span> <span class="pi">}</span> <span class="pi">}</span>

<span class="c1"># Previous session only (diff_run_mult)</span>
<span class="na">sorts</span><span class="pi">:</span> <span class="pi">[{</span> <span class="nv">property</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Date"</span><span class="pi">,</span> <span class="nv">direction</span><span class="pi">:</span> <span class="s2">"</span><span class="s">descending"</span> <span class="pi">}]</span><span class="err">,</span> <span class="na">page_size</span><span class="pi">:</span> <span class="m">1</span>

<span class="c1"># All Difficulty-5 sessions (Deep Diver badge, Hardcore badge)</span>
<span class="na">filter</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">Difficulty</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">equals</span><span class="pi">:</span> <span class="nv">5</span> <span class="pi">}</span> <span class="pi">}</span>

<span class="c1"># Active quests only</span>
<span class="na">filter</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">Status</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">equals</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Active"</span> <span class="pi">}</span> <span class="pi">}</span>
</code></pre>

</div>



<h3>
  
  
  <code>notion_create_page</code> — Writing session data
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">notion_create_page</span><span class="p">(</span>
  <span class="nx">parent</span><span class="p">:</span> <span class="p">{</span> <span class="nl">database_id</span><span class="p">:</span> <span class="nx">master_dev_log_id</span> <span class="p">},</span>
  <span class="nx">properties</span><span class="p">:</span> <span class="p">{</span>
    <span class="dl">"</span><span class="s2">Session Title</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Async Queue System for Ad Insertion Engine</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">Project</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">ad-engine</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">Stack</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">TypeScript</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Node.js</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Redis</span><span class="dl">"</span><span class="p">],</span>  <span class="err">←</span> <span class="nx">always</span> <span class="nx">an</span> <span class="nx">array</span><span class="p">,</span> <span class="nx">never</span> <span class="nx">a</span> <span class="nx">string</span>
    <span class="dl">"</span><span class="s2">Difficulty</span><span class="dl">"</span><span class="p">:</span> <span class="mi">4</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">XP Earned</span><span class="dl">"</span><span class="p">:</span> <span class="mi">247</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">Date</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">2026-03-19</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">Commit Hash</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">a3f9b2c</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">Commit Message</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">feat: implement async ad queue with retry logic</span><span class="dl">"</span>
  <span class="p">}</span>
<span class="p">)</span>
</code></pre>

</div>



<p><code>Stack</code> is always written as an array of strings — a hard constraint in <code>CLAUDE.md</code> because early versions wrote <code>"TypeScript, React"</code> as a single value, which broke the tech frequency aggregation in weekly recaps.</p>

<h3>
  
  
  <code>notion_update_page</code> — State management
</h3>

<p>After writing the session, DevPulse updates the Profile and patches the session row if quests completed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="err">#</span> <span class="nx">Profile</span> <span class="nx">update</span>
<span class="nf">notion_update_page</span><span class="p">(</span><span class="nx">profile_row_id</span><span class="p">,</span> <span class="p">{</span>
  <span class="dl">"</span><span class="s2">Current Streak</span><span class="dl">"</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">Total XP</span><span class="dl">"</span><span class="p">:</span> <span class="mi">1847</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">Current Level</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Staff Engineer</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">Badges Earned</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">First Flame</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Bug Slayer</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Week Warrior</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Challenge Accepted</span><span class="dl">"</span><span class="p">],</span>
  <span class="dl">"</span><span class="s2">Last Active Date</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">2026-03-19</span><span class="dl">"</span>
<span class="p">})</span>

<span class="err">#</span> <span class="nx">Session</span> <span class="nx">row</span> <span class="nx">patched</span> <span class="kd">with</span> <span class="nx">final</span> <span class="nx">XP</span> <span class="nx">after</span> <span class="nx">quest</span> <span class="nx">bonuses</span> <span class="nx">resolved</span>
<span class="nf">notion_update_page</span><span class="p">(</span><span class="nx">session_row_id</span><span class="p">,</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">XP Earned</span><span class="dl">"</span><span class="p">:</span> <span class="mi">247</span> <span class="p">})</span>  <span class="err">←</span> <span class="nx">was</span> <span class="mi">187</span> <span class="nx">before</span> <span class="nx">quest</span> <span class="nx">bonus</span>

<span class="err">#</span> <span class="nx">Quest</span> <span class="nx">marked</span> <span class="nx">complete</span>
<span class="nf">notion_update_page</span><span class="p">(</span><span class="nx">quest_row_id</span><span class="p">,</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">Status</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Completed</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Completed On</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">2026-03-19</span><span class="dl">"</span> <span class="p">})</span>
</code></pre>

</div>



<h3>
  
  
  <code>notion_update_database</code> — Schema migration
</h3>

<p>V1→V2 upgrade adds missing columns automatically:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">notion_update_database</span><span class="p">(</span><span class="nx">master_dev_log_id</span><span class="p">,</span> <span class="p">{</span>
  <span class="dl">"</span><span class="s2">XP Earned</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span> <span class="na">number</span><span class="p">:</span> <span class="p">{}</span> <span class="p">},</span>
  <span class="dl">"</span><span class="s2">Commit Message</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span> <span class="na">rich_text</span><span class="p">:</span> <span class="p">{}</span> <span class="p">}</span>
<span class="p">})</span>
</code></pre>

</div>



<p>Runs once, on the first V2 <code>/pulse-check</code>, only if the columns don't exist.</p>

<h3>
  
  
  Auto-creating databases on first run
</h3>

<p>If DevPulse Profile or DevPulse Quests don't exist, Claude creates them with the full schema and seeds all 25 quests in a single batch — 12 milestone quests as Active rows, plus 3 weekly quests timed to the current week's deadline.</p>




<h3>
  
  
  Why Notion MCP Was the Right Choice
</h3>

<p>DevPulse works because Notion is actually a real relational database, not just a note-taking app. Filtering by date, sorting by difficulty, updating specific properties, running schema migrations — all the things you'd normally need a backend for are just Notion API calls.</p>

<p>@notionhq/notion-mcp-server gives Claude direct access to every Notion operation from inside the conversation context. No server needed. The entire DevPulse backend is: Claude + a markdown config file + MCP.</p>

<p>The MCP interface is the key piece. Without it, I'd need a server to broker requests between Claude and the Notion API. With <code>@notionhq/notion-mcp-server</code>, Claude has direct access to every Notion operation from inside the conversation context. The entire DevPulse backend is: Claude + a markdown config file + MCP. That's it.</p>




<p>DevPulse is open source and works for anyone with Claude Code. Clone the repo, run <code>setup.sh</code>, and your first <code>/pulse-check</code> sets everything up automatically — including writing your first entry to Notion and starting your streak at day 1.</p>

<p>If you're just starting to code, that first entry is going to mean more than you think it will six months from now.</p>

