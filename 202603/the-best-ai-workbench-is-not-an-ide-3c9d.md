---
Title: The best AI workbench is not an IDE
Description: 
Author: Nithin
Date: 2026-03-21T21:31:28.000Z
Robots: noindex,nofollow
Template: index
---
<p>There is a common beginner mistake in AI-assisted development: picking a single tool and asking it to do everything.</p>

<p>That is how people end up defending editors as if they were operating systems.</p>

<p>An editor is not a strategy. It is a window.</p>

<p>What matters is the execution surface behind the window: authentication, tool access, agent reliability, MCP support, and whether team knowledge can survive a change of shell.</p>

<p>After using Cursor, Cursor CLI, IntelliJ IDEA with AI Assistant wired to external providers, IntelliJ with Kilo Code, and Codex CLI, I arrived at a conclusion that is not fashionable but is practical:</p>

<p>IntelliJ should remain the workbench. Codex should become the primary agent. Cursor should be mined for what it does well, mostly skills and rules. Kilo Code should be demoted to utility work such as autocomplete and commit message generation.</p>

<p>This is not a benchmark of clever demos. It is a benchmark of operational friction.</p>

<p>This is also a personal conclusion, not a team mandate. The point is not to move the team to a new stack. The point is to build a personal workbench that works best for me while still adapting cleanly to the team's existing Cursor-based conventions.</p>

<h2>
  
  
  Short verdict
</h2>

<p>If the goal is a practical daily setup rather than a vendor bet, the cleanest arrangement is:</p>

<ul>
<li>IntelliJ IDEA for editing, navigation, refactoring, and inspections.</li>
<li>Codex CLI as the primary agent runtime.</li>
<li>Cursor rules and skills preserved through <code>AGENTS.md</code>.</li>
<li>Kilo Code used narrowly for autocomplete and commit-message assistance.</li>
</ul>

<p>That conclusion is not based on abstract model preference. It is based on where authentication, MCP access, and team workflow actually held up under daily use.</p>

<h2>
  
  
  Cursor is strongest when you stay inside Cursor
</h2>

<p>Cursor has one undeniable advantage: if your team has already invested in Cursor rules and skills, the product rewards conformity.</p>

<p>That matters.</p>

<p>Institutional knowledge beats raw model quality surprisingly often. A mediocre prompt with strong team conventions can outperform a brilliant model used in a vacuum.</p>

<p>Cursor authentication also keeps you inside the ecosystem where those skills are first-class citizens. If the team works through Cursor-native skills, Cursor the IDE feels coherent. The tool, the auth model, and the team workflow all point in the same direction.</p>

<p>But this coherence has a price: it is local to Cursor.</p>

<p>The moment you try to separate the agent from the IDE, the promise weakens.</p>

<p>Cursor's own CLI documentation says MCP in the CLI uses the same configuration as the editor and that configured servers should work in both surfaces. In theory, that is elegant. In practice, the theory leaks. In a Cursor forum thread from January 21, 2026, a team member explicitly acknowledged a known issue: <code>cursor-agent</code> could not handle OAuth correctly for remote MCP servers such as Figma, even though the IDE could. The same thread was still receiving reports on March 18, 2026.<sup id="fnref1">1</sup> <sup id="fnref2">2</sup></p>

<p>For workflows that depend on remote OAuth-backed MCPs, that limitation is enough to keep Cursor CLI out of the primary slot.</p>

<p>That matters because an increasing amount of useful agent work now sits behind authenticated remote tools rather than local prompts alone.</p>

<h2>
  
  
  The real test is OAuth, not chat quality
</h2>

<p>The market is still pretending that model selection is the hard problem.</p>

<p>It is not.</p>

<p>The hard problem is whether the agent can reach the systems that matter without turning your desk into a museum of half-working clients.</p>

<p>Figma is the cleanest example.</p>

<p>Figma's remote MCP server requires OAuth. Their official documentation says so directly and includes a Codex CLI flow that prompts for authentication after <code>codex mcp add figma --url https://mcp.figma.com/mcp</code>.<sup id="fnref3">3</sup></p>

<p>There is also no useful escape hatch through personal access tokens. In a Figma community thread opened on November 18, 2025, a user asked for PAT-based auth for the remote MCP endpoint. A quoted reply from Figma Support said the remote MCP server does not support authentication using personal access tokens and that this cannot be enabled; OAuth is the recommended path.<sup id="fnref4">4</sup></p>

<p>That closes the usual loophole.</p>

<p>If a client does not handle OAuth correctly, the integration is not merely inconvenient. It is functionally unavailable.</p>

<p>This is why Cursor CLI is disqualified for my primary workflow today.</p>

<p>Not because Cursor is weak.</p>

<p>Because Figma is unforgiving.</p>

<p>Notion belongs in the same category. Remote MCP access is valuable only when the client can complete the auth dance cleanly and repeatably. A tool that supports MCP on a slide but fails during OAuth is doing theater, not engineering.</p>

<h2>
  
  
  IntelliJ is still the best room in the house
</h2>

<p>IntelliJ IDEA remains the best physical workspace.</p>

