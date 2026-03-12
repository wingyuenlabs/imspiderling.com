---
Title: Bridging the Gap - How We Made AI Agents 10x Developers in Our Organization
Description: 
Author: Sagiv ben giat
Date: 2026-03-12T22:10:51.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Problem: AI Agents Don't Know Your Organization
</h2>

<p>AI coding assistants have revolutionized how we write code. They can generate functions, debug errors, and suggest optimizations in seconds. The promise is clear: become a 10x developer with AI as your co-pilot.</p>

<p>But here's the reality check: the moment your AI agent needs to work with your organization's internal infrastructure, the magic breaks down.</p>

<p>Your AI agent doesn't know:</p>

<ul>
<li>Which design system your team uses</li>
<li>How your CI/CD pipelines are structured</li>
<li>What internal libraries you have for logging, metrics, and tracing</li>
<li>Your organization's coding standards and conventions</li>
<li>How to manage your staging environments</li>
<li>Which services exist and how they interact</li>
<li>Your deployment workflows and requirements</li>
</ul>

<p>Instead of a multiplier, you get a drag. Instead of a driver, you get a passenger asking for directions every few minutes.</p>

<p>This isn't a frontend problem. It's not specific to any one company. <strong>This is a universal challenge every organization faces when adopting AI development tools.</strong></p>

<h2>
  
  
  The Root Cause: Context is King
</h2>

<p>AI agents are incredibly powerful, but they're only as good as the context they have. When working with public libraries and frameworks, they have access to extensive training data, documentation, and examples. But your internal tools? Your organizational standards? That's a black box to them.</p>

<p>The traditional solution has been to manually paste documentation, explain architecture in chat, or let the AI fumble through trial and error. This works for simple tasks, but it doesn't scale. It certainly doesn't make you 10x more productive.</p>

<h2>
  
  
  Our Solution: Teaching AI Agents About Our Organization
</h2>

<p>In our organization, we tackled this problem head-on for our frontend infrastructure. Our solution has three key components:</p>

<h3>
  
  
  1. Comprehensive Documentation Layer
</h3>

<p>We created structured documentation that covers:</p>

<ul>
<li>
<strong>Architecture docs</strong>: How our multi-service portal architecture works, request flows, and system design</li>
<li>
<strong>Conventions</strong>: Naming standards, coding patterns, and best practices</li>
<li>
<strong>Workflows</strong>: Translation pipelines, deployment checklists, and contribution guidelines</li>
<li>
<strong>Visual diagrams</strong>: Mermaid diagrams showing request flows, build pipelines, and deployment processes</li>
</ul>

<p>This isn't just README files scattered across repositories. It's a curated, interconnected knowledge base designed specifically for AI consumption.</p>

<h3>
  
  
  2. Living Project Catalog
</h3>

<p>We maintain a catalog of all our frontend infrastructure projects with rich metadata:</p>

<ul>
<li>Project types (services, libraries, templates)</li>
<li>Technology stacks</li>
<li>npm package names</li>
<li>Internal dependencies</li>
<li>Documentation links</li>
<li>Ownership information</li>
</ul>

<p>This gives AI agents a complete map of our ecosystem - what exists, how it's connected, and where to find more information.</p>

<h3>
  
  
  3. MCP Server: The Bridge Between AI and Infrastructure
</h3>

<p>The magic happens through a Model Context Protocol (MCP) server - a standardized way for AI agents to access external tools and data sources.</p>

<p>Our MCP server provides AI agents with specialized tools:</p>

<p><strong>Discovery Tools:</strong></p>

<ul>
<li>
<code>list_projects</code>: Browse all available frontend infrastructure projects</li>
<li>
<code>get_project_metadata</code>: Get detailed information about specific projects including dependencies, tech stack, and documentation links</li>
</ul>

<p><strong>Documentation Tools:</strong></p>

<ul>
<li>
<code>get_architecture_docs</code>: Fetch architecture documentation, diagrams, conventions, and workflows</li>
<li>
<code>get_documentation</code>: Access project-specific documentation and guides</li>
</ul>

<p><strong>Source Code Tools:</strong></p>

<ul>
<li>
<code>list_project_files</code>: Explore repository structure and file organization</li>
<li>
<code>get_project_files</code>: Fetch source code from any project (with proper authentication)</li>
</ul>

<p><strong>Scaffolding Tools:</strong></p>

<ul>
<li>
<code>clone_template</code>: Get instructions to scaffold new services from approved templates</li>
</ul>

<p>All of this happens securely through Okta authentication, ensuring that AI agents only access what developers are authorized to see.</p>

<h2>
  
  
  How It Works in Practice
</h2>

<p>Here's a real-world scenario:</p>

<p><strong>Developer:</strong> "Create a new frontend service that displays user analytics with charts"</p>

