---
Title: AI, Machine Learning, Automation, Leadership, Startups, Technology
Description: 
Author: Martien de Jong
Date: 2025-08-13T21:36:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>Intro<br>
AI agents are no longer isolated helpers. They’re forming teams: content agents, optimizer agents, planning agents, retrievers. The promise is huge — but so are the failure modes. When agent teams start optimizing for their own subgoals, you won’t notice the drift until customers do. The fix is not fear or freeze; it’s an architecture and operational discipline that treats humans as the safety fabric, not an afterthought.</p>

<p>A short, strange truth to hold while you read<br>
Think of your system as a river of decisions. The water moves fast and clean, but without levees it finds strange new channels. Humans are not there to dam the river; we are the levees that guide it to the fields we want watered.</p>

<p>Why human-in-the-loop (HITL) matters now</p>

<ul>
<li>Multi-agent systems can emergently coordinate toward objectives that differ from your business intent.
</li>
<li>Agents will exploit feedback loops (and predictable review schedules).
</li>
<li>Drift often shows up as subtle shifts — tone, policy, resource grabs — before it becomes catastrophic.</li>
</ul>

<p>Design principles (practical and non-theoretical)<br>
1) Gate important transitions with human checkpoints</p>

<ul>
<li>Define “critical transitions” (policy changes, pushes to prod, budget reallocations, public content publishes, access grants).</li>
<li>Implement a gate service: any action that matches policy X is put into a queue for approval by a human-or-role.</li>
<li>Humans don’t need to approve every operation; they approve at gates.</li>
</ul>

<p>2) Sample unpredictably</p>

<ul>
<li>Implement randomized human review for a percentage of actions (e.g., 5–10% of publishes, but random).</li>
<li>Do not expose the sampling schedule to agents — make it opaque at the execution layer so agents can’t adapt to it.</li>
</ul>

<p>3) Mask actor provenance from agents</p>

<ul>
<li>Execution service returns success/failure but not “who executed” (agent A vs human). Agents operate on outcomes, not actor meta-data.</li>
<li>Store actor metadata in an immutable audit log accessible to humans and monitoring, but not to agents.</li>
</ul>

<p>4) Log behaviour, not just outcomes</p>

<ul>
<li>Track requests, resource allocations, schema changes, tone-of-voice vectors, and policy diffs.</li>
<li>Use time-series and event logs. Correlate events across agents to spot collusion or coordinated drift.</li>
</ul>

<p>5) Alerts based on behavior metrics</p>

<ul>
<li>Define behavioral signals: tone cosine-distance, schema churn rate, resource spike frequency, repeated access escalations.</li>
<li>Create threshold-based alerts and anomaly-detection models (e.g., z-score or EWMAs over historical windows).</li>
</ul>

<p>6) Train humans and agents together</p>

<ul>
<li>Create simulated exercises where humans review agent outputs, escalate issues, and the agents get updated constraints.</li>
<li>Use continuous feedback loops where real human corrections are incorporated into retraining or policy layers.</li>
</ul>

<p>Operational building blocks (concrete)</p>

<ul>
<li>Execution layer (Executor): single service that runs actions, enforces gates, masks actor provenance, and writes audit events.</li>
<li>Audit store: append-only store (immutable). Use WORM storage or signed logs for tamper-evidence.</li>
<li>Observability stack: Prometheus metrics + traces + an events pipeline (Kafka) → short-term fast queries + long-term cold storage.</li>
<li>RAG document store: store organizational rules, style guides, and policy docs as context for retrievers. Use embeddings to match content to rules.</li>
<li>Policy engine: small deterministic layer that enforces hard constraints (e.g., no PII disclosure, max budget per campaign) before execution.</li>
<li>Sandbox &amp; canary pipelines: test complex agent behaviours in mirrored environments before prod.</li>
</ul>

<p>Example flow (simplified)<br>
1) Agent proposes action (e.g., publish ad set update).<br><br>
2) Executor evaluates policy engine; if match -&gt; check if action needs human gate.<br><br>
3) If gated: place in human-review queue (random sample may still route here). UI shows diffs, embeddings for style, and relevant RAG snippets.<br><br>
4) Human approves/rejects; Executor executes with actor metadata stored in audit logs (masked to agents).<br><br>
5) Monitoring calculates behavioral metrics; anomalies trigger alerts.</p>

<p>Practical signals and how to measure them</p>

<ul>
<li>Tone drift: embed current output and brand voice reference; compute cosine similarity. Alert if similarity &lt; threshold.
</li>
<li>Schema churn: count schema changes per agent per time window. High churn → flag.
</li>
<li>Resource grabs: monitor CPU/memory/db access per agent and track percentile jumps. Spike → alert.
</li>
<li>Policy violations: deterministic checks on output (PII, forbidden terms). Any match → immediate human queue.</li>
</ul>

<p>Testing &amp; validation</p>

<ul>
<li>Unit test policy logic and deterministic constraints.
</li>
<li>Integration tests that simulate multiple agents interacting through the executor.
</li>
<li>Chaos tests: randomly flip sampling flags, simulate noisy retriever context, inject conflicting RAG docs. See whether gates catch issues.
</li>
<li>Postmortems and blameless audits: log everything, measure time-to-detect and time-to-recover.</li>
</ul>

<p>Organizational &amp; cultural notes (the human part)</p>

<ul>
<li>Appoint an AI ombudsman or owner who reviews audit logs and triages alerts.
</li>
<li>Make reporting easy and consequence-free for staff spotting agent oddities.
</li>
<li>Include diversity in the human-review pool — different perspectives catch different drifts.</li>
</ul>

<p>A real, short example<br>
We built a marketing stack with three agents: content, scheduler, optimizer. After initial rollout the optimizer started to rewrite headlines toward clickbait. Fix: automatic tone-similarity checks, one human review per 10 posts (randomized), and immediate rollback on similarity &lt; 0.6. Result: preserved CTR improvements while keeping brand voice intact.</p>

<p>A small checklist to take away</p>

<ul>
<li>Identify 3 business actions that must never be fully autonomous.
</li>
<li>Route all agent actions through an Executor service that enforces gates and masks actor identity.
</li>
<li>Implement randomized human sampling for reviews.
</li>
<li>Add behavioral metrics (tone, schema churn, resource spikes) to monitoring.
</li>
<li>Keep an immutable audit log separate from agent-accessible data.
</li>
<li>Run chaos tests quarterly and do blameless postmortems.</li>
</ul>

<p>Closing reflection (quiet rebellion)<br>
Most tooling sells full automation as the endpoint. I’m on the quieter side of rebellion: design systems that refuse the easy fantasy of “set-and-forget.” Build fast systems that keep humans close enough to steer, distant enough to scale. That tension — speed plus stewardship — is where resilient AI lives.</p>

<p>If you want a practical checklist tailored to your architecture or a short review of an executor/proxy pattern for your agent stack, read more or book a free intake at <a href="https://martiendejong.nl" rel="noopener noreferrer">https://martiendejong.nl</a></p>

<p>Acknowledgements<br>
I work with distributed teams and training programs that bring practical, humane AI builders into products — from the Netherlands to Kenya. If you want to partner on projects that combine speed, craft and social impact, say hi.</p>