<p>It is still the superior editor for large codebases, code navigation, inspections, refactors, and plain old concentration. I do not want to keep both Cursor IDE and IntelliJ open just to preserve one niche capability from each.</p>

<p>That setup is not power-user sophistication. It is a tax.</p>

<p>JetBrains has moved aggressively on AI Assistant and MCP. Their current documentation shows AI Assistant can connect to MCP servers over STDIO, Streamable HTTP, and SSE.<sup id="fnref5">5</sup> JetBrains also allows third-party providers and OpenAI-compatible endpoints, but this path is explicitly API-key driven, and feature coverage varies by provider.<sup id="fnref6">6</sup></p>

<p>That distinction matters.</p>

<p>An IDE feature that can <em>consume a model</em> is not automatically equivalent to a first-class <em>agent runtime</em>. The former is a panel. The latter is an operating surface.</p>

<p>Figma's own remote MCP installation page is a useful reality check here. As of March 22, 2026, it provides setup documentation for Claude Code, Codex by OpenAI, Cursor, and VS Code. IntelliJ is not listed among the supported client setup guides.<sup id="fnref7">7</sup></p>

<p>That does not prove IntelliJ can never be made to work with remote MCP. It does show where official support and documentation currently concentrate.</p>

<p>In my setup, IntelliJ AI Assistant wired to external providers did not become the stable answer for OAuth-backed remote MCP workflows. The practical result was still inferior to using a dedicated agent client that treats authentication and tool invocation as primary concerns rather than attached features.</p>

<p>So IntelliJ stays.</p>

<p>But it stays as the cockpit, not as the pilot.</p>

<h2>
  
  
  Codex CLI solves the right problem
</h2>

<p>Codex CLI succeeded where the supposedly more "native" combinations became awkward.</p>

<p>The decisive point was simple: OAuth-backed MCP worked.</p>

<p>Figma remote MCP worked properly. Notion MCP worked properly. The boring but essential part of the stack, sign-in, token flow, callback handling, tool availability, simply held together.</p>

<p>That is what maturity looks like in agent tooling: fewer broken rituals between intention and execution.</p>

<p>Once Codex handled remote MCP cleanly, the rest of the architecture became obvious.</p>

<p>Use IntelliJ as the IDE.</p>

<p>Use Codex as the primary agent runtime.</p>

<p>Keep Cursor knowledge without keeping Cursor as the center of gravity.</p>

<h2>
  
  
  The bridge matters more than the model
</h2>

<p>The most expensive part of switching tools is not the subscription.</p>

<p>It is the loss of team memory.</p>

<p>That is why the <code>AGENTS.md</code> bridge is the most important piece of this workbench.</p>

<p>By teaching Codex to treat <code>.cursor/rules/*.mdc</code> and <code>.cursor/skills/**/SKILL.md</code> as source-of-truth instructions, the setup stops being ideological and becomes mechanical. Cursor skills remain usable. Cursor rules remain usable. Team conventions survive. My workbench changes; the shared operating doctrine does not.</p>

<p>This is the right way to personalize AI tooling inside a team without forcing the team to follow.</p>

<p>Do not rewrite every rule because a vendor changed.</p>

<p>Do not ask the team to learn a second set of rituals.</p>

<p>Move the execution engine locally. Preserve the doctrine globally.</p>

<p>That is exactly why Codex is a better primary workflow than Cursor IDE in this setup: it can inherit the team's Cursor knowledge while also handling the remote MCP reality that Cursor CLI still fumbles.</p>

<h2>
  
  
  The bug that weakens trust
</h2>

<p>There is another reason not to make Cursor IDE the center of the system: reliability debt.</p>

<p>I have seen occasional cases where agent changes do not appear correctly in the Git changelist inside Cursor. There are workarounds, restarts, and folklore. I have not found a permanent fix worth trusting a primary workflow to.</p>

<p>Intermittent bugs are how tools slowly lose authority. A development surface must be boring when it comes to source control visibility. If the user has to wonder whether a file changed or merely failed to appear, the editor has ceased to be infrastructure and has become weather.</p>

<h2>
  
  
  Comparison in practice
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Setup</th>
<th>What worked</th>
<th>What broke or stayed weak</th>
<th>Verdict</th>
</tr>
</thead>
<tbody>
<tr>
<td>Cursor IDE</td>
<td>Cursor-native rules and skills, coherent in-product workflow</td>
<td>Git changelist visibility sometimes unreliable, requires staying in Cursor to get the full benefit</td>
<td>Useful, but not my center of gravity</td>
</tr>
<tr>
<td>Cursor CLI</td>
<td>Reuses Cursor MCP configuration and team skills in principle</td>
<td>Remote MCP OAuth remained unreliable for the workflows that mattered most</td>
<td>Not suitable as primary runtime</td>
</tr>
<tr>
<td>IntelliJ + AI Assistant with external providers</td>
<td>Excellent IDE, strong editing and refactoring environment, growing MCP support</td>
<td>Not listed in Figma's official remote MCP client setup docs; also did not become the most reliable home for remote OAuth-backed MCP workflows in my setup</td>
<td>Best editor, not best agent runtime</td>
</tr>
<tr>
<td>Codex CLI</td>
<td>Remote MCP with OAuth worked cleanly for Figma and Notion; agent workflow felt dependable</td>
<td>Less value from Cursor-native ecosystem unless bridged deliberately</td>
<td>Best primary runtime</td>
</tr>
<tr>
<td>Kilo Code</td>
<td>Helpful for lightweight assistance</td>
<td>Not the tool I would trust for heavy MCP-centric workflows</td>
<td>Keep it narrow</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  The resulting workbench
</h2>

