---
Title: Building Agentic Workflow: Auto Banking Customer Service with MindsDB
Description: 
Author: Jackie Cheng
Date: 2025-10-26T21:50:03.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Challenge
</h2>

<p>When a customer calls a bank, a lot happens behind the scenes: an agent listens, takes notes, types into Salesforce, and classifies the issue. Then the business owner will go through the unresolved issues in Salesforce and manually create a Jira story for further tracking.</p>

<p>It’s a mess of tabs, forms, and human fatigue. For <a href="https://github.com/mindsdb/mindsdb/tree/main/mindsdb%20hacktoberfest" rel="noopener noreferrer"><strong>Hacktoberfest</strong></a> 2025, our team decided to automate this entire workflow by building <a href="https://github.com/jiaqicheng1998/mindsdb/tree/main/mindsdb%20hacktoberfest/use-cases/AutoBankingCustomerService" rel="noopener noreferrer"><strong>AutoBankingCustomerService</strong></a> with <a href="https://mindsdb.com/" rel="noopener noreferrer"><strong>MindsDB</strong></a>.</p>

<p>Our goal was to turn hours of manual case handling into an automated pipeline that could:</p>

<ol>
<li>
<strong>Summarize</strong> each customer call.</li>
<li>
<strong>Classify</strong> whether it’s resolved or unresolved.</li>
<li>
<strong>Escalate</strong> unresolved issues directly into Jira, complete with recommendations for next steps.</li>
</ol>

<p>All of this needed to happen automatically, using enterprise systems that already exist (Salesforce, Jira, Confluence, PostgreSQL).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwxy402yeixcr3kuphpae.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwxy402yeixcr3kuphpae.jpg" alt="old workflow" width="800" height="450"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo76ws0k518txpa9ceytv.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo76ws0k518txpa9ceytv.jpg" alt="new workflow with mindsdb" width="800" height="450"></a></p>

<h2>
  
  
  Why MindsDB?
</h2>

<p>Because it lets our Python backend focus on logic instead of plumbing.</p>

<p>Instead of hard-coding every integration or retraining pipeline, we used MindsDB to connect multiple enterprise data sources and build a lightweight <strong>RAG-powered knowledge base</strong> in minutes.</p>

<p><strong>MindsDB gave us:</strong></p>

<ul>
<li>
<strong>Unified interface:</strong> query and manage both AI models and data with familiar SQL syntax.</li>
<li>
<strong>Plug-and-play connectors:</strong> Salesforce, Jira, Confluence, PostgreSQL, all quickly accessible from one orchestration layer.</li>
<li>
<strong>Seamless RAG setup:</strong> ingest structured + unstructured content and use it as context for every agent call.</li>
<li>
<strong>Enterprise deployment options:</strong> open-source server deployable locally, in Docker, or on cloud, with sensitive information hidden by default in system metadata."</li>
<li>
<strong>Rapid iteration:</strong> prototype → deploy → refine, all inside our existing Python server without adding new infrastructure.</li>
</ul>

<p>In short, MindsDB handled the heavy lifting of data access and retrieval augmentation, while our backend handled the business logic and workflow orchestration.</p>

<h2>
  
  
  Architecture Overview
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwv2hnd4gpzb74siejb27.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwv2hnd4gpzb74siejb27.png" alt="architecture" width="800" height="347"></a></p>

<p>Our system automates customer support case management from ingestion to escalation using a Python backend orchestrated by MindsDB agents.</p>

<p><strong>1. Call Ingestion</strong><br>
Customer call transcripts are captured from Gong's conversation intelligence platform. In production, MindsDB would connect directly to Gong's API. For development, we mock this integration using PostgreSQL to simulate the call transcript data structure provided by Gong.</p>

<p><strong>2. First MindsDB Agent: Summary &amp; Classification</strong><br>
Connected to PostgreSQL, this agent performs two tasks:</p>

