---
Title: Building Healthcare-Grade Multi-Agent Systems with Gemini
Description: 
Author: Disleve Kanku
Date: 2025-12-03T21:38:02.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Introduction — Why I Joined the Agents Intensive</strong></p>

<p>When I enrolled in the Kaggle AI Agents Intensive, I had one goal:<br>
to learn how to build AI systems that go beyond chatbots, systems that can reason, take actions, coordinate, and transform real workflows.</p>

<p>As someone working at the intersection of data engineering and healthcare, I wanted to see whether agentic architectures could meaningfully improve clinical research, where slow evidence synthesis costs time, money, and sometimes lives.</p>

<p>The answer turned out to be yes, and the Intensive course gave me the exact structure, concepts, and building blocks I needed.</p>

<p><strong>The Problem — Healthcare Research Overload</strong></p>

<p>Clinical researchers are drowning in papers.<br>
PubMed adds thousands of new articles every week, yet most researchers still copy-paste into Google Docs, manually extract findings, and lose track of what they’ve reviewed.</p>

<p>This creates four fundamental problems:</p>

<ol>
<li>Too much literature to scan manually</li>
</ol>

<p>2.Fragmented tools (PubMed to PDFs to spreadsheets)</p>

<p>3.No structured memory across searches</p>

<p>4.No reproducibility when using typical LLM assistants</p>

<p>Healthcare research requires traceability, auditability, and evidence-grounded outputs, something a single monolithic LLM cannot reliably guarantee.</p>

<p>The Intensive helped me realize that agents, not prompts, are the right solution.</p>

<p><strong>Why Multi-Agent Systems?</strong></p>

<p>The turning point in the Intensive course was Day 5.</p>

<p>I realized that real-world workflows already operate like agent systems:</p>

<ul>
<li><p>Someone interprets the question</p></li>
<li><p>Someone searches</p></li>
<li><p>Someone extracts the evidence</p></li>
<li><p>Someone synthesizes</p></li>
<li><p>Someone reviews the output</p></li>
</ul>

<p>So instead of forcing one LLM to do everything, I built a pipeline of specialized agents, each with a clear and testable responsibility.</p>

<p>Kaggle’s lessons on:</p>

<ul>
<li><p>tool design,</p></li>
<li><p>memory/state,</p></li>
<li><p>agent-to-agent communication,</p></li>
<li><p>observability,</p></li>
<li><p>evaluation,</p></li>
<li><p>and deployment</p></li>
</ul>

<p>all showed me how to move from “chatbot” thinking to real “AI system” engineering.</p>

<p><strong>What I Built — The Healthcare Research Agent</strong></p>

<p>Using everything from the Intensive course, I built a fully functioning, healthcare-grade multi-agent research system powered by:</p>

<ul>
<li><p>Gemini 1.5 Pro for reasoning</p></li>
<li><p>PubMed E-utilities for biomedical search</p></li>
<li><p>Supabase for sessions, logs, and reproducibility</p></li>
<li><p>FastAPI for orchestration</p></li>
<li><p>Docker + Cloud Run for deployment</p></li>
</ul>

<p><strong>The Agents:</strong></p>

<ol>
<li><p><strong>QueryAgent</strong><br>
Converts a biomedical question into PICO + a valid PubMed search query.</p></li>
<li><p><strong>SearchAgent</strong><br>
Hits PubMed, retrieves metadata, and returns candidate papers.</p></li>
<li><p><strong>EvidenceExtractorAgent</strong><br>
Pulls abstracts and extracts structured evidence: interventions, outcomes, effect sizes.</p></li>
<li><p><strong>SummarizerAgent</strong><br>
Synthesizes everything into a concise research brief with inline citations.</p></li>
<li><p><strong>Orchestrator</strong><br>
Coordinates all agent-to-agent messaging and ensures the workflow executes safely.</p></li>
</ol>

<p><strong>Why this matters</strong></p>

<p>Instead of asking a chatbot<br>
"What’s the evidence for metformin + insulin in Type 2 diabetes?"<br>
and hoping it’s correct…</p>

<p>…my system performs a full evidence pipeline, with logs, provenance, citations, and stateful sessions.</p>

<p>It's built for researchers, not casual AI use.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuputughjk7a4hj3pm61i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuputughjk7a4hj3pm61i.png" alt="Data Architecture" width="800" height="800"></a></p>

<p><strong>What I Learned from Kaggle</strong></p>

<p>The Intensive course taught me concepts that are rare to find all in one place:</p>

<ul>
<li><p>Agents need tools</p></li>
<li><p>Tools need safety</p></li>
<li><p>Sessions create continuity</p></li>
<li><p>Memory creates personalization</p></li>
<li><p>Observability makes production possible</p></li>
<li><p>Evaluation prevents regression</p></li>
<li><p>Deployment forces clarity</p></li>
</ul>

<p>This wasn’t a tutorial, it was a playbook for building real AI systems.</p>

<p>The biggest shift:<br>
I no longer think in terms of prompts.<br>
I now think in modular workflows, state machines, and agent ecosystems.</p>

<p><strong>What’s Next — The Road Ahead</strong></p>

<p>In the next 90 days, I plan to:</p>

<ul>
<li><p>Expand to full-text PDF extraction</p></li>
<li><p>Add a vector RAG pipeline</p></li>
<li><p>Build specialized oncology and cardiology variants</p></li>
<li><p>Create a research agent dashboard</p></li>
<li><p>Publish an open benchmark for biomedical agent systems</p></li>
</ul>

<p>My long-term goal is to contribute to the emerging field of healthcare-grade agentic AI, where safety, reproducibility, and transparency matter most.</p>

<p>The Kaggle AI Agents Intensive gave me the structure, the tools, and the conceptual foundation to build something I’ve wanted for years.</p>

<p>This experience has genuinely been transformative and I’m excited to keep pushing the boundaries of what multi-agent systems can do in clinical research.</p>

<p>Thank you to the Kaggle team, instructors, and community for this opportunity.</p>

