---
Title: MCPs for Developers Who Think They Don't Need MCPs
Description: 
Author: Angie Jones
Date: 2025-11-30T21:50:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>Lately, I've seen more developers online starting to side eye MCP. There was a <a href="https://x.com/ibuildthecloud/status/1990221860018204721" rel="noopener noreferrer">tweet</a> by Darren Shepherd that summed it up well:</p>

<blockquote>
<p>"Most devs were introduced to MCP through coding agents (Cursor, VSCode) and most devs struggle to get value out of MCP in this use case... so they are rejecting MCP because they have a CLI and scripts available to them which are way better for them."</p>
</blockquote>

<p>Fair. Most developers were introduced to MCPs through some chat-with-your-code experience, and sometimes it doesn't feel better than just opening your terminal and using the tools you know. But here's the thing...</p>

<p><strong>MCPs weren't built just for developers.</strong></p>

<p>They're not just for IDE copilots or code buddies. At Block, we use MCPs across <em>everything</em>, from finance to design to legal to engineering. <a href="https://youtu.be/IDWqWdLESgY?si=Mjoi-MGEPW9sxvmT" rel="noopener noreferrer">I gave a whole talk</a> on how different teams are using goose, an AI agent. The point is MCP is a protocol. What you build on top of it can serve all kinds of workflows.</p>

<p>But I get it... let's talk about the dev-specific ones that <em>are</em> worth your time.</p>

<h2>
  
  
  GitHub: More Than Just the CLI
</h2>

<p>If your first thought is "why would I use <a href="https://github.com/github/github-mcp-server" rel="noopener noreferrer">GitHub MCP</a> when I have the CLI?" I hear you. GitHub's MCP is kind of bloated right now. (They know. They're working on it.)</p>

<p>But also: <strong>you're thinking too local.</strong></p>

<p>You're imagining a solo dev setup where you're in your terminal, using GitHub CLI to do your thing. And honestly, if all you’re doing is opening a PR or checking issues, you probably should use the CLI.</p>

<p>But the CLI was never meant to coordinate across tools. It’s built for local, linear commands. But what if your GitHub interactions happened <em>somewhere else</em> entirely? </p>

<p>MCP shines when your work touches multiple systems like GitHub, Slack, and Jira without you stitching it together.</p>

<p>Here's a real example from our team:</p>

<blockquote>
<p>Slack thread. Real developers in realtime.</p>

<p><strong>Dev 1:</strong> I think there's a bug with xyz</p>

<p><strong>Dev 2:</strong> Let me check... yep, I think you're right.</p>

<p><strong>Dev 3:</strong> <code>@goose</code> is there a bug here?</p>

<p><strong>goose:</strong> Yep. It's in these lines...[code snippet]</p>

<p><strong>Dev 3:</strong> Okay <code>@goose</code>, open an issue with the details. What solutions would you suggest?</p>

<p><strong>goose:</strong> Here are 3 suggestions: [code snippets with rationale]</p>

<p><strong>Dev 1:</strong> I like Option 1</p>

<p><strong>Dev 2:</strong> me too</p>

<p><strong>Dev 3:</strong> <code>@goose</code>, implement Option 1</p>

<p><strong>goose:</strong> Done. Here's the PR.</p>
</blockquote>

<p>All of that happened <em>in Slack</em>. No one opened a browser or a terminal. No one context switched. Issue tracking, triaging, discussing fixes, implementing code in one thread in a 5-minute span.</p>

<p>We've also got teams tagging Linear or Jira tickets and having goose fully implement them. One team had goose do <strong>15 engineering days</strong> worth of work in a single sprint. The team literally ran out of tasks and had to pull from future sprints. Twice!</p>

<p>So yes, GitHub CLI is great. But MCP opens the door to workflows where GitHub isn't the only place where dev work happens. That's a shift worth paying attention to.</p>

<h2>
  
  
  Context7: Docs That Aren't Dated
</h2>

<p>Here's another pain point developers hit: documentation.</p>