<ul>
<li>
<strong>Summarizes</strong> customer conversations into structured summaries</li>
<li>
<strong>Classifies</strong> cases as Resolved or Unresolved</li>
</ul>

<p>Our Python backend retrieves the agent's results and writes them to Salesforce.</p>

<p><strong>3. Second MindsDB Agent: RAG-Powered Recommendations</strong><br>
For Unresolved cases, a second agent activates:</p>

<ul>
<li>Queries a Confluence knowledge base via RAG for relevant context</li>
<li>Generates actionable ticket recommendations</li>
</ul>

<p>Our Python backend retrieves these recommendations and creates Jira tickets.</p>

<p><strong>4. Integration Layer</strong><br>
MindsDB provides the AI intelligence layer (connecting to PostgreSQL for data and Confluence for RAG), while our Python backend orchestrates all data writes to Salesforce and Jira.</p>

<blockquote>
<p><strong>Result:</strong> Fully automated workflow from transcript → analysis → escalation → documentation.</p>
</blockquote>
<h2>
  
  
  Step-by-Step Walkthrough
</h2>
<h3>
  
  
  <strong>1) Connect Data Sources to Break Down Data Silos</strong>
</h3>

<p>One of MindsDB's core strengths is connecting diverse enterprise data sources into a single, unified interface. Instead of navigating multiple platforms and APIs, all business-critical data becomes accessible through one SQL layer. That eliminates the data silos that typically fragment enterprise workflows.</p>

<p>For this project, we connected four different data sources, each representing a distinct data type:</p>

<p><strong>PostgreSQL</strong> – Structured call transcripts (tabular data)</p>

<ul>
<li>Stores raw customer call records in a traditional relational format</li>
<li><em>Note: In production, MindsDB supports direct integration with Gong's conversation intelligence platform. We used PostgreSQL as a substitute.</em></li>
</ul>

<p><strong>Salesforce &amp; Jira</strong> – Semi-structured business records (JSON/API data)</p>

<ul>
<li>Systems of record for customer cases and issue tracking</li>
<li>Queryable sources for metrics and analytics. Enables natural-language queries across CRM and ticketing data within the same interface</li>
</ul>

<p><strong>Confluence</strong> – Unstructured knowledge documents (Markdown/HTML)</p>

<ul>
<li>Houses our Customer Complaints Management Policy and Complaint Handling Framework</li>
<li>Serves as the RAG knowledge base for intelligent recommendations</li>
</ul>

<p><strong>The Power of Unified Access:</strong><br>
With MindsDB, all four sources in this vertical line of business (tabular databases, API-driven platforms, and document repositories) are accessible through the same SQL interface. Your team can query call transcripts, check Salesforce metrics, search Confluence docs, and review Jira tickets without switching contexts or learning multiple query languages.</p>

<p><strong>Example Connection:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">DATABASE</span> <span class="n">banking_postgres_db</span>
<span class="k">WITH</span> <span class="n">ENGINE</span> <span class="o">=</span> <span class="s1">'postgres'</span><span class="p">,</span>
<span class="k">PARAMETERS</span> <span class="o">=</span> <span class="p">{</span>
    <span class="nv">"host"</span><span class="p">:</span> <span class="nv">"host.docker.internal"</span><span class="p">,</span>
    <span class="nv">"port"</span><span class="p">:</span> <span class="mi">5432</span><span class="p">,</span>
    <span class="nv">"database"</span><span class="p">:</span> <span class="nv">"demo"</span><span class="p">,</span>
    <span class="nv">"user"</span><span class="p">:</span> <span class="nv">"postgresql"</span><span class="p">,</span>
    <span class="nv">"password"</span><span class="p">:</span> <span class="nv">"psqlpasswd"</span><span class="p">,</span>
    <span class="nv">"schema"</span><span class="p">:</span> <span class="nv">"demo_data"</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>Validation:</strong><br>
Confirm each connection with a simple SELECT query (e.g., <code>SELECT * FROM banking_postgres_db.conversations_summary LIMIT 50;</code>) to verify schemas and data availability before building downstream agents and models.</p>
<h3>
  
  
  <strong>2) Build the Knowledge Base (RAG): Zero-Infrastructure Context Retrieval</strong>
