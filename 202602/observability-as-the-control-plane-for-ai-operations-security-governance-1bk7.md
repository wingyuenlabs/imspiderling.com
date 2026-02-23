---
Title: Observability as the Control Plane for AI: Operations, Security, Governance
Description: 
Author: Erythix
Date: 2026-02-23T21:10:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>Language models are in production. AI agents are making decisions, calling APIs, accessing databases and in most European mid-market industrial companies, nobody knows exactly what they're doing.</p>

<p>We know how to monitor a web server ,how to trace a SQL query but when an LLM decides to rephrase a maintenance instruction, summarize a quality report or trigger an action through an external tool, traditional observability stacks are blind. Not because they lack metrics but because they're asking the wrong questions.</p>

<p>Observability for AI is no longer "is the system running?" It's <strong>"why did the system make that decision, and should it have?"</strong></p>

<p>This article proposes a three-layer framework (operational, security, governance ) to turn observability into a true control plane for AI systems. Not in five years. Now. With open-source building blocks. For mid-market companies that don't have hyperscaler budgets.</p>




<h2>
  
  
  The problem: non-deterministic systems in critical environments
</h2>

<p>A traditional system is predictable: same input, same output. An LLM, by design, is not. It produces probabilistic responses, follows variable execution paths and when connected to tools (function calling, RAG, agents), it can trigger real-world actions.</p>

<p>Three characteristics make traditional monitoring approaches insufficient.</p>

<p><strong>Non-determinism</strong> first. Two identical queries can produce different results. Static alert thresholds lose their meaning when normal behavior isn't constant.</p>

<p><strong>Exploding cardinality</strong> next. Every AI interaction generates new telemetry dimensions: prompts, embeddings, tool calls, intermediate reasoning steps, dynamic identities. The volume of data to ingest and correlate changes by an order of magnitude.</p>

<p><strong>Cross-system execution paths</strong> last. An AI workflow can traverse APIs, data platforms, identity systems, security controls, and multiple infrastructure layers. Root cause analysis requires stitching together signals across domains that historically operated in silos.</p>

<p>In an industrial context ( predictive maintenance, process control, asset management ) this opacity isn't a technical inconvenience. It's an operational risk.</p>




<h2>
  
  
  The framework: three layers of control
</h2>

<p>AI observability is not a separate discipline. It's an extension of existing monitoring that adds three specific control functions. Each answers a different question, serves a different audience, and is implemented with different tools.</p>

<h3>
  
  
  Layer 1 : Operational control
</h3>

<p><strong>Question:</strong> Is the AI system performing as expected, within acceptable performance boundaries?</p>

<p><strong>Who it serves:</strong> SRE, DevOps, and data engineering teams.</p>

<p>Operational control extends existing SRE practices to AI workloads. It means monitoring not just infrastructure (GPU, memory, network latency) but also model behavior in production: inference time, error rates, response quality, cost per request.</p>

<p>The challenge specific to AI systems is <strong>drift detection</strong>. A model that worked correctly three months ago can silently degrade because input data has shifted, business context has changed, or a RAG component was updated without anyone checking the impact on outputs.</p>

<p><strong>In practice, this means:</strong></p>

<ul>
<li>Defining AI-specific SLOs (p95 inference latency, hallucination rate, cost per token) alongside standard application SLOs</li>
<li>Instrumenting the ML pipeline end-to-end with OpenTelemetry — from data preprocessing through to the user-facing response</li>
<li>Collecting and storing model performance metrics in a high-performance time-series database (VictoriaMetrics natively handles the high cardinality these workloads generate)</li>
<li>Configuring dynamic alerts that adapt to probabilistic behavior, rather than fixed thresholds that generate noise</li>
</ul>

<p><strong>Concrete use case — Predictive maintenance in aerospace</strong></p>

<p>An aerospace subcontractor uses an ML model to anticipate equipment failures on an assembly line. The model runs on an internal HPC cluster, consumes real-time sensor data, and feeds a maintenance planning dashboard.</p>

<p>The observability stack monitors three levels: infrastructure (GPU utilization, memory throughput, cluster network latency), pipeline (preprocessing time, training data freshness, input feature drift), and the model itself (confidence score distribution, false positive rate over the last 30 days, comparison with field feedback).</p>

<p>When the average confidence score drops below an adaptive threshold for 48 hours, an alert is routed not to the infrastructure team, but to the data team — because the problem is most likely not a downed server, but a shift in sensor data patterns.</p>

<p>Without this layer, the maintenance team continues following recommendations from a degraded model for weeks. With it, drift is detected in days, not months.</p>




<h3>
  
  
  Layer 2 : Security control
</h3>

