---
Title: The Mistakes Didn't Change. The Speed Did.
Description: 
Author: Felix Ortiz
Date: 2026-03-28T21:42:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>Everyone is measuring how fast agents write code. Few are measuring what that code introduces.</p>

<p>This year, independent researchers tested the major AI coding agents building applications from scratch. <a href="https://www.helpnetsecurity.com/2026/03/13/claude-code-openai-codex-google-gemini-ai-coding-agent-security/" rel="noopener noreferrer">Most pull requests contained at least one vulnerability</a>. Inside Fortune 50 companies, AI-generated code <a href="https://apiiro.com/blog/4x-velocity-10x-vulnerabilities-ai-coding-assistants-are-shipping-more-risks/" rel="noopener noreferrer">introduced 10,000+ new security findings per month</a>. Logic and syntax bugs went down. Privilege escalation paths jumped over 300%. Yikes!</p>

<p>The code improved while the vulnerabilities got worse. Agents just produce the same old mistakes faster. One customer seeing another customer's data. Login flows that leave a back door wide open. Endpoints exposed to the entire internet.</p>

<h2>
  
  
  The mistakes are harder to see
</h2>

<p>The code looks clean. It follows the right patterns, uses the right frameworks, passes initial agent-driven code review. It just quietly skips the check that asks "should this user be allowed to do this?" or "has this request been authenticated?"</p>

<p>These are judgment mistakes. The security tooling most teams rely on was built to catch known-bad patterns, not missing logic. <a href="https://www.dryrun.security/blog/top-ai-sast-tools-2026" rel="noopener noreferrer">Over 80% of vulnerabilities in AI-generated code go undetected by traditional static analysis</a>. Pattern matching catches code that is obviously wrong. It cannot catch logic that is missing.</p>

<h2>
  
  
  The window to catch them is closing
</h2>

<p>A human writes one insecure endpoint per sprint. An agent writes twenty in an afternoon. That alone changes the math on how much your security infrastructure needs to handle.</p>

<p>It goes further. The agentic loop is getting tighter. Agents writing code, agents reviewing code, agents merging code. Each iteration shrinks the window between generation and production, and the human verification layer gets thinner every time.</p>

<p>When that window was wide, pattern-matching tools and human reviewers could compensate for each other's blind spots. As it narrows, both have less time to work with, and the mistakes that slip through are the ones neither was designed to catch.</p>

<h2>
  
  
  The tooling is evolving, and so is the attack surface
</h2>

<p>The next generation of security tooling is starting to reason about code rather than just match patterns. Security review as a continuous practice embedded in the development loop, not a gate at the end of it. That direction is right.</p>

<p>More tooling also means more surface area. Earlier this year, <a href="https://www.heyuan110.com/posts/ai/2026-03-10-mcp-security-2026/" rel="noopener noreferrer">a wave of CVEs hit MCP infrastructure</a>, many of them the same class of vulnerability these tools exist to catch. If you are going to trust your security pipeline, you have to secure the pipeline itself. OWASP and GitHub are already publishing <a href="https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/" rel="noopener noreferrer">frameworks</a> and <a href="https://github.blog/ai-and-ml/generative-ai/under-the-hood-security-architecture-of-github-agentic-workflows/" rel="noopener noreferrer">reference architectures</a> for this.</p>

<h2>
  
  
  What I'm doing about it
</h2>

<p>On my own platform, I have the pattern-matching layer in place. Static analysis on every PR, dynamic scanning nightly. That catches what it was designed to catch. The floor is built.</p>

<p>Now I need what goes above it: security agents that reason about logic-level gaps, tooling integrated at generation time via MCP instead of just at review time, and a hardened pipeline that gets the same isolation and least-privilege treatment as production.</p>

<p>The mistakes agents make are not new. The speed at which they make them is.</p>