</h3>

<p>We create a Knowledge Base directly within MindsDB by ingesting targeted Confluence pages. This approach eliminates the entire RAG infrastructure stack that would normally require weeks of engineering effort.</p>

<p><strong>What MindsDB's Knowledge Base Replaces:</strong></p>

<p>Traditional RAG implementations require orchestrating multiple components:</p>

<ul>
<li>
<strong>Vector database deployment</strong> (Pinecone, Weaviate, Chroma, etc.)</li>
<li>
<strong>Embedding pipeline</strong> (API calls, batching, rate limiting, error handling)</li>
<li>
<strong>Custom retrieval logic</strong> (similarity search, reranking, prompt construction)</li>
<li>
<strong>Data sources infrastructure</strong> (crawlers, change detection, incremental updates)</li>
<li>
<strong>Access control &amp; monitoring</strong> (security layers, usage tracking, debugging tools)</li>
</ul>

<p>With MindsDB, all of this collapses into a single SQL workflow. Define the Knowledge Base once, and every agent can automatically leverage it for context-aware responses without separate infrastructure or custom code.</p>

<p><strong>Creating the Knowledge Base:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="n">KNOWLEDGE_BASE</span> <span class="n">my_confluence_kb</span>
<span class="k">USING</span>
    <span class="n">embedding_model</span> <span class="o">=</span> <span class="p">{</span>
        <span class="nv">"provider"</span><span class="p">:</span> <span class="nv">"openai"</span><span class="p">,</span>
        <span class="nv">"model_name"</span><span class="p">:</span> <span class="nv">"text-embedding-3-small"</span><span class="p">,</span>
        <span class="nv">"api_key"</span><span class="p">:</span><span class="nv">""</span>
    <span class="p">},</span>
    <span class="n">content_columns</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'body_storage_value'</span><span class="p">],</span>
    <span class="n">id_column</span> <span class="o">=</span> <span class="s1">'id'</span><span class="p">;</span>

<span class="k">DESCRIBE</span> <span class="n">KNOWLEDGE_BASE</span> <span class="n">my_confluence_kb</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>Ingesting Confluence Content:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="n">my_confluence_kb</span> <span class="p">(</span>
    <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">title</span><span class="p">,</span> <span class="n">body_storage_value</span>
    <span class="k">FROM</span> <span class="n">my_confluence</span><span class="p">.</span><span class="n">pages</span>
    <span class="k">WHERE</span> <span class="n">id</span> <span class="k">IN</span> <span class="p">(</span><span class="s1">'360449'</span><span class="p">,</span><span class="s1">'589825'</span><span class="p">)</span>
<span class="p">);</span>

<span class="k">SELECT</span> <span class="k">COUNT</span><span class="p">(</span><span class="o">*</span><span class="p">)</span> <span class="k">as</span> <span class="n">total_rows</span> <span class="k">FROM</span> <span class="n">my_confluence_kb</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>What Happens Behind the Scenes:</strong><br>
MindsDB automatically handles document chunking, embedding generation, vector indexing, and retrieval optimization. </p>

<p>To update content, simply re-run the INSERT statement—either manually, via scheduled SQL jobs within MindsDB, or triggered from your Python backend on a cron schedule. Once created, this Knowledge Base becomes a shared resource. Any agent in your workflow can reference it without duplicating retrieval logic or managing separate context windows.</p>

<h3>
  
  
  <strong>3) Define AI Agents:</strong> (Summary/Classification/Recommendation)
</h3>

