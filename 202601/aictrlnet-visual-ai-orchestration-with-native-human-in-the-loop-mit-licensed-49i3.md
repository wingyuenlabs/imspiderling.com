---
Title: AICtrlNet: Visual AI Orchestration with Native Human-in-the-Loop (MIT Licensed)
Description: 
Author: Srirajasekhar Koritala
Date: 2026-01-29T21:34:48.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This post was originally published on the <a href="https://aictrlnet.com/blog/2026/01/introducing-aictrlnet/" rel="noopener noreferrer">AICtrlNet blog</a>.</em></p>

<p>We're excited to announce AICtrlNet, an open core AI orchestration platform that treats humans and AI as equal participants in workflows—not afterthoughts.</p>

<p>The Community Edition is MIT licensed and available today on <a href="https://github.com/Bodaty/aictrlnet-community" rel="noopener noreferrer">GitHub</a> and <a href="https://pypi.org/project/aictrlnet/" rel="noopener noreferrer">PyPI</a>.</p>

<h2>
  
  
  The Problem We Kept Running Into
</h2>

<p>Over the past few years, we've built AI systems for enterprises across healthcare, finance, and legal. Every project hit the same wall: <strong>AI workflows that work in demos fail in production because they ignore humans.</strong></p>

<p>The existing tools fell into two camps:</p>

<p><strong>Code-first frameworks</strong> (LangChain, CrewAI, AutoGen) are powerful but assume developers will handle everything programmatically. There's no visual way to design workflows, no built-in governance, and adding human approval steps means building custom infrastructure.</p>

<p><strong>Visual automation tools</strong> (n8n, Zapier, Dify) make it easy to connect things, but AI is bolted on—not native. When you need a human to review an AI decision before it executes, you're back to building custom solutions.</p>

<p>We needed something that didn't exist: <strong>visual workflow design with native human-in-the-loop capabilities and real governance controls.</strong></p>

<p>So we built it.</p>

<h2>
  
  
  What AICtrlNet Does
</h2>

<p>AICtrlNet is an orchestration engine that coordinates AI agents, human workers, and external systems into auditable workflows.</p>

<h3>
  
  
  Visual Workflow Design
</h3>

<p>Design workflows visually with HitLai (our React-based UI), or programmatically via API. Your choice.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[AI: Generate Report] → [Human: Review &amp; Approve] → [AI: Distribute] → [Human: Confirm Delivery]
</code></pre>

</div>



<p>Every node can be an AI model, a human task, or an external service. The engine handles routing, state management, and execution.</p>

<h3>
  
  
  Human-in-the-Loop Native
</h3>

<p>This is the core differentiator. Humans aren't a fallback for when AI fails—they're first-class workflow participants.</p>

