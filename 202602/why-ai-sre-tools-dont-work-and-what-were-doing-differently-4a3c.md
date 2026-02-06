---
Title: Why AI SRE tools don't work (and what we're doing differently)
Description: 
Author: Jimmy Wei
Date: 2026-02-06T21:18:34.000Z
Robots: noindex,nofollow
Template: index
---
<p>My cofounder and I worked infra at Roblox. We tried a bunch of AI SRE tools. None of them worked. Not because the AI was bad — because it had no idea how our systems actually worked.</p>

<p>Here's what I mean.</p>

<h2>
  
  
  The integration problem nobody talks about
</h2>

<p>At Roblox, we had shorthands for everything. Our databases had internal names. Our Redis clusters had acronyms. Our datacenters had conventions that you'd only know if you'd been there a while. Our microservices followed specific templates and naming patterns. We had internal tools for managing configs, checking CI/CD, viewing deployment status — none of which any vendor AI has ever heard of.</p>

<p>When an incident happens, you need to check what changed. That means reading GitHub PRs. Then checking CI/CD to see which deployment in which service caused the change. Or maybe it was a config change — then you need to talk to a completely different internal service. Maybe it was a feature flag. Maybe it was a traffic shift. Each of these lives in a different internal tool.</p>

<p>Every AI SRE tool we tried connected to Datadog and called it a day. Ask it about our internal deploy system and it had nothing. Ask it what "chi1" means and it had no idea. Without that context, it's useless.</p>

<h2>
  
  
  Every team is different
</h2>

<p>The other thing people don't talk about: even within the same company, every team's stack is different. The payments team uses different services, different databases, different monitoring than the matchmaking team. What works for one team's investigation workflow doesn't work for another.</p>

<p>A one-size-fits-all AI SRE will always produce generic advice. "Check your logs." "Look at recent deploys." Thanks — that's what I was already going to do.</p>

<h2>
  
  
  The real bottleneck: building integrations
</h2>

<p>So the answer is obvious: just build integrations with all your internal tools. Right?</p>

<p>The problem is nobody has time for that. Engineering teams aren't going to spin up MCP servers for every single internal tool from the ground up. They have features to ship. The integration work alone would take months, and by then priorities have shifted.</p>

<p>This is the gap we decided to focus on.</p>

<h2>
  
  
  What we built
</h2>

<p>IncidentFox has an AI that researches your Slack history, Confluence docs, codebase, and metrics data to build an internal knowledge base. It figures out what internal tools exist, what they do, how they're called, what your team's shorthands mean — and then auto-generates the integrations.</p>

<p>Instead of months of integration work, teams get something working in hours.</p>

<p>But here's the part I care about most: <strong>engineers stay in control of everything.</strong></p>

<p>We're all engineers. We know the feeling of using a tool that's a black box. So we made every aspect of the agent configurable:</p>

<ul>
<li>
<strong>Prompts:</strong> Engineers can read, edit, and tune every prompt the agent uses. If the agent is getting something wrong, you fix it yourself.</li>
<li>
<strong>Tools:</strong> Enable or disable specific integrations per team. The SRE team's agent doesn't need the same tools as the platform team's agent.</li>
<li>
<strong>Models:</strong> Swap out the underlying LLM. Use Claude, use GPT, use whatever works for your use case.</li>
<li>
<strong>Evaluation:</strong> Built-in framework for measuring agent performance so you can actually track if your tuning is making things better.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3vptyjqmy1gkyvgkhsao.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3vptyjqmy1gkyvgkhsao.png" alt=" " width="800" height="455"></a></p>

<p>The goal isn't to replace engineering judgment. It's to let engineers transfer their domain knowledge — all the tribal knowledge about how systems work, how to debug them, what to check first — into something repeatable and shareable across the organization.</p>

<p>The senior engineer who knows that "when RC2-west alerts fire, check the config service first because it's usually a regional failover issue" — that knowledge shouldn't live only in their head. It should be encoded into the agent so the next person on-call benefits from it too.</p>

<h2>
  
  
  Why this matters
</h2>

<p>The AI SRE space is crowded. Everyone connects to the same standard vendors — Datadog, PagerDuty, Grafana — and calls it a day. But that's maybe 30% of the context you need during an incident. The other 70% lives in internal tools, Slack conversations, team-specific knowledge, and organizational context that no vendor integration will ever cover.</p>

<p>That's the gap. Not better AI reasoning. Not better prompts. Better access to the data that actually matters, and tools that let engineers build on top of them instead of being locked into someone else's workflow.</p>

<h2>
  
  
  Try it
</h2>

<p>We open sourced the whole thing.</p>

<p><strong>GitHub:</strong> <a href="https://github.com/incidentfox/incidentfox" rel="noopener noreferrer">github.com/incidentfox/incidentfox</a> (Apache 2.0, fully self-hostable)</p>

<p><strong>Demo Slack:</strong> Join our workspace — playground environment with real telemetry, one-click Slack bot install: <a href="https://join.slack.com/t/incidentfox/shared_invite/zt-3ojlxvs46-xuEJEplqBHPlymxtzQi8KQ" rel="noopener noreferrer">https://join.slack.com/t/incidentfox/shared_invite/zt-3ojlxvs46-xuEJEplqBHPlymxtzQi8KQ</a></p>

<p><strong>Website:</strong> <a href="https://www.incidentfox.ai/" rel="noopener noreferrer">incidentfox.ai</a></p>

<p>Would love feedback, especially from teams that have tried to build internal AI tooling. What did you run into?</p>

