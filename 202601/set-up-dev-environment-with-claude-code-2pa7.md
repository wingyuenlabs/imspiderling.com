---
Title: Set up dev environment with Claude Code
Description: 
Author: sabyasachi
Date: 2026-01-10T21:17:24.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>Originally published at <a href="https://blog.sabyasachi.io/posts/prep-for-claude/" rel="noopener noreferrer">https://blog.sabyasachi.io/posts/prep-for-claude/</a></p>
</blockquote>

<p>As the competition for coding agents intensifies, Claude Code seems one of the popular choices. If you are relatively new to the space and thinking about how to get started, here is my two cents after using Claude Code for both personal projects and day job. </p>

<h3>
  
  
  Prepare the code so that Claude understands its context
</h3>

<p>To do that generate <code>CLAUDE.md</code> file in the repo. It is a file that Claude automatically pulls into the context.<br>
   <code>CLAUDE.md</code> can contain following information - </p>

<ul>
<li>An overview of what this repo is all about.</li>
<li>Important files. </li>
<li>Bash commands, including commands to test and build. </li>
<li>Formatting styles.
Claude Code comes with an in-built <code>/init</code> command which generates an initial <code>CLAUDE.md</code> file after exploring the codebase. Important is to review, add and update this first version to reflect the information about the codebase.</li>
</ul>

<p>Once we have a baseline <code>CLAUDE.md</code> file, it is important to keep it updated so that changes made to the codebase can be reflected. We can either automate this part by running some job after each main push or make it using human-in-the-loop, so that a reviewer can mark what needs to be added to the md file.</p>

<h3>
  
  
  GitHub Access
</h3>

<p>Considering majority using GitHub, Claude can access GitHub in two ways </p>

<ul>
<li>using <code>gh</code> CLI.</li>
<li>using GitHub MCP server.</li>
</ul>

<p>Often the whole truth about the codebase may not be within the same repo. For example it might be using some shared libs which are in different repo. Or we just simply want to search for usage of a certain pattern. Claude Code can use either <code>gh</code> CLI or MCP server.</p>

<h3>
  
  
  Setup MCP servers
</h3>

<p>MCP servers are what connects the model with the data it requires. I mentioned about GitHub MCP above, additionally I would recommend to use Context7 for documentation of open source libs, so that Claude can look into the documentation if required. The exact set of MCP is dependent on what you are actually building. In my context who is writing microservices using Java and AWS, my go-to MCP servers are </p>

<ul>
<li>GitHub</li>
<li>Context7</li>
<li>AWS knowledge MCP
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  Some MCP servers can bring too many tools or tool output, which can take up a significant chunk of the context window. GitHub is one of the MCP servers where tool surface area can get big.
  See the https://github.com/github/github-mcp-server/issues/1286
</code></pre>

</div>



<p>Additionally MCPs can bring security concerns. Fundamentally, once connected, Claude Code can call whatever tools the server exposes. Make sure to add only well-known, official servers, and use allowlists/denylists and MCP permissions where possible. Also note that project-scoped MCP servers live in <code>.mcp.json</code> and Claude Code prompts for approval before using them.</p>

<h3>
  
  
  Sub-Agents, Commands, Skills
</h3>

<p>At this point you have a repo with necessary <code>CLAUDE.md</code> file and you may have added a few MCP servers. You are already on good fundamentals. Claude can produce better results, from this section onwards we move into more powerful constructs of Claude which will take the experience into next level.</p>

<ul>
<li><p>Sub-Agents - Claude Code already runs a main agent loop. Sub-Agents are custom agents for specific workflow. For example, as developer once the feature development is done we can create a <code>code-review</code> sub agent which does a first phase of check before sending out the PR, or maybe a sub agent which fixes trivial bugs or update dependencies. Sub agents starts with a clean context so this can also be seen as extension of the main context. Sub agents can use tools coming from MCP servers. Claude Code can use a specific agent based on the agent description and context, otherwise we can mention to Claude Code to use the sub agent. </p></li>
<li><p>Commands - Once we start using Claude Code regularly for development, we will develop a few prompts which we repeat. For example it can be <code>please tell me about this repo</code> or <code>review my code</code> etc. As we repeat the same prompt a time saver is to create commands from them and use <code>/code-review</code> instead of writing a whole prompt. By doing this we also remain consistent.<br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>   Note- Claude Code comes with a few commands for regular coding tasks like code review and exploring PR. While those work, you may still need to create your own command that fits your code context.
</code></pre>

</div>



<ul>
<li>
<p>Skills - This is a real game changer. Skills are modular set of instructions, scripts and other resources which can extend the capability of the agent. Skills are model-invoked: Claude decides which Skills to use based on your request. Skills are auto-discoverable, and we can also give Skills to sub agents specifically. Let's take an example - maybe you always need to check for your org specific code formatting and imagine you have a feature-developer sub agent and a code-reviewer sub-agent. We can create a <code>format</code> skill and add the skill to both the sub agent. Now, skills are also discoverable by the main agent. For first time when I encountered skills my doubt was if main agent can discover skill should I start creating more skills or sub-agents ? The answer is still not clear to me but there are subtle nuances - </p>

<ul>
<li>Main agent only loads the skill name + description at startup and the full skill is loaded on demand hence saving some context.</li>
<li>Skills are chosen based on context and here we are dependent on the choice of the main agent.
In recent versions of Claude Code Skills have become more powerful. </li>
</ul>


</li>

<li><p>Skills can run in a forked context, and can also be run in the context of an agent (including custom ones).</p></li>

<li><p>Skills can be user-invocable too (there is a <code>user-invocable</code> flag that controls whether the skill appears in the slash command menu).</p></li>

</ul>

<h3>
  
  
  Sandbox
</h3>

<p>So, you may have a few agents, augmented with skills, maybe some command which summons the agent. That's a solid setup. But hey we haven't spoken about security. The next few points I would like stress about it so that we do not end up in yet another "Claude wiped up my prod db" scenario. When we run Claude without a sandbox two things happen - it is running on your hardware directly having access to your filesystem, second, Claude being itself conscious about permissions keep on asking for permissions for any command that it wants to access/do some operations. Sandbox provides an isolated environment for Claude Code to work on. Also note there are sandbox modes (auto-allow vs regular permissions mode), so you can tune how many prompts you get.</p>

<h3>
  
  
  Permissions
</h3>

<p>In many scenarios, we may not want Claude Code to access some files or, in an enterprise scenario we might want to restrict fetching from web. For these things set up Permissions. In practice this is Read/Edit/WebFetch rules, so you can restrict what Claude can read, what it can edit, and which domains it can fetch.</p>

<h3>
  
  
  Env
</h3>

<p>MCP configs support environment variable expansion in <code>.mcp.json</code> (useful for machine-specific paths and sensitive values like API keys). Keep secrets out of the repo and wire them via env vars instead.</p>

<p>Once you are here, you have a pretty solid baseline with Sandbox and Permissions to start with. You can now tweak this based on experience. </p>

<p>Agentic coding is gaining traction, models are becoming increasingly efficient to generate code, as developers we need to adopt these tools. Consider these agents as your peer programmer or assistant rather than a replacement. The onus now will be on reviewing what the code generates, tweaking the agentic coding environment to match your coding style etc. <br>
Hope this post will help to setup Claude Code and navigate the basic concepts.</p>