<ul>
<li>Define approval workflows with escalation paths</li>
<li>Set up human validation checkpoints</li>
<li>Route tasks based on confidence scores</li>
<li>Track human decisions with full audit trails
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Example: AI generates content, human reviews before publishing
</span><span class="n">workflow</span> <span class="o">=</span> <span class="nc">Workflow</span><span class="p">(</span>
    <span class="n">nodes</span><span class="o">=</span><span class="p">[</span>
        <span class="nc">AINode</span><span class="p">(</span><span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gpt-4</span><span class="sh">"</span><span class="p">,</span> <span class="n">task</span><span class="o">=</span><span class="sh">"</span><span class="s">generate_content</span><span class="sh">"</span><span class="p">),</span>
        <span class="nc">HumanNode</span><span class="p">(</span><span class="n">role</span><span class="o">=</span><span class="sh">"</span><span class="s">content_reviewer</span><span class="sh">"</span><span class="p">,</span> <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">approve_or_reject</span><span class="sh">"</span><span class="p">),</span>
        <span class="nc">ConditionalNode</span><span class="p">(</span>
            <span class="n">if_approved</span><span class="o">=</span><span class="nc">AINode</span><span class="p">(</span><span class="n">task</span><span class="o">=</span><span class="sh">"</span><span class="s">publish</span><span class="sh">"</span><span class="p">),</span>
            <span class="n">if_rejected</span><span class="o">=</span><span class="nc">AINode</span><span class="p">(</span><span class="n">task</span><span class="o">=</span><span class="sh">"</span><span class="s">revise</span><span class="sh">"</span><span class="p">)</span>
        <span class="p">)</span>
    <span class="p">]</span>
<span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  AI Governance Built-In
</h3>

<p>We've seen what happens when AI workflows run without guardrails. AICtrlNet includes:</p>

<ul>
<li>5-layer AI Workflow Security Gateway</li>
<li>Bias detection and monitoring</li>
<li>Complete audit trails (who did what, when, why)</li>
<li>Compliance framework support (HIPAA, GDPR, SOC2)</li>
</ul>

<p>This isn't an enterprise upsell—it's in the Community Edition.</p>

<h3>
  
  
  Model Context Protocol (MCP) Support
</h3>

<p>Native MCP integration for standardized AI model communication. Connect any MCP-compatible model or service without writing custom adapters.</p>

<h2>
  
  
  The Editions
</h2>

<p>We're releasing AICtrlNet as open core:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Edition</th>
<th>What You Get</th>
<th>License</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Community</strong></td>
<td>Core orchestration engine, essential adapters, governance controls</td>
<td>MIT</td>
</tr>
<tr>
<td><strong>Business</strong></td>
<td>+ HitLai visual UI, ML-enhanced features, RAG, 43 industry packs</td>
<td>Commercial</td>
</tr>
<tr>
<td><strong>Enterprise</strong></td>
<td>+ Multi-tenancy, federation, SSO, white-label</td>
<td>Commercial</td>
</tr>
</tbody>
</table></div>

<p>The Community Edition is genuinely MIT licensed. Not "fair-code," not "source-available with restrictions." MIT. Fork it, modify it, build commercial products on it.</p>

<h2>
  
  
  How It Compares
</h2>

<p>We're not trying to replace your existing tools—we integrate with them.</p>

<p><strong>vs. LangChain</strong>: We use LangChain under the hood for AI execution. AICtrlNet adds visual orchestration, HITL, and governance on top. Use both.</p>

<p><strong>vs. n8n</strong>: n8n is great for traditional automation. We integrate with n8n for its 400+ connectors. AICtrlNet handles the AI-native workflows where governance and human oversight matter.</p>

<p><strong>vs. Dify/Flowise</strong>: Great visual AI builders. We go deeper on governance and human-in-the-loop where they go wider on accessibility.</p>

<p><strong>vs. CrewAI</strong>: CrewAI orchestrates AI teams. We orchestrate AI + humans. Different problem spaces.</p>

<h2>
  
  
  Quick Start
</h2>

<p><strong>Docker (recommended)</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/Bodaty/aictrlnet-community.git
<span class="nb">cd </span>aictrlnet-community
docker-compose up <span class="nt">-d</span>
<span class="c"># API at http://localhost:8000</span>
</code></pre>

</div>



<p><strong>pip</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>aictrlnet
</code></pre>

</div>



<p><strong>Your first workflow</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:8000/api/v1/workflows <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "name": "Review Workflow",
    "nodes": [
      {"type": "ai", "model": "gpt-4", "task": "analyze"},
      {"type": "human", "role": "reviewer", "action": "approve"}
    ]
  }'</span>
</code></pre>

</div>



<p>Full documentation: <a href="https://aictrlnet.com/docs" rel="noopener noreferrer">aictrlnet.com/docs</a></p>

<h2>
  
  
  What's Next
</h2>

<p>We're actively developing:</p>

<ul>
<li>More adapters (community requests welcome)</li>
<li>Enhanced MCP capabilities</li>
<li>Visual workflow designer improvements</li>
<li>True multi-tenant SaaS (Enterprise)</li>
</ul>

<p>Have an idea? <a href="https://github.com/Bodaty/aictrlnet-community/issues/new?template=feature_request.md" rel="noopener noreferrer">Open a feature request</a> and let us know what matters most.</p>




<p>If you found this useful, drop a reaction and follow <a href="https://dev.to/bobbykoritala">@bobbykoritala</a> for updates on AICtrlNet development.</p>

<ul>
<li>Star us: <a href="https://github.com/Bodaty/aictrlnet-community" rel="noopener noreferrer">github.com/Bodaty/aictrlnet-community</a>
</li>
<li>Join the conversation: <a href="https://github.com/Bodaty/aictrlnet-community/discussions" rel="noopener noreferrer">GitHub Discussions</a>
</li>
<li>Try it: <code>pip install aictrlnet</code>
</li>
</ul>

