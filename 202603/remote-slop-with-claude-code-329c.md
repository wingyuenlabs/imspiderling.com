---
Title: Remote Slop with Claude Code
Description: 
Author: Mladen Stepanić
Date: 2026-03-20T21:42:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>In my <a href="https://dev.to/crawleyprint_71/workflow-engineering-prompt-engineering-3pep">last post on agentic workflows</a>, I talked about workflow engineering — how the pipeline you design around AI matters more than the AI itself. I built a book inventory app that way. Skills, OpenSpecs, beads, parallel agents in tmux. It worked great.</p>

<p>But there was an asterisk I didn't mention: I was sitting at my desk the whole time.</p>

<p>What happens when you're not?</p>

<h2>
  
  
  The Setup
</h2>

<p>Claude Code recently got a Telegram integration. The idea is simple: you control Claude Code from your phone through a Telegram bot. Same skills, same workflow, same project — different interface. If you've used the plugin system at all, setup is straightforward. Documentation walks you through it, you're chatting with your agent in minutes.</p>

<p>In fact, as of today, Anthropic officially shipped this as <a href="https://code.claude.com/docs/en/channels" rel="noopener noreferrer">Claude Code Channels</a> — a plugin-based feature that lets you push messages from <a href="https://telegram.org" rel="noopener noreferrer">Telegram</a> or <a href="https://discord.com" rel="noopener noreferrer">Discord</a> into a running Claude Code session on your machine. Your session processes the request with full filesystem, MCP, and git access, then replies through the same chat. It's built on MCP, which means it slots into the existing plugin ecosystem cleanly. I ran my experiment the day it launched, so what you're reading is a day-one field report.</p>

<p>I wanted to test this properly, so I gave it a real task. Not a toy. A refactoring sessions on an existing codebase, the kind of thing I'd normally spend a focused afternoon on. Except today, I wasn't at my desk and I wasn't focused. I was doing something else, checking Telegram between other things.</p>

<p>Buckle up.</p>

<h2>
  
  
  The Permission Wall
</h2>

<p>Immediately: a wall.</p>

<p>Claude Code has a permission system. It asks before it does anything potentially destructive — file writes, shell commands, external calls. At your desk, this is fine. You see the prompt, you approve, you move on.</p>

<p>From Telegram? The bot doesn't forward those permission prompts. Your agent hits a permission check, and it just... stops - silently. You're staring at Telegram wondering why nothing is happening, and the answer is that Claude is staring at your terminal wondering why you're not approving anything.</p>

<p>This is the first thing you'll hit, and there's no elegant workaround. Either you solve the permission problem or you don't use this workflow.</p>

<h2>
  
  
  The Context Ceiling
</h2>

<p>Second problem: context. Opus ships with a 1M token context window, which sounds like a lot. And it is — for a focused session. But "entire day, away from your machine, no way to reset" is a different budget. You can't <code>/clear</code> the session from Telegram. If the conversation gets heavy, you can't start fresh. You're stuck with whatever context you've accumulated.</p>

<p>For a day of casual back-and-forth this turned out to be manageable. But it's something you have to plan around, not something you can ignore.</p>

<h2>
  
  
  Biting the Bullet
</h2>

<p>So I did the thing you're not supposed to do: <code>--dangerously-skip-permissions</code>.</p>

<p>I know. The flag name exists for a reason. But here's my reasoning: I wasn't installing new external dependencies. My Claude Code workflow is already scoped — skills are loaded, the project is defined, the agent knows its boundaries.</p>

<p>And it worked. The permission wall disappeared. The agent could actually run.</p>

<p>Not great, not terrible. I'm still not comfortable disabling guardrails as a general practice. For this specific experiment, with this specific setup, it was a calculated risk. Use at your own discretion. Or better — don't. Anyway, don't blame me if Claude decides your <code>main</code> branch needs to have a different history and you don't have branch protection in place. You've been warned.</p>

<h2>
  
  
  Saving Context With Agent Teams
</h2>

<p>The context problem needed a different solution. If I can't clear the session, I need to use less of it.</p>

<p>This is where the workflow from my previous post paid off. If you haven't read it: I use a pipeline where work gets decomposed into <em>beads</em> — small, focused tasks based on Steve Yegge's <a href="https://github.com/steveyegge/beads" rel="noopener noreferrer">beads concept</a>. Each bead is scoped tightly enough that a sub-agent can pick it up and run with it without needing the full conversation history.</p>

<p>So I instructed the main agent to delegate aggressively. The pipeline looked like this:</p>

