---
Title: The Great Decoupling: The Enterprise Capability Graph
Description: 
Author: Jay Desmarais
Date: 2026-01-16T21:33:54.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Part 2 of "The Great Decoupling" series</em></p>

<p>In <a href="https://dev.to/gd-tech-guru/the-great-decoupling-the-end-of-frozen-interactions-48f2">Part 1</a>, I argued that we're witnessing the decoupling of capability from presentation — that the SaaS interface, far from being the product, is becoming an ephemeral rendering layer generated on demand. MCP has emerged as the enabling standard for this shift, proving the pattern works and catalyzing industry-wide adoption.</p>

<p>But that's just the first domino.</p>

<p>Here's the question that kept nagging at me: if external SaaS products expose capabilities via MCP, why wouldn't internal enterprise systems do the same? And if they do, what happens to the boundary between "our software" and "their software"?</p>

<p>The answer, I've come to believe, is that the boundary dissolves. Internal and external systems become equivalent nodes in a unified capability graph. And that changes everything about how enterprises think about architecture, vendors, and control.</p>

<h2>
  
  
  The Symmetry No One's Talking About
</h2>

<p>Most discussions of MCP focus on AI agents calling external services — your assistant querying your CRM, pulling data from your document platform, triggering workflows in your automation tools. That's valuable, but it's only half the picture.</p>

<p>The pattern works identically for internal systems. Your custom ERP. Your homegrown analytics pipeline. That sprawling collection of internal tools your platform team maintains. Each of these can expose capabilities through the same protocol.</p>

<p>Once they do, something interesting happens. The orchestration layer — whatever routes requests, manages authentication, handles authorization — stops caring about the origin of capabilities. It routes to <code>get_customer_credit_limit</code> without knowing or caring whether that capability lives in an internal Oracle instance or an external Experian API.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4ddkc79r96wbgaisfspu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4ddkc79r96wbgaisfspu.png" alt="Enterprise Orchestrator" width="800" height="533"></a></p>

