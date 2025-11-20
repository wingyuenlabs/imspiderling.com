---
Title: Skills, MCPs, and Commands are the same context engineering trend.
Description: 
Author: Cesar Vega
Date: 2025-11-20T21:54:01.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  What are skills:
</h1>

<p>Skills are folders of instructions and scripts that Claude loads to do specialized tasks. <strong>The point of skills is to make Claude follow guidelines and be more deterministic.</strong></p>

<h2>
  
  
  Benefits.
</h2>

<ul>
<li>More deterministic</li>
<li>Easy to edit, share, and set up.</li>
<li>Efficient context use.*</li>
</ul>

<h1>
  
  
  Make AI more deterministic.
</h1>

<p>This is one of the biggest issues with LLMs right now: they spout a lot of code, and they can and will come up with two different approaches and answers to the same prompt. With skills, we add guardrails, instructions, and scripts that tell the LLM exactly what to do. For example, the <code>playwright-skill</code> I downloaded has a <code>helpers.js</code> file which has a bunch of scripts like <code>launchBrowser</code> or <code>safeClick</code>, so Claude can use those functions and the result will always be the same.<br>
    * This was also a benefit that MCPs had.</p>
<h1>
  
  
  Easy to edit.
</h1>

<p>Since Skills are just <code>.md</code> files and scripts, they are really easy to edit and share if necessary. The downside is that the files can be hard to maintain if they don't have a single source of truth.</p>
<h1>
  
  
  Context
</h1>
<h2>
  
  
  Understanding Claude context
</h2>

<p>Running <code>/context</code> in Claude code (And in most other CLIs) gives you a breakdown of how context is used in the CLI:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp97jaxtc7j1bvj5y8akm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp97jaxtc7j1bvj5y8akm.png" alt=" " width="800" height="241"></a></p>

<p>The context is divided into:</p>

<ul>
<li>System prompt: The instructions set that defines Claude's identity, behavior, and core operational guidelines

<ul>
<li>Skills are loaded here.</li>
</ul>
</li>
<li>System tools: Built-in capabilities available in every conversation (file operations, execution, web access, agents).</li>
<li>MCP tools: MCP tools and descriptions of each tool.</li>
<li>Memory files: Project-specific context files (like CLAUDE.md) that provide persistent knowledge about your projects and preferences.</li>
<li>Messages: The actual conversation history between you and Claude in the current session.</li>
<li>Free space: Available context window capacity for additional content.</li>
<li>Autocompact buffer: Reserved space that triggers automatic context compression when filled to prevent overflow.</li>
</ul>
<h2>
  
  
  Skills context
</h2>

<p>Claude's path when using skills works like this:</p>

<p>At the start of a session, Claude scans all available skill files and <strong>reads a short explanation for each one from the frontmatter YAML in the Markdown file</strong> - this is very token efficient: each skill only takes up a few dozen extra tokens, with the full details only loaded in should the user request a task that the skill can help solve.</p>

<p>For example, at the very beginning of each session, Claude generates a system context file. In which skills are loaded in a section like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   &lt;available_skills&gt;
     &lt;skill&gt;
       &lt;name&gt;analyzing-financial-statements&lt;/name&gt;
       &lt;description&gt;This skill calculates key financial ratios and metrics from financial statement data <span class="k">for </span>investment analysis
  <span class="o">(</span>project<span class="o">)</span>&lt;/description&gt;
       &lt;location&gt;managed&lt;/location&gt;
     &lt;/skill&gt;
     &lt;skill&gt;
       &lt;name&gt;example-skill&lt;/name&gt;
       &lt;description&gt;Demonstrates the basic structure of a Claude Skill. Use this as a template when creating new skills. <span class="o">(</span>project<span class="o">)</span>&lt;/description&gt;
       &lt;location&gt;managed&lt;/location&gt;
     &lt;/skill&gt;
     &lt;skill&gt;
       &lt;name&gt;playwright-skill&lt;/name&gt;
       &lt;description&gt;Complete browser automation with Playwright. Auto-detects dev servers, writes clean <span class="nb">test </span>scripts to /tmp. Test pages, fill forms,
  take screenshots, check responsive design, validate UX, <span class="nb">test </span>login flows, check links, automate any browser task. Use when user wants to <span class="nb">test
  </span>websites, automate browser interactions, validate web functionality, or perform any browser-based testing. <span class="o">(</span>project<span class="o">)</span>&lt;/description&gt;
       &lt;location&gt;managed&lt;/location&gt;
     &lt;/skill&gt;
   &lt;/available_skills&gt;
</code></pre>

</div>



<p><em>You can see an example of the whole prompts at leaked prompts at <a href="https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-4.5-sonnet.md" rel="noopener noreferrer">ageirtj's repo</a></em></p>