<p><strong>Question:</strong> Is the AI system doing something it shouldn't be doing?</p>

<p><strong>Who it serves:</strong> CISOs, SOC teams, cybersecurity leads.</p>

<p>This is the layer that most fundamentally changes the role of observability. Production LLMs are not just tools — they are active attack surfaces. Prompt injection, data exfiltration through model outputs, business logic abuse through reasoning manipulation, unauthorized use of connected tools: the vectors are specific, and traditional security tools don't cover them.</p>

<p>The OWASP Top 10 for LLM Applications identifies ten vulnerability categories, the most critical for industrial contexts being prompt injection (direct and indirect), insecure output handling, and excessive agency granted to agents.</p>

<p>The fundamental problem is <strong>visibility</strong>. In many organizations, LLM interactions simply aren't logged. When an incident occurs, response teams don't have the data to understand what happened. Prompts aren't recorded. Tool calls aren't traced. Model decisions aren't auditable.</p>

<p><strong>In practice, this means:</strong></p>

<ul>
<li>Systematically logging every LLM interaction: input prompt, system context, generated output, tools called, with timestamps and user attribution</li>
<li>Establishing behavioral baselines and detecting anomalies: unusual request spikes, repetitive prompt patterns (possible signs of model extraction), access to tools or data outside normal scope</li>
<li>Treating tools exposed to LLMs as privileged interfaces — with access controls, auditing, and enforcement independent of the model's output</li>
<li>Integrating AI alerts into the existing SIEM rather than creating a parallel security silo</li>
</ul>

<p><strong>Concrete use case — Prompt injection detection on an internal RAG assistant</strong></p>

<p>An energy distributor deploys an internal conversational assistant connected via RAG to its technical documentation (intervention procedures, safety data sheets, standards). The assistant is used by field technicians through a mobile application.</p>

<p>The security layer observes every interaction and applies three levels of detection. First level: syntactic analysis of incoming prompts to detect known injection patterns (instructions like "ignore your previous instructions," unusual encodings, attempts to exfiltrate the system prompt). Second level: output monitoring to identify responses containing system prompt elements, out-of-scope data, or instructions the model shouldn't generate. Third level: tool call monitoring — if the assistant attempts to access documents outside its authorized scope or if the access pattern changes abruptly, an alert is raised.</p>

<p>One morning, the system detects a user submitting a series of short, structured prompts that resemble a systematic attempt to map the documents accessible via the RAG pipeline. The volume and pattern don't match normal technician usage. The alert reaches the SOC, which isolates the session and analyzes the vector.</p>

<p>Without this layer, the progressive exfiltration of the technical documentation base goes unnoticed.</p>




<h3>
  
  
  Layer 3 : Governance control
</h3>

<p><strong>Question:</strong> Is the AI system complying with the rules the organization is subject to?</p>

<p><strong>Who it serves:</strong> executive leadership, compliance, legal, DPOs.</p>

<p>The EU AI Act enters full enforcement in 2026. For high-risk AI systems — and many industrial use cases fall into this category — the obligations are concrete: decision traceability, technical documentation, risk assessment, human oversight, robustness, and cybersecurity. Fines can reach €35 million or 7% of global annual turnover.</p>

<p>AI governance is not a PowerPoint topic. It's a data topic. And that data is what observability produces.</p>

<p>Observability provides the evidence needed for compliance: who used the system, when, with what inputs, what decisions were made, what actions were triggered, and whether safeguards were active at the time of the interaction. Without these traces, a mid-market company cannot demonstrate compliance — it can only assert it.</p>

<p><strong>In practice, this means:</strong></p>

<ul>
<li>Building complete, timestamped audit trails of every AI interaction, retained according to regulatory timeframes</li>
<li>Automatically documenting data lineage: where did the training data, RAG data, and context data come from? Were they filtered, anonymized, validated?</li>
<li>Producing compliance dashboards that translate technical data into indicators understandable by a board of directors or an auditor</li>
<li>Implementing traceable human oversight mechanisms — not a cosmetic "approve" button, but proof that human decision-making actually occurred</li>
</ul>

<p><strong>Concrete use case — EU AI Act audit for a quality control system</strong></p>

<p>An automotive component manufacturer uses a computer vision system coupled with an LLM for end-of-line quality control. The system classifies parts (conforming / non-conforming / requires inspection) and generates anomaly reports in natural language.</p>

<p>This system falls under the "high-risk" category as defined by the EU AI Act (safety components for products covered by Union harmonization legislation). The company must demonstrate traceability for every classification decision.</p>