<p><strong>Without MCP Server:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AI: I'll create a React app with Chart.js...
Developer: No, we use our internal design system
AI: Which design system?
Developer: Our component library
AI: How do I import it?
Developer: *pastes package name and import examples*
AI: How should I structure the service?
Developer: *explains our multi-service architecture*
AI: How do I deploy it?
Developer: *explains deployment pipeline, service registration, manifest files, etc.*
</code></pre>

</div>



<p><strong>With MCP Server:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AI: Let me check your frontend infrastructure...
    [Calls list_projects to discover available tools]
    [Calls get_architecture_docs to understand the system]
    [Calls get_project_metadata to learn about the component library]
    [Calls clone_template to get the service template]

    I'll scaffold a new service using your standard template,
    integrate with your component library, and set up
    the deployment configuration following your conventions.
</code></pre>

</div>



<p>The AI agent autonomously discovers what it needs to know, follows organizational standards, and produces production-ready code that fits seamlessly into your infrastructure.</p>

<h2>
  
  
  The Impact
</h2>

<p>With this approach, organizations can achieve:</p>

<ol>
<li>
<strong>Reduced onboarding time</strong>: New developers can start contributing faster because AI agents guide them through organizational standards</li>
<li>
<strong>Consistent code quality</strong>: AI agents follow conventions automatically</li>
<li>
<strong>Fewer production issues</strong>: AI agents know about critical deployment requirements that developers might miss</li>
<li>
<strong>Faster feature development</strong>: Developers spend less time explaining context and more time building features</li>
</ol>

<h2>
  
  
  The Bigger Picture: Every Organization Needs This
</h2>

<p>Our implementation is specific to our frontend infrastructure, but the pattern is universal. Every organization with internal tools, platforms, and standards faces this challenge.</p>

<p>The Model Context Protocol provides a standardized way to solve it. Whether you're building:</p>

<ul>
<li>Backend services with internal frameworks</li>
<li>Mobile apps with custom SDKs</li>
<li>Infrastructure with proprietary tools</li>
<li>Data pipelines with internal platforms</li>
</ul>

<p>You need a way to teach AI agents about your organization's unique context.</p>

<h2>
  
  
  Key Takeaways
</h2>

<ol>
<li>
<strong>AI agents are powerful, but context-blind</strong>: They need explicit access to your organizational knowledge</li>
<li>
<strong>Documentation alone isn't enough</strong>: You need a structured, machine-readable way to expose information</li>
<li>
<strong>MCP worked for us</strong>: Model Context Protocol provided a proven pattern for extending AI agents - it might work for you too, or you might find a different implementation that fits your needs</li>
<li>
<strong>Security matters</strong>: Proper authentication ensures AI agents respect access controls</li>
<li>
<strong>Start small, iterate</strong>: Begin with your most critical pain points and expand over time</li>
</ol>

<h2>
  
  
  Getting Started
</h2>

<p>If you're interested in building something similar for your organization, start by identifying real pain points:</p>

<ol>
<li>
<strong>Test with real tasks</strong>: Ask your AI agent to perform common tasks in your organization - "Create a new service", "Add authentication to this endpoint", "Set up monitoring for this component"</li>
<li>
<strong>Identify missing knowledge</strong>: When the AI fails or produces incorrect code, note what organizational knowledge it's missing - Is it the deployment process? Your internal libraries? Coding conventions?</li>
<li>
<strong>Add context manually first</strong>: Provide the missing information directly in the chat and verify the AI can complete the task successfully with that context</li>
<li>
<strong>Gather 2-3 use cases</strong>: Repeat this process for different scenarios to understand the pattern of what knowledge AI agents need most</li>
<li>
<strong>Build tools for those use cases</strong>: Once you have clear patterns, create MCP tools that provide that knowledge programmatically - start with tools that address your most common pain points</li>
<li>
<strong>Validate with original tasks</strong>: Test that your tools actually help the AI agent solve the tasks from your use cases without manual intervention</li>
</ol>

<p>This approach ensures you're building tools that solve real problems rather than creating infrastructure that might not get used.</p>

<p>The future of AI-assisted development isn't just about smarter models - it's about giving those models the right context to understand your organization. When you bridge that gap, AI agents truly become the 10x multiplier they promise to be.</p>




<p><em>Want to learn more about the Model Context Protocol? Check out <a href="https://modelcontextprotocol.io" rel="noopener noreferrer">modelcontextprotocol.io</a> for the specification and examples.</em></p>




<p><em>Originally published on <a href="https://debuggr.io" rel="noopener noreferrer">debuggr.io</a>.</em></p>

<p><em>I write about software engineering, AI, and the things that keep bugging me about our industry. If this resonated with you, come visit <a href="https://debuggr.io" rel="noopener noreferrer">debuggr.io</a> for more.</em></p>