<p>All my skills take approximately 200-250 tokens (including XML tags, descriptions, and metadata). The actual descriptions alone (just the text content) would be 35~40 tokens total.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fffi80ak36mai7fykzb05.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fffi80ak36mai7fykzb05.webp" alt=" " width="800" height="234"></a></p>

<p>Claude only loads the .yaml-like description and data at the top of the document, and from there it decides if it's worth reading more (Progressive Disclosure):</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyz1rrh5eigpqvyi5vc2p.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyz1rrh5eigpqvyi5vc2p.webp" alt=" " width="800" height="450"></a></p>

<h3>
  
  
  NOTE: Skills depend on a coding environment
</h3>

<p>One thing to note is that the skills mechanism is entirely dependent on the model having access to a filesystem, tools to navigate it, and the ability to execute commands in that environment. The filesystem is your computer, and it can change things in YOUR computer. So you should always verify whether your functions or commands have side effects that could affect your system.</p>

<p>This can also be beneficial because you are not dealing with external dependencies, but it brings back the "It works in my computer" problem.</p>

<h1>
  
  
  Skills vs MCP
</h1>

<p>I mentioned that skills are not FULLY loaded upfront; they load the basic description, and then the agent decides whether it's worth reading more. MCPs used to be designed (And some still are) in a way that all of their tools were loaded up front, taking a lot of context. <strong>But new MCPs are improving their architecture, and they are using the same patterns as Skills to have separate code execution, which brings their context size to similar levels as Skills.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd5bqwy1shl9vw61na7hz.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd5bqwy1shl9vw61na7hz.webp" alt=" " width="800" height="450"></a></p>

<p><strong>Note: Technically, you can use scripts in skills and avoid MCPs. (And vice-versa)</strong><br>
You can use both together: MCP connections give Claude access to tools, while Skills teach Claude how to use those tools effectively.<br>
MCP connects Claude to external services and data sources. Skills provide instructions for how to complete specific workflows. </p>
<h2>
  
  
  Skills PROS:
</h2>

<ul>
<li>
<strong>Skills are simple</strong>: The core simplicity of the skills design is why I'm so excited about it.
MCP is a whole protocol specification, covering hosts, clients, servers, resources, prompts, tools, sampling, roots, elicitation, and three different transports (stdio, streamable HTTP, and originally SSE). Skills are Markdown with a tiny bit of YAML metadata and some optional scripts in whatever you can make executable in the environment.</li>
</ul>
<h2>
  
  
  MCP PROS:
</h2>

<ul>
<li>
<strong>Can connect to external applications</strong>: This is the biggest pro in my opinion; If you want an API-like interface that can connect to other applications, this is the way to go, especially since there are many officially supported MCPs from big organizations like Linear/Slack/Stripe. You can also benefit from MCP servers that run the code in the owner's cloud instead of locally.</li>
</ul>
<h2>
  
  
  Things to consider that I didn't delve into:
</h2>

<ul>
<li>Reliability, security/isolation, cross-device availability, latency, required runtime permissions, collaboration features, etc...</li>
</ul>
<h2>
  
  
  Plawright MCP vs skill example.
</h2>