<p><em>(Meridian is a fictional CRM — a stand-in for the dominant platforms you're already thinking of. The pattern applies regardless of vendor.)</em></p>

<p>Internal and external become implementation details. The enterprise sees a unified graph of capabilities, some provided internally, some externally, all accessible through the same interface.</p>

<p>This symmetry has profound implications.</p>

<h2>
  
  
  Build vs. Buy Transforms
</h2>

<p>The traditional "build vs. buy" decision pits two options against each other: build a feature internally (control but cost) or purchase a product with a UI your employees learn (faster but lock-in). The comparison involves not just functionality but also interfaces, training, integration work, and long-term dependency.</p>

<p>Capability-first architecture transforms this into <strong>expose vs. consume</strong>.</p>

<p>Should we build this capability internally or consume it from an external provider? The integration is identical either way — same protocol, same orchestration layer, same developer experience. The evaluation becomes purely about the capability itself: cost, quality, reliability, compliance, and specialization.</p>

<p>This doesn't eliminate the decision's difficulty, but it changes its shape. You're no longer comparing "our custom tool with our custom UI" against "their product with their UI that we need to train everyone on." You're comparing capability providers on capability merits.</p>

<p>And critically, the decision becomes more reversible. If you start with an external capability provider and later decide to build internally, the integration doesn't change. If you build internally and later find a better external option, same story. The orchestration layer abstracts the origin.</p>

<p>This reversibility is strategic gold in the current environment, where the right answer keeps changing.</p>

<h2>
  
  
  Capability Architecture as Chaos Survival
</h2>

<p>Let me paint a picture of the current enterprise reality.</p>

<p>AI capabilities are emerging faster than evaluation cycles can process them. The vendor landscape shifts monthly — acquisitions, pivots, new entrants, sudden deprecations. Build vs. buy decisions that seemed sound six months ago look questionable today. Integration complexity explodes as each new AI tool requires its own connection pattern. Technical debt accumulates from point-to-point connections that made sense at the time.</p>

<p>Every enterprise architect I talk to describes some version of this chaos. The ground won't stop shifting.</p>

<p>Capability-based architecture directly addresses this reality:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Challenge</th>
<th>Capability Architecture Response</th>
</tr>
</thead>
<tbody>
<tr>
<td>Rapid AI evolution</td>
<td>Swap capability providers without rewiring consumers</td>
</tr>
<tr>
<td>Vendor uncertainty</td>
<td>Reduce lock-in via standard interfaces</td>
</tr>
<tr>
<td>Build/buy fluidity</td>
<td>Internal and external capabilities integrate identically</td>
</tr>
<tr>
<td>Integration complexity</td>
<td>Single protocol, not N×M point-to-point connections</td>
</tr>
<tr>
<td>Technical debt</td>
<td>Clean abstractions prevent integration spaghetti</td>
</tr>
</tbody>
</table></div>

<p>This isn't about preparing for some distant future. It's about surviving the present. The patterns that make your architecture maintainable today — isolation, explicit contracts, standard interfaces — are exactly what you need to navigate an environment where the right answer keeps changing.</p>

<p>The pitch to enterprises isn't "adopt this for the future." It's "adopt this to stay agile now."</p>

<h2>
  
  
  The Orchestration Layer Emerges
</h2>

<p>As enterprises deploy capability-based architectures, a new layer crystallizes: the <strong>orchestrator</strong>.</p>

<p>This isn't just an API gateway or a service mesh, though it shares DNA with both. The orchestrator handles:</p>

<p><strong>Capability discovery</strong>: What capabilities are available? What can this identity access? The orchestrator maintains the registry and handles dynamic discovery.</p>

<p><strong>Request routing</strong>: When a request comes in, where does it go? The orchestrator routes based on capability type, data residency requirements, load, cost optimization, or custom rules.</p>

<p><strong>Authentication and authorization</strong>: Can this identity invoke this capability with these parameters on this data? The orchestrator enforces access control consistently across internal and external capabilities.</p>

<p><strong>Audit and observability</strong>: What was invoked, when, by whom, with what parameters, returning what results? The orchestrator maintains the audit trail that compliance requires.</p>

<p><strong>Context management</strong>: What's the session state? What permissions were delegated? What constraints apply? The orchestrator maintains context across capability invocations.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjbvgiqlzv53dw41md16f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjbvgiqlzv53dw41md16f.png" alt=" " width="800" height="533"></a></p>

<p>The orchestrator becomes the enterprise's control plane for capabilities. Not the capabilities themselves — those remain distributed across internal systems and external providers — but the layer that makes them accessible, governable, and composable.</p>

<p>This is a new category of infrastructure. Not quite an integration platform (though it handles integration). Not quite an AI framework (though it enables AI). Not quite identity management (though it handles identity). It's the connective tissue of the capability-first enterprise.</p>

<h2>
  
  
  What Platform Teams Become
</h2>

<p>If this architecture emerges, the role of enterprise platform teams transforms.</p>

<p>Today, platform teams manage specific systems. "We own the Meridian instance." "We maintain the internal analytics platform." "We run the integration middleware." Each system is a distinct responsibility with its own expertise.</p>

<p>In the capability-first model, platform teams become <strong>capability curators</strong>. They don't just maintain systems — they curate the capability graph. Their responsibilities shift:</p>

<p><strong>From</strong>: Managing the CRM instance<br>
<strong>To</strong>: Ensuring CRM capabilities are available, performant, and properly governed — regardless of whether those capabilities come from Meridian, a competitor, or internal implementation</p>

<p><strong>From</strong>: Building integration pipelines between systems<br>
<strong>To</strong>: Defining capability contracts and ensuring providers (internal or external) meet them</p>

<p><strong>From</strong>: Training users on specific application UIs<br>
<strong>To</strong>: Ensuring capabilities are discoverable and well-documented for AI and human consumption alike</p>

<p><strong>From</strong>: Negotiating vendor contracts for products<br>
<strong>To</strong>: Evaluating capability providers on capability merits — quality, reliability, cost, compliance</p>

<p>This is a meaningful elevation. Platform teams move from system administrators to capability brokers, from tool maintainers to graph architects.</p>

<h2>
  
  
  The Enterprise AI Fabric
</h2>

<p>Here's where I see this converging. The combination of:</p>

<ul>
<li>Unified capability graph (internal and external systems equivalent)</li>
<li>Orchestration layer (discovery, routing, auth, audit)</li>
<li>Contextual rendering (interfaces generated on demand)</li>
<li>AI agents (as primary capability consumers)</li>
</ul>

<p>...constitutes something new. Call it the <strong>Enterprise AI Fabric</strong> — the connective layer that makes AI-native operations possible.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fartvduavqnxvw80whf0v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fartvduavqnxvw80whf0v.png" alt="Contextual Rendering" width="800" height="533"></a></p>

<p>This fabric enables scenarios that are currently painful or impossible:</p>

<p><strong>Cross-system operations without integration projects</strong>: "Update the customer record, adjust their credit limit, and notify the account team" becomes a single orchestrated flow across CRM, financial system, and communication platform — without building a custom integration.</p>

<p><strong>Graceful capability substitution</strong>: When a vendor raises prices or degrades quality, swap to an alternative without consumer-side changes. The orchestrator routes differently; everything else continues.</p>

<p><strong>AI agents with appropriate enterprise access</strong>: Instead of giving AI tools direct API keys to everything (security nightmare) or nothing (useless), the orchestrator mediates access with proper authorization and audit.</p>

<p><strong>Federated capabilities across organizational boundaries</strong>: Partners, suppliers, and customers can expose capabilities into your graph (with appropriate access controls), enabling inter-organization workflows without point-to-point integrations.</p>

<h2>
  
  
  The Forcing Function Is Already Here
</h2>

<p>I want to be clear: this isn't a ten-year vision. The forcing function is already present.</p>

<p>Every major enterprise is grappling with AI adoption. Every one of them is hitting the same walls — how do we give AI access to our systems safely? How do we avoid building N×M integrations? How do we maintain governance while enabling experimentation?</p>

<p>Capability-based architecture is the answer emerging from these pressures. MCP — or whatever it evolves into — is the enabling standard crystallizing from the chaos. The orchestration layer is what enterprises build when they realize they need to manage capabilities systematically.</p>

<p>MCP's current form isn't the point. The point is that it's proven the pattern works, the major players have aligned around it, and the architectural direction is now clear. What MCP looks like in 2028 may be quite different from today — but the doors it opened won't close.</p>

<p>The question isn't whether this architecture emerges. It's whether your enterprise leads or follows.</p>




<h2>
  
  
  But There's an Uncomfortable Question
</h2>

<p>If capabilities become standardized and interchangeable... if interfaces become ephemeral rendering layers... if internal and external systems become equivalent nodes in a graph... where does value live?</p>

<p>For two decades, SaaS vendors have built moats around their data. Your CRM doesn't just provide CRM capabilities — it accumulates your institutional memory. Every interaction, every deal progression, every customer relationship pattern. That's not a feature. That's a hostage.</p>

<p>The capability-first architecture makes this hostage-taking visible. When the orchestrator asks for customer data and the capability responds with friction designed to keep data inside vendor walls, the customer notices.</p>

<p>This leads somewhere uncomfortable for vendors and potentially liberating for enterprises: the data sovereignty question we've been deferring for twenty years.</p>

<p>That's Part 3.</p>




<p><em>Next in the series: **The Great Decoupling: The Data Sovereignty Correction</em>* — How capability-first architecture inverts the SaaS power structure*</p>




<h2>
  
  
  References
</h2>

<ol>
<li><p><strong>Enterprise AI Agent Adoption</strong>: G2's August 2025 survey found 57% of companies have AI agents in production, with another 22% in the pilot phase. PwC research shows 79% of organizations have adopted AI agents to some extent. Source: <a href="https://guptadeepak.com/the-complete-guide-to-model-context-protocol-mcp-enterprise-adoption-market-trends-and-implementation-strategies/" rel="noopener noreferrer">Gupta Deepak - MCP Enterprise Adoption Guide</a></p></li>
<li><p><strong>Multi-Agent System Designs</strong>: 66.4% of enterprises use multi-agent system designs rather than single-agent approaches, creating demand for coordination protocols. Source: <a href="https://guptadeepak.com/the-complete-guide-to-model-context-protocol-mcp-enterprise-adoption-market-trends-and-implementation-strategies/" rel="noopener noreferrer">Gupta Deepak - MCP Enterprise Adoption Guide</a></p></li>
<li><p><strong>Agentic AI Project Challenges</strong>: Gartner predicts over 40% of agentic AI projects will be canceled by the end of 2027 due to unclear ROI and implementation challenges. Source: <a href="https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027" rel="noopener noreferrer">Gartner Press Release</a></p></li>
<li><p><strong>AWS Agentic AI Security Framework</strong>: AWS published the Agentic AI Security Scoping Matrix, defining four security scopes from basic tool use to fully autonomous systems. Source: <a href="https://aws.amazon.com/blogs/security/the-agentic-ai-security-scoping-matrix-a-framework-for-securing-autonomous-ai-systems/" rel="noopener noreferrer">AWS Security Blog</a></p></li>
<li><p><strong>AI Agent Security Concerns</strong>: Gartner predicts 25% of enterprise breaches will trace back to AI agent abuse by 2028. Source: <a href="https://www.cio.com/article/4024106/autonomous-ai-agents-autonomous-security-risk.html" rel="noopener noreferrer">CIO - Autonomous AI Agents = Autonomous Security Risk</a></p></li>
<li><p><strong>MCP and A2A Protocols</strong>: Analysis of how MCP (tool integration) and Google's A2A (agent-to-agent coordination) are positioned as complementary rather than competing standards. Source: <a href="https://www.koyeb.com/blog/a2a-and-mcp-start-of-the-ai-agent-protocol-wars" rel="noopener noreferrer">Koyeb - A2A and MCP: Start of the AI Agent Protocol Wars?</a></p></li>
</ol>

