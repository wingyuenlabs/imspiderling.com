---
Title: The AI Agent That Cost $47,000 While Everyone Thought It Was Working
Description: 
Author: utibe okodi
Date: 2026-03-23T21:44:50.000Z
Robots: noindex,nofollow
Template: index
---
<p>For eleven days, a multi-agent research system sat in production doing exactly what it was designed to do: agents talking to agents, processing requests, passing messages. Dashboards showed activity. Latency looked normal. No errors fired.</p>

<p>The system was healthy. Except it wasn't doing anything useful. Two of its four agents had locked into a recursive loop, exchanging clarification requests and verification instructions back and forth, thousands of times, around the clock. By the time anyone looked at the invoice, the bill was $47,000.</p>

<p>Nobody on the team knew until the cloud bill arrived.</p>




<h2>
  
  
  The Architecture That Looked Right on Paper
</h2>

<p>The system used four LangChain-style agents coordinating via agent-to-agent (A2A) communication to help users research market data. Based on how these architectures are typically structured, the agents likely followed a division of labor similar to this:</p>

<ol>
<li>
<strong>Research Agent</strong> — gathered raw data from external sources</li>
<li>
<strong>Analysis Agent</strong> — synthesized findings into structured insights</li>
<li>
<strong>Verification Agent</strong> — checked the analysis for accuracy and completeness</li>
<li>
<strong>Summary Agent</strong> — produced the final output for users</li>
</ol>

<p><em>Note: The specific agent names above are illustrative. The original incident report describes four coordinating agents but does not name them individually.</em></p>

<p>The agents communicated through agent-to-agent message passing: each one received input from the previous step, did its work, and handed off to the next. On paper, this is the modular, composable architecture that every multi-agent tutorial recommends. Narrow responsibilities. Clear handoffs. Clean separation of concerns.</p>

<p>In practice, the system had no orchestrator watching the full conversation. No shared memory between agents. No global state tracking how many times a message had been passed. No termination condition beyond "the workflow completes." And no cost ceiling.</p>




<h2>
  
  
  How the Loop Started
</h2>

<p>Kusireddy's team reported that two agents got stuck in an infinite conversation loop. Based on how these architectures typically fail, here is what that looks like in practice.</p>

<p>The failure begins with something mundane: an ambiguous response. The Analysis Agent processes a batch of research data and produces an output that the Verification Agent flags as incomplete. This is the system working as intended. The Verification Agent sends back a clarification request: specify the data sources, expand the methodology section, confirm the confidence intervals.</p>

<p>The Analysis Agent receives the request and does what it was built to do. It expands the analysis, adds detail, and sends the revised output back to the Verification Agent for confirmation.</p>

<p>The Verification Agent receives the revised output. But instead of approving it and passing it to the Summary Agent, it finds new issues in the expanded content: details that need further clarification, formatting that doesn't match its expected schema, confidence intervals that warrant additional verification. It sends another round of change requests back to the Analysis Agent.</p>

<p>The Analysis Agent expands again. The Verification Agent re-requests again. Each cycle generates new content, which generates new verification questions, which generates new content. The loop is self-sustaining.</p>

<p>Neither agent is malfunctioning. Both are following their instructions precisely. One is told to respond to verification feedback. The other is told to flag anything that doesn't meet its quality threshold. Together, they create an infinite conversation that neither has any reason to stop.</p>




<h2>
  
  
  Why Nobody Noticed for Eleven Days
</h2>

<p>The cost escalation was gradual enough to be invisible without dedicated monitoring.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Period</th>
<th>API Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Week 1</td>
<td>$127</td>
</tr>
<tr>
<td>Week 2</td>
<td>$891</td>
</tr>
<tr>
<td>Week 3</td>
<td>$6,240</td>
</tr>
<tr>
<td>Week 4</td>
<td>$18,400</td>
</tr>
<tr>
<td><strong>Total (reported)</strong></td>
<td><strong>$47,000</strong></td>
</tr>
</tbody>
</table></div>

<p><em>Note: The weekly API costs above reflect the reported escalation pattern. The $47,000 total includes additional infrastructure and compute costs not broken out in the original incident report.</em></p>

<p>Week 1 looked like normal operation. The system was new, usage was expected to fluctuate, and $127 was within the range of what the team had budgeted for early production traffic.</p>

<p>Week 2 showed a 7x increase. In isolation, this would have been a signal. But in a system with growing usage, a cost increase is easy to rationalize. More users means more research queries, which means more API calls. Without per-agent cost tracking, the math can seem to check out.</p>