<p><strong>DISCLAIMER:</strong> It's not the same maintainer, so some of the features in the MCP are not in the SKILL and vice versa.<br>
The Playwright skill uses around 100 tokens in context, while the MCP uses ONE HUNDRED times more (10'000 tokens). The main difference is that the Skill has a lot of the functionality the MCP has, but it gets loaded ONLY when it's necessary, while the MCP is always in memory.</p>

<p>Here is a breakdown of the approximate token use that Playwrite MCP uses:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  Based on the JSON schema definitions visible to me, here<span class="s1">'s a rough estimate:

  Simple tools (minimal parameters):
  - browser_close, browser_navigate_back, browser_install, browser_console_messages
  - ~150-250 tokens each

  Medium tools (moderate parameters):
  - browser_resize, browser_navigate, browser_press_key, browser_take_screenshot, browser_wait_for, browser_tabs
  - ~300-500 tokens each

  Complex tools (extensive parameters with nested schemas):
  - browser_evaluate, browser_fill_form, browser_click, browser_type, browser_drag, browser_hover, browser_select_option, browser_run_code
  - ~500-800 tokens each

  Rough total estimate: 8,000-10,000 tokens for all 22 Playwright MCP tools, including their full JSON schema definitions, descriptions, and parameters
  specifications, and metadata.
</span></code></pre>

</div>



<p>I will say this again, <strong>10'000 tokens in context for something we might not use is not worth it</strong>. I have personally disabled the Playwright MCP, but I had to change the Skill a bit to fit my work better (which is waay easier to do in Skills, since I have the files right there in my codebase).</p>

<h2>
  
  
  Semi-outdated MCP criticisms (In my opinion):
</h2>

<p>I saw this quote by Simon Willson, but I think the main point about spending tokens can be mitigated by good MCP design:<br>
    * "My own interest in MCPs has waned ever since I started taking coding agents seriously. Almost everything I might achieve with an MCP can be handled by a CLI tool instead. LLMs know how to call cli-tool --help, which means you don't have to spend many tokens describing how to use them—the model can figure it out later when it needs to."<br>
    * My comment about this is that there is a new trend with MCPs that do the same as Skills, they display a short description and then point to relevant files and scripts to be used. - But note: For most tasks, it's probably still better to use Skills.<br>
        - <a href="https://www.anthropic.com/engineering/code-execution-with-mcp" rel="noopener noreferrer">https://www.anthropic.com/engineering/code-execution-with-mcp</a></p>

<h1>
  
  
  What about RAG?
</h1>

<p>RAG is a concept that was very hyped before, and it's still relevant. RAG is a middle step where the agent searches and retrieves external data with whatever tools are available to it - grep, vector database, or any relevancy algorithm. </p>

<p>It can be similar to Skills or MCPs in the way that it's a tool that an LLM can use to improve its answer.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe7squqhnqltg4e88z6s0.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe7squqhnqltg4e88z6s0.jpg" alt=" " width="800" height="473"></a></p>

<h1>
  
  
  What about Claude commands?
</h1>

<p>Custom slash commands allow you to define frequently-used prompts as Markdown files that Claude Code can execute. There are plenty of pre-loaded commands in Claude code like <code>/context</code>, <code>/mcp</code>, <code>/github</code>, etc... <br>
They behave almost the same as skills in the way that they are some kind of script or prompt that the LLM runs when you tell it to. </p>

<p>It seems that commands haven't gained a lot of traction, and I think it's because they tend to be focused on small changes. I think of Skills as an orchestrator of commands. If a command is a single function or bash, skills are a group of functions or commands.</p>

<h1>
  
  
  Understanding that context is stateless,
</h1>

<p><em>We pass the whole conversation every time</em><br>
One thing that really helped me understand why keeping context clean is important is that LLMs are stateless; in every new message, we are just passing the whole conversation back and forth to the model (Maybe some providers optimize for this?). So every new message ends up being your whole conversation and the latest message attached to it. </p>

<p>Sending a single-line message at the end of a 400-line discussion with a lot of back-and-forth is a lot different than sending one at the end of a 100-line structured discussion. MCPs and Skills can help us keep this context as small as possible and even save their thoughts in concise documents that we can reference in a new chat with an LLM.</p>

<h1>
  
  
  Conclusion
</h1>

<p>Claude, it's becoming a general agent, an orchestrator. Skills, Commands, MCPs, and Agents make this explicit. And we as developers will have to focus a lot more on managing Context and providing guardrails, scripts, and instructions to restrict the LLM when necessary and let it work in what it does best.</p>

<p>Even when models are pushing context windows of 1 million tokens, they still struggle to keep track of things we have done in long conversations, so keeping things understandable and documented is still critical.</p>

<p>Feel free to send me sources and comments about this.</p>

<h1>
  
  
  Sources:
</h1>

<p><a href="https://support.claude.com/en/articles/12512176-what-are-skills" rel="noopener noreferrer">https://support.claude.com/en/articles/12512176-what-are-skills</a><br>
<a href="https://simonwillison.net/2025/Oct/16/claude-skills/" rel="noopener noreferrer">https://simonwillison.net/2025/Oct/16/claude-skills/</a><br>
<a href="https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills" rel="noopener noreferrer">https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills</a><br>
<a href="https://github.com/lackeyjb/playwright-skill?tab=readme-ov-file" rel="noopener noreferrer">https://github.com/lackeyjb/playwright-skill?tab=readme-ov-file</a><br>
<a href="https://news.ycombinator.com/item?id=45619537" rel="noopener noreferrer">https://news.ycombinator.com/item?id=45619537</a><br><br>
<a href="https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-4.5-sonnet.md" rel="noopener noreferrer">https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-4.5-sonnet.md</a><br>
<a href="https://www.anthropic.com/engineering/code-execution-with-mcp" rel="noopener noreferrer">https://www.anthropic.com/engineering/code-execution-with-mcp</a><br>
<a href="https://aws.amazon.com/what-is/retrieval-augmented-generation/" rel="noopener noreferrer">https://aws.amazon.com/what-is/retrieval-augmented-generation/</a><br>
<a href="https://code.claude.com/docs/en/slash-commands" rel="noopener noreferrer">https://code.claude.com/docs/en/slash-commands</a><br>
<a href="https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents" rel="noopener noreferrer">https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents</a></p>