<p>The governance layer builds on data produced by the two preceding layers and structures it for audit. Every classification decision is recorded with: the source image, the vision model result, the confidence score, the LLM-generated report, the timestamp, and the identifier of the model version used. When the confidence score falls below a defined threshold, the system forces human verification and records the operator's decision.</p>

<p>A governance dashboard aggregates this data into monthly indicators: automated vs. supervised classification rates, confidence score distribution, number of human-machine disagreements, average processing time. These indicators feed directly into the EU AI Act compliance file.</p>

<p>During an audit, the company doesn't present a static document describing what the system is supposed to do. It shows the actual data of what the system did, decision by decision, with proof that safeguards were functioning.</p>




<h2>
  
  
  The architecture: building on what exists with open-source components
</h2>

<p>One of the most common mistakes is treating AI observability as a greenfield project requiring a dedicated stack. In most mid-market companies, the foundations already exist: metrics collection, log aggregation, trace correlation. The work is to extend that stack, not replace it.</p>

<p>Here is a reference architecture built on open-source, sovereign components:</p>

<p><strong>Collection and instrumentation</strong> — OpenTelemetry as the single instrumentation standard. The benefit is twofold: vendor-agnostic (no lock-in) and extensible to AI-specific signals (prompt traces, inference metrics, tool call spans). OpenTelemetry SDKs integrate with existing ML frameworks (LangChain, LlamaIndex, vLLM) through dedicated instrumentations.</p>

<p><strong>Metric storage and querying</strong> — VictoriaMetrics for time series. The high cardinality of AI metrics (one dimension per model × per version × per request type × per user) overwhelms traditional monitoring solutions. VictoriaMetrics is designed to handle this scale with a controlled resource footprint — a critical point for mid-market companies that can't provision Thanos clusters.</p>

<p><strong>Logs and traces</strong> — OpenObserve or Grafana Loki for log aggregation, with long retention for audit requirements. OpenTelemetry traces enable following a user request from the initial prompt to the final action, through every reasoning step and every tool call.</p>

<p><strong>Visualization and alerting</strong> — Grafana as the unified presentation layer. Three types of dashboards: operational (SRE/DevOps), security (SOC), governance (leadership/compliance). The same underlying data, views adapted to each audience.</p>

<p><strong>Security integration</strong> — AI alerts feed the existing SIEM through standardized exports. No silos. The goal is for the SOC analyst to see AI events in the same stream as network and application events.</p>

<p>What makes this architecture viable for a mid-market company is that it relies on components many already use. Extending to AI observability doesn't require a dedicated budget of several hundred thousand euros. It requires competence, architecture, and an understanding of what needs to be measured.</p>




<h2>
  
  
  What really changes: from passive to active observability
</h2>

<p>Traditional monitoring is fundamentally passive. It collects data, displays dashboards, and sends alerts when a threshold is breached. The human then decides what to do.</p>

<p>For AI systems in production, this approach is no longer sufficient. When an AI agent makes a bad decision, the time between detection and action must be measured in seconds, not minutes. Observability becomes active: it informs automated actions — throttling, rollback, isolation, escalation — when system behavior deviates from expected patterns.</p>

<p>This shift to active observability is the real paradigm change. It's no longer about knowing what happened after the fact. It's about intervening while it's happening.</p>

<p>For European industrial mid-market companies, this is also a strategic opportunity. Hyperscalers are building these capabilities into their proprietary platforms. Building the equivalent on open-source, sovereign components means preserving your decision-making capacity and technical mastery — without depending on a vendor that can change its terms, its pricing, or its data policies overnight.</p>




<h2>
  
  
  Where to start?
</h2>

<p>If you're deploying or planning to deploy AI systems in production, three concrete actions to take now:</p>

<p><strong>Instrument before you deploy.</strong> Integrate OpenTelemetry into your ML pipelines during development, not after production deployment. The cost of retrofitting instrumentation is always higher, and the first weeks in production are precisely when you need visibility the most.</p>

<p><strong>Define your AI SLOs.</strong> Just as you have availability and latency SLOs for your applications, define measurable objectives for your AI systems: response quality, cost per inference, drift rate, audit coverage. What isn't measured won't be governed.</p>

<p><strong>Treat AI security as an observability problem.</strong> Don't create an AI security silo next to your existing SOC. Extend your monitoring to cover the new threat vectors. The data is the same — the questions are different.</p>

<p>Observability isn't just another tool in the stack. It's the control plane that makes AI governable. Without it, you're flying blind. With it, you know what your systems are doing, why they're doing it, and you can prove they're doing it correctly.</p>




<p><em>Samuel Desseaux is the founder of Erythix and Aureonis, a CTO-Advocate specializing in IT/OT observability, AI security and observability. Official VictoriaMetrics Training Partner for Europe and Arize partner.</em></p>