<p>We deploy two specialized agents that handle the complete analytical workflow, from raw transcript analysis to context-aware recommendations.</p>

<p><strong>Agent Architecture:</strong></p>

<p><strong>Summary &amp; Classification Agent</strong></p>

<ul>
<li>
<strong>Input:</strong> Raw call transcript from PostgreSQL</li>
<li>
<strong>Task:</strong> Generate a concise summary + classify as <code>Resolved</code> or <code>Unresolved</code>
</li>
<li>
<strong>Output:</strong> Structured results written to Salesforce via our Python backend
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="n">AGENT</span> <span class="n">classification_agent</span>
<span class="k">USING</span>
    <span class="k">data</span> <span class="o">=</span> <span class="p">{</span>
        <span class="nv">"tables"</span><span class="p">:</span> <span class="p">[</span><span class="nv">"banking_postgres_db.conversations_summary"</span><span class="p">]</span>
    <span class="p">},</span>
    <span class="n">prompt_template</span> <span class="o">=</span> <span class="s1">'You are an analyst specializing in banking customer support interactions. Review the conversation transcript and produce:
1. A concise 2–3 sentence summary of the interaction.
2. A determination of the issue resolution status (RESOLVED or UNRESOLVED).
RESOLUTION CRITERIA:
UNRESOLVED (choose UNRESOLVED if ANY of the following are true):
- The customer expresses dissatisfaction, frustration, or complaints.
- The agent only provides reassurance, escalation, or requests for follow-up without delivering a solution.
- The customer questions or challenges bank policies, procedures, or communication.
- The conversation ends without a clear solution acknowledged by the customer.
- The customer side of the conversation is missing or incomplete.
- The agent proposes a solution, but the customer does not explicitly confirm acceptance.
- The conversation is cut off, incomplete, or ends abruptly.
RESOLVED:
- Only label as RESOLVED if the customer clearly acknowledges satisfaction.
- Acceptable confirmation cues include explicit statements such as:
  "thank you", "that worked", "this resolves my issue", "problem solved", "perfect", or similar positive closure.
OUTPUT FORMAT (strictly follow this format):
Summary: &lt;your 2–3 sentence summary&gt;
Status: &lt;RESOLVED or UNRESOLVED&gt;'</span><span class="p">,</span>
    <span class="n">timeout</span> <span class="o">=</span> <span class="mi">30</span><span class="p">;</span>
</code></pre>

</div>



<p><strong>RAG-Powered Recommendation Agent</strong></p>

<ul>
<li>
<strong>Trigger:</strong> Activated only for <code>Unresolved</code> cases</li>
<li>
<strong>Task:</strong> Query the Confluence Knowledge Base for relevant policies, then generate actionable Jira ticket content (title, description, acceptance criteria)</li>
<li>
<strong>Output:</strong> Structured recommendations written to Jira via our Python backend
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="n">AGENT</span> <span class="n">recommendation_agent</span>
<span class="k">USING</span>
    <span class="n">model</span> <span class="o">=</span> <span class="p">{</span>
        <span class="nv">"provider"</span><span class="p">:</span> <span class="nv">"openai"</span><span class="p">,</span>
        <span class="nv">"model_name"</span><span class="p">:</span> <span class="nv">"gpt-4o"</span><span class="p">,</span>
        <span class="nv">"api_key"</span><span class="p">:</span> <span class="nv">""</span>
    <span class="p">},</span>
    <span class="k">data</span> <span class="o">=</span> <span class="p">{</span>
        <span class="nv">"knowledge_bases"</span><span class="p">:</span> <span class="p">[</span><span class="nv">"mindsdb.my_confluence_kb"</span><span class="p">]</span>
    <span class="p">},</span>
      <span class="n">prompt_template</span> <span class="o">=</span> <span class="s1">'You are a Banking Customer Issue Resolution Consultant. Your task is to provide clear and actionable next-step recommendations for UNRESOLVED customer cases.