<p>You're working with a new library. Or integrating an API. Or wrestling with an open source tool. </p>

<p>The <a href="https://github.com/upstash/context7" rel="noopener noreferrer">Context7 MCP</a> pulls up-to-date docs, code examples, and guides right into your AI agent's brain. You just ask questions and get answers like:</p>

<ul>
<li>"How do I create a payment with the Square SDK?"</li>
<li>"What's the auth flow for Firebase?"</li>
<li>"Is this library tree-shakable?"</li>
</ul>

<p>It doesn't rely on stale LLM training data from two years ago. It scrapes the source of truth <em>right now</em>. Giving it updated... say it with me... CONTEXT.</p>

<p>Developer "flow" is real, and every interruption steals precious focus time. This MCP helps you figure out new libraries, troubleshoot integrations, and get unstuck without leaving your IDE. </p>

<h2>
  
  
  Repomix: Know the Whole Codebase Without Reading It
</h2>

<p>Imagine you join a new project or want to contribute to an open source one, but it's a huge repo with lots of complexity.</p>

<p>Instead of poking around for hours trying to draw an architectural diagram in your head, you just ask your agent:</p>

<blockquote>
<p>"goose, pack this project up."</p>
</blockquote>

<p>It runs <a href="https://github.com/yamadashy/repomix" rel="noopener noreferrer">repomix</a>, which compresses the entire codebase into an AI-optimized file. From there, your convo might go like this:</p>

<ul>
<li>"Where's the auth logic?"</li>
<li>"Show me how API calls work."</li>
<li>"What uses <code>UserContext</code>?"</li>
<li>"What's the architecture?"</li>
<li>"What's still a TODO?"</li>
</ul>

<p>You get direct answers with context, code snippets, summaries, and suggestions. It's like onboarding with a senior dev who already knows everything. Sure, you could grep around and piece things together. But repomix gives you the whole picture - structure, metrics, patterns - compressed and queryable.</p>

<p>And it even works with remote public GitHub repos, so you don't need to clone anything to start exploring.</p>

<p>This is probably my favorite dev MCP. It's a huge time saver for new projects, code reviews, and refactoring.</p>

<h2>
  
  
  Chrome DevTools MCP: Web Testing While You Code
</h2>

<p>The <a href="https://github.com/ChromeDevTools/chrome-devtools-mcp" rel="noopener noreferrer">Chrome DevTools MCP</a> is a must-have for frontend devs. You're building a new form/widget/page/whatever. Instead of opening your browser, typing stuff in, and clicking around, you just tell your agent:</p>

<blockquote>
<p>"Test my login form on localhost:3000. Try valid and invalid logins. Let me know what happens."</p>
</blockquote>

<p>Chrome opens, test runs, screenshots captured, network traffic logged, console errors noted. All done by the agent.</p>

<p>This is gold for frontend devs who want to actually test their work before throwing it over the fence.</p>




<p>Could you script all this with CLIs and APIs? Sure, if you want to spend your weekend writing glue code. But why would you want to do that when MCP gives you that power right out of the box... in any MCP client?!</p>

<p>So no, MCPs are not overhyped. They're how you plug AI into everything you use: Slack, GitHub, Jira, Chrome, docs, codebases - and make that stuff work <em>together</em> in new ways.</p>

<p>Recently, Anthropic called out the <a href="https://www.anthropic.com/engineering/advanced-tool-use" rel="noopener noreferrer">real issue</a>: most dev setups load tools naively, bloat the context, and confuse the model. It's not the protocol that's broken. It's that most people (and agents) haven't figured out how to use it well yet. Fortunately, goose has - it <a href="https://block.github.io/goose/docs/mcp/extension-manager-mcp" rel="noopener noreferrer">manages MCPs by default</a>, enabling and disabling as you need them. </p>

<p>But I digress.</p>

<p>Step outside the IDE, and that's when you really start to see the magic.</p>

<p>P.S. Happy first birthday, MCP! 🎉</p>