<p>The final arrangement is not elegant in the way marketing teams prefer. It is elegant in the way a good wrench is elegant.</p>

<p>The workbench is:</p>

<ul>
<li>IntelliJ IDEA for editing, navigation, refactoring, and staying in flow.</li>
<li>Codex CLI as the primary agent for serious work, especially when remote MCP with OAuth is involved.</li>
<li>Cursor CLI as a secondary utility when team-specific Cursor workflows are still useful, but not as the foundation.</li>
<li>Kilo Code for lightweight assistance such as autocomplete and commit message generation through OpenAI auth.</li>
</ul>

<p>This division of labor is healthy.</p>

<p>Each tool is forced to do only the job it is actually good at.</p>

<p>That is rare in modern AI tooling, where every vendor wants to be your editor, shell, browser, memory layer, and therapist at the same time.</p>

<h2>
  
  
  Conclusion
</h2>

<p>The winning setup is not Cursor versus Codex versus IntelliJ.</p>

<p>That framing is too small.</p>

<p>The real contest is between two philosophies.</p>

<p>One philosophy wants a single branded surface to own the whole experience.</p>

<p>The other treats the development environment as a federation: one editor, one primary agent runtime, a few specialized helpers, and a thin compatibility layer to preserve team knowledge.</p>

<p>The second philosophy wins because it respects reality better.</p>

<p>Today, that means IntelliJ IDEA as the room, Codex CLI as the worker, Cursor rules and skills as inherited doctrine, and Kilo Code as a useful but limited assistant.</p>

<p>The best AI workbench is not the one with the most features.</p>

<p>It is the one that authenticates, executes, and gets out of the way.</p>




<h2>
  
  
  Sources
</h2>

<ul>
<li>Cursor CLI MCP docs: <a href="https://docs.cursor.com/cli/mcp" rel="noopener noreferrer">https://docs.cursor.com/cli/mcp</a>
</li>
<li>Cursor forum, "Cursor CLI - Figma support," January 21, 2026: <a href="https://forum.cursor.com/t/cursor-cli-figma-support/149491" rel="noopener noreferrer">https://forum.cursor.com/t/cursor-cli-figma-support/149491</a>
</li>
<li>Figma MCP remote server docs: <a href="https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/" rel="noopener noreferrer">https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/</a>
</li>
<li>Figma forum, PAT auth request for remote MCP, November 18, 2025: <a href="https://forum.figma.com/ask-the-community-7/support-for-pat-personal-access-token-based-auth-in-figma-remote-mcp-47465" rel="noopener noreferrer">https://forum.figma.com/ask-the-community-7/support-for-pat-personal-access-token-based-auth-in-figma-remote-mcp-47465</a>
</li>
<li>JetBrains AI Assistant MCP docs: <a href="https://www.jetbrains.com/help/ai-assistant/mcp.html" rel="noopener noreferrer">https://www.jetbrains.com/help/ai-assistant/mcp.html</a>
</li>
<li>JetBrains AI Assistant custom model docs: <a href="https://www.jetbrains.com/help/ai-assistant/use-custom-models.html" rel="noopener noreferrer">https://www.jetbrains.com/help/ai-assistant/use-custom-models.html</a>
</li>
</ul>




<ol>

<li id="fn1">
<p>Cursor documentation says the CLI shares MCP configuration with the editor and exposes <code>cursor-agent mcp login</code> for authentication. ↩</p>
</li>

<li id="fn2">
<p>Cursor staff acknowledged on January 21-22, 2026 that <code>cursor-agent</code> had a known OAuth issue for remote MCP servers such as Figma and GitHub, with additional user reports still appearing on March 18, 2026. ↩</p>
</li>

<li id="fn3">
<p>Figma's remote MCP documentation says the remote server requires OAuth and includes a Codex CLI setup path that prompts for authentication. ↩</p>
</li>

<li id="fn4">
<p>This relies on a quoted support reply in the Figma community thread, so treat it as support guidance surfaced through the forum rather than a standalone product-spec page. ↩</p>
</li>

<li id="fn5">
<p>JetBrains AI Assistant documentation currently lists STDIO, Streamable HTTP, and SSE as supported transports for MCP servers. ↩</p>
</li>

<li id="fn6">
<p>JetBrains documents third-party provider integration through API keys and notes that some feature availability depends on provider and model support. ↩</p>
</li>

<li id="fn7">
<p>On March 22, 2026, Figma's remote MCP installation page listed setup guides for Claude Code, Codex by OpenAI, Cursor, and VS Code, but not IntelliJ. ↩</p>
</li>

</ol>