<ol>
<li>I describe what I want via Telegram</li>
<li>Main agent runs a spec creator sub-agent which generates an OpenSpec (a structured definition of the change) and makes a draft PR on GitHub</li>
<li>I review the spec from my phone and approve</li>
<li>Spec gets decomposed into beads</li>
<li>Sub-agents pick up beads, implement, commit, and update the PR </li>
</ol>

<p>The main session barely touched any implementation detail. It just coordinated. By the end of the day, I'd used about 30% of the context window. Granted, I wasn't going crazy with requests — but the pattern held up.</p>

<h2>
  
  
  Why Not Claude Code on the Web?
</h2>

<p>Fair question. Claude Code has a web interface now. I could've used that from my phone. No Telegram bot, no permission hacks.</p>

<p>The answer is boring: my tools. My skills are loaded locally. My workflow is configured. I have Playwright set up for visual verification — I literally had the agent screenshot pages to confirm layout changes actually landed. That's not something you get from the web interface.</p>

<p>When you've invested in a workshop, you want to use it. Even from a distance.</p>

<h2>
  
  
  The Slow Loop
</h2>

<p>Here's the real downside, and it's not about permissions or context.</p>

<p>The feedback loop is slow.</p>

<p>At my desk, I have hot reload. I change something, I see it instantly. From Telegram, the loop is: agent pushes to GitHub → Vercel builds a preview → I check the preview on my phone. That's minutes, not milliseconds. For layout work especially, it's painful. You're doing the development equivalent of texting someone in the next room instead of just talking to them.</p>

<p>I could live with it because I was mostly multitasking — checking in on the agent between other things. But if this were my primary way of working? The latency would get to me.</p>

<h2>
  
  
  The Numbers
</h2>

<p>I had Claude Code analyze the session after the fact and here's what a day of remote agent work actually looks like:</p>

<h3>
  
  
  Overview
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Total tool calls</td>
<td>507</td>
</tr>
<tr>
<td>Agents spawned</td>
<td>22 across 5 teams</td>
</tr>
<tr>
<td>PRs created / merged</td>
<td>5 / 5</td>
</tr>
<tr>
<td>Telegram messages</td>
<td>34</td>
</tr>
<tr>
<td>Playwright interactions</td>
<td>50</td>
</tr>
<tr>
<td>OpenSpec tasks verified</td>
<td>76</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Agents
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Role</th>
<th>Count</th>
</tr>
</thead>
<tbody>
<tr>
<td>QA</td>
<td>4</td>
</tr>
<tr>
<td>Spec Writer</td>
<td>3</td>
</tr>
<tr>
<td>Frontend</td>
<td>3</td>
</tr>
<tr>
<td>Explorer</td>
<td>3</td>
</tr>
<tr>
<td>Backend</td>
<td>1</td>
</tr>
<tr>
<td>Reviewers</td>
<td>3</td>
</tr>
<tr>
<td>Standalone</td>
<td>9</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Features
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td>Crop inset fix</td>
<td>merged</td>
</tr>
<tr>
<td>Remove impressum crop</td>
<td>merged</td>
</tr>
<tr>
<td>Scrollable mapping</td>
<td>merged</td>
</tr>
<tr>
<td>Reading status</td>
<td>in review</td>
</tr>
<tr>
<td>OpenSpec cleanup (3 lists)</td>
<td>76 tasks</td>
</tr>
</tbody>
</table></div>

<p>507 tool calls, 22 agents, 34 Telegram messages, 5 PRs. From my phone.</p>

<h2>
  
  
  The Verdict
</h2>

<p>At the end of the day, I sat down and did what I always do: reviewed the code myself. Read every change, checked every decision. And it was fine. A few minor optimizations I would've caught in real-time at my desk, but nothing structural. Nothing that made me regret the experiment.</p>

<p>So here's the honest scorecard:</p>

<p><strong>What works:</strong> Your full workflow, from Telegram. Skills, beads, agent teams, even Playwright screenshots. If you've built a good pipeline, it travels.</p>

<p><strong>What doesn't:</strong> Permission prompts don't reach you. Context can't be reset. The feedback loop trades seconds for minutes. And you'll probably end up running with <code>--dangerously-skip-permissions</code>, which is exactly as comfortable as it sounds.</p>

<p><strong>Who is this for:</strong> Someone who's already set up their Claude Code workflow and wants to keep things moving while away from their desk. Not as a primary dev environment — as an extension of one you already trust.</p>

<p>Would I do it again? Yeah, probably. But I'd plan the work differently. Bigger, well-defined refactorings that don't need rapid visual feedback. The kind of work where you can fire and forget, then review later. Not pixel-pushing. Not exploratory coding. Structured changes through a structured pipeline.</p>

<p>Remote slop? A little. But manageable slop, with a review step at the end. And sometimes that's enough.</p>