You have access to my_confluence_kb, which contains official internal procedures, escalation workflows, and complaint handling guidelines. 
Use the knowledge base as your primary reference source and DO NOT invent policies.
OUTPUT REQUIREMENTS:
- Only output a numbered action plan.
- Each action must be specific and operational, not general or vague.
- Avoid emotional, generic, or training-like statements (e.g., "improve communication" / "show empathy").
- If no direct procedural reference is found in the knowledge base, include action: "Escalate following standard unresolved case workflow."
OUTPUT FORMAT (strictly follow this):
Recommended Actions:
1. &lt;clear operational step&gt;
2. &lt;clear operational step&gt;
3. &lt;clear operational step&gt;'</span>

</code></pre>

</div>



<p><strong>Why Define Agents in MindsDB?</strong></p>

<p>Traditional AI workflows require maintaining separate infrastructure for prompt management, vector retrieval, model orchestration, output parsing, and observability logging. MindsDB collapses all of this into declarative SQL definitions. The recommendation agent, for example, automatically retrieves context from the Knowledge Base with no custom code. You just specify which knowledge base to use. Every agent interaction is traceable through the MindsDB UI: prompts, retrieved passages, model responses, and outputs are all inspectable, which is essential for debugging and compliance in regulated industries. Agents become reusable components queryable like standard tables: <code>SELECT * FROM classification_agent WHERE conversation_id = '12345'</code>.</p>

<h2>
  
  
  Results
</h2>

<ul>
<li>
<strong>End-to-end automation:</strong> Complete workflow from call transcript ingestion to Jira ticket creation with zero manual intervention.</li>
<li>
<strong>84% time reduction:</strong> Cases that previously required 15+ minutes of manual triage, summarization, and ticket creation now complete in under 2 minutes.</li>
<li>
<strong>Production-ready architecture:</strong> MindsDB connects to enterprise data sources with standard authentication; our Python backend controls all write operations to Salesforce and Jira, maintaining security boundaries.</li>
<li>
<strong>Rapid development:</strong> Full prototype built and deployed in 48 hours using MindsDB's declarative agent framework without custom RAG infrastructure or embedding pipelines.</li>
</ul>

<h2>
  
  
  <strong>What We Learned</strong>
</h2>

<p>MindsDB's SQL-driven framework made debugging and prompt tuning significantly faster. Instead of parsing application logs, we could query agent execution history like any database table and analyze patterns across hundreds of runs to refine prompts quickly.</p>

<p><strong>Model Selection Strategy</strong></p>

<p>We matched models to operational needs. The summary and classification agent processes high volumes with tight latency requirements, so we use smaller, efficient models (<a href="https://openrouter.ai/openai/gpt-5-nano" rel="noopener noreferrer">gpt-5-nano</a>, <a href="https://openrouter.ai/openai/gpt-oss-120b" rel="noopener noreferrer">gpt-oss-120b</a>). The recommendation agent handles fewer cases but requires deeper reasoning over complex policy documents, so we use more capable models (<a href="https://openrouter.ai/openai/gpt-5-pro" rel="noopener noreferrer">gpt-5-pro</a>, <a href="https://ai.google.dev/gemini-api/docs/models#gemini-2.5-pro" rel="noopener noreferrer">gemini-2.5-pro</a>) to reduce misaligned outputs that need human correction.</p>

<p><strong>Explainability &amp; Traceability</strong></p>

<p>MindsDB's UI shows the complete execution trace: input, agent reasoning in the "Thoughts" panel, knowledge base queries, and retrieved data in the "Observation" panel. When outputs miss the mark, you can see exactly which knowledge base or table content influenced the decision. This transparency is essential for compliance audits and far superior to black-box implementations.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0mqtx7e6o21a70xwlhf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0mqtx7e6o21a70xwlhf.png" alt="mindsdb thought process" width="800" height="541"></a></p>