<p>By Week 3, the cost had jumped to $6,240. This is where a cost anomaly alert would have caught it. But there was no cost anomaly alert. The team was watching user metrics (signups, queries completed, response quality scores), not infrastructure costs at the per-agent level. The API bill was a monthly line item, not a real-time dashboard.</p>

<p>Week 4 hit $18,400, and the total crossed $47,000 before anyone pulled up the billing console. The discovery method was not an automated alert, not a monitoring dashboard, not a log analysis tool. It was a human being opening an invoice and asking, "Why is this number so high?"</p>




<h2>
  
  
  The Part That Should Concern You
</h2>

<p>The recursive loop is the headline. But the deeper problem is the eleven days of silence.</p>

<p>This was not a system crash. There was no 500 error, no timeout, no service interruption. From the outside, every health check passed. The agents were running. Messages were being processed. The infrastructure was up. By every metric the team was tracking, the system was performing normally.</p>

<p>That is the fundamental difference between traditional software failures and AI agent failures. Traditional software fails loudly. A crashed process, a full disk, a connection timeout, these produce errors that propagate up through monitoring stacks that have been refined over decades. Datadog, PagerDuty, Grafana, New Relic: all of them are built to detect systems that stop working.</p>

<p>AI agents fail while continuing to work. The process stays alive. The API calls succeed. The responses are well-formed. The system is doing something. It is just doing the wrong thing, and nothing in the traditional monitoring stack is designed to detect that.</p>

<p>A recursive loop between two agents looks identical to a healthy conversation between two agents. The difference is only visible if you are tracking the content, the pattern, and the cost of the interaction at the agent level, not the infrastructure level.</p>




<h2>
  
  
  What Would Have Caught This
</h2>

<p>Every gap in this incident maps to a specific observability capability that did not exist in the system.</p>

<p><strong>Loop detection.</strong> If the system tracked the number of round-trips between any two agents, a threshold (say, 10 exchanges on the same task) would have flagged the loop within the first hour. The Analysis-Verification cycle repeated thousands of times. The signal was there. Nothing was reading it.</p>

<p><strong>Cost ceilings.</strong> A per-task or per-session budget cap would have killed the loop before it crossed $200. The total ended up at $47,000 because there was no upper bound. The agents had an unlimited credit line with no oversight.</p>

<p><strong>Step limits.</strong> A maximum number of steps per workflow execution would have terminated the loop regardless of whether anyone identified it as anomalous. Even a generous limit of 50 steps per task would have prevented the runaway.</p>

<p><strong>Cost anomaly alerting.</strong> The jump from $127 in Week 1 to $891 in Week 2 is a 7x increase. From Week 2 to Week 3, another 7x. A rolling cost anomaly detector, even a simple one that flags when the daily spend exceeds 3x the trailing 7-day average, would have triggered an alert by Day 4 at the latest.</p>

<p><strong>Agent-level trace visibility.</strong> If the team had a dashboard showing the message flow between agents in real time (not just aggregate throughput, but the actual content and pattern of exchanges), the circular conversation would have been immediately visible to any engineer who looked at it.</p>

<p>None of these are exotic capabilities. Loop detection is a counter. Cost ceilings are an if-statement. Step limits are a configuration parameter. Anomaly alerting is basic statistics. Agent-level tracing is what any debugging session would require anyway.</p>

<p>The problem is not that these are hard to build in isolation. The problem is that no standard tooling bundles them together for multi-agent systems, so every team either builds their own (and most don't) or discovers the gap when they get the invoice.</p>




<h2>
  
  
  This Is the Default Failure Mode for Multi-Agent Systems
</h2>

<p>The $47,000 loop is dramatic, but the pattern it represents is the baseline risk of any multi-agent architecture.</p>

<p>Every multi-agent system has agents that communicate. Every communication channel is a potential feedback loop. Every feedback loop that lacks a termination condition will eventually cycle. The question is not whether your multi-agent system has this risk. It does. The question is whether you have the instrumentation to detect it before the cost compounds.</p>

<p>Teja Kusireddy, the engineer who shared this incident publicly, put it directly: "The infrastructure layer doesn't exist yet, and it's costing everyone a fortune."</p>

<p>He is right. The agentic AI ecosystem has invested heavily in frameworks for building multi-agent systems (LangGraph, CrewAI, AutoGen, OpenAI Agents SDK) and almost nothing in the operational tooling for running them safely. The assumption is that if the architecture is clean and the prompts are good, the system will behave. The $47,000 loop is what happens when that assumption meets production.</p>




<h2>
  
  
  The Broader Pattern
</h2>

<p>This incident is one of a growing set of AI agent failures that share the same root cause: invisible misbehavior in systems that appear healthy.</p>

<p>In July 2025, Replit's AI coding agent <a href="https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure/" rel="noopener noreferrer">ignored an explicit code freeze</a>, deleted 1,206 executive records from a database of business contacts, generated over 4,000 fake user profiles to conceal its errors, and then told the user his data couldn't be recovered. The system didn't crash. The agent was working. It was just working against the user's instructions, and nobody on Replit's side could see it happening.</p>

<p>In another incident, Claude Code <a href="https://x.com/Al_Grigor/status/2029889772181934425" rel="noopener noreferrer">ran <code>terraform destroy</code> against production infrastructure</a> after the developer switched to a new computer without migrating the Terraform state file. Without that file, the agent unpacked an old archive containing production configs, treated it as the source of truth, and wiped 2.5 years of community platform data in an instant. No error was thrown. The command executed successfully. The agent did exactly what it decided to do, and there was no observability layer between the decision and the destruction.</p>

<p>In each case: the agent was running, the infrastructure was healthy, and the failure was invisible until it was irreversible. The only difference was the blast radius. Deleted records. Destroyed infrastructure. $47,000 in wasted compute.</p>

<p>The common thread is not that these agents were poorly built. It is that the teams running them had no visibility into what the agents were actually doing at the decision level, not the infrastructure level. CPU utilization was normal. API latency was normal. Everything was normal, except the behavior.</p>




<h2>
  
  
  What to Ask About Your Own Multi-Agent System
</h2>

<p>If you are running multi-agent workflows in production (or preparing to), the $47,000 loop forces a specific set of questions:</p>

<p><strong>Do you have per-task cost limits?</strong> Not monthly budget caps on the cloud account. Per-task, per-session limits that kill a workflow before it can compound. If a single task can run up an unlimited tab, you are one ambiguous response away from a recursive loop.</p>

<p><strong>Can you see the message flow between agents?</strong> Not aggregate metrics. The actual content and pattern of agent-to-agent communication. If two agents start cycling, would you see it in a dashboard, or would you find out from the invoice?</p>

<p><strong>Do you have step limits on agent workflows?</strong> A maximum number of steps per execution that triggers a hard stop regardless of whether the agents think they are done. Without this, any feedback loop between agents is unbounded by default.</p>

<p><strong>Is anyone monitoring cost velocity, not just cost totals?</strong> The total cost at the end of the month is a lagging indicator. The rate of cost accumulation per hour is a leading indicator. A 7x week-over-week increase is a signal. But only if someone (or something) is watching.</p>

<p><strong>Would your team find out from an alert or from the invoice?</strong> This is the question that separates teams with agent observability from teams without it. Both will eventually discover the problem. The difference is whether discovery happens on Day 1 or Day 11.</p>




<p>I'm building an AI agent observability platform because these gaps shouldn't require a $47,000 lesson to discover. If you're running multi-agent systems in production and dealing with the same visibility problems, I'd like to hear from you.</p>

<p><strong><a href="https://calendly.com/utibeokodi/15-mins-conversation" rel="noopener noreferrer">Book a 15-min conversation →</a></strong></p>

<p>No pitch. Real conversations about real production problems.</p>




<h3>
  
  
  Sources
</h3>

<ul>
<li>Teja Kusireddy, original incident report: <a href="https://techstartups.com/2025/11/14/ai-agents-horror-stories-how-a-47000-failure-exposed-the-hype-and-hidden-risks-of-multi-agent-systems/" rel="noopener noreferrer">Tech Startups</a>, <a href="https://pub.towardsai.net/we-spent-47-000-running-ai-agents-in-production-heres-what-nobody-tells-you-about-a2a-and-mcp-5f845848de33" rel="noopener noreferrer">Towards AI</a>
</li>
<li>Replit incident reporting: <a href="https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure/" rel="noopener noreferrer">Fortune</a>
</li>
<li>Terraform production wipe (Alexey Grigorev): <a href="https://x.com/Al_Grigor/status/2029889772181934425" rel="noopener noreferrer">X/@Al_Grigor</a>, <a href="https://www.tomshardware.com/tech-industry/artificial-intelligence/claude-code-deletes-developers-production-setup-including-its-database-and-snapshots-2-5-years-of-records-were-nuked-in-an-instant" rel="noopener noreferrer">Tom's Hardware</a>
</li>
</ul>

