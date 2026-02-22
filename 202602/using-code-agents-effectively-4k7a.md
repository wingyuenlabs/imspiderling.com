---
Title: Using Code Agents Effectively
Description: 
Author: Anson Biggs
Date: 2026-02-22T21:36:15.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1745356688130-7de19fefdf0a%3Fcrop%3Dentropy%26cs%3Dtinysrgb%26fit%3Dmax%26fm%3Djpg%26ixid%3DM3wxMTc3M3wwfDF8c2VhcmNofDcwfHxjb2RlcGFpcnxlbnwwfHx8fDE3NTk3MTcyMTR8MA%26ixlib%3Drb-4.1.0%26q%3D80%26w%3D2000" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1745356688130-7de19fefdf0a%3Fcrop%3Dentropy%26cs%3Dtinysrgb%26fit%3Dmax%26fm%3Djpg%26ixid%3DM3wxMTc3M3wwfDF8c2VhcmNofDcwfHxjb2RlcGFpcnxlbnwwfHx8fDE3NTk3MTcyMTR8MA%26ixlib%3Drb-4.1.0%26q%3D80%26w%3D2000" alt="Using Code Agents Effectively" width="800" height="450"></a></p>

<p>Every time LLMs get an "upgrade" they feel like magic, the first impressions are always so strong. That's how I felt with code agents, all of a sudden models were hyper context aware to my exact system, and could iterate on problems until they made a working solution. The craziest part is I don't even have to understand the solution because the Agent already implemented it. But as it always goes, playing with this new capability you quickly find it has hard limits. Beyond creating a toy project they begin to cut major corners, or making any edits to a large repo they just completely fail.</p>

<p>The latest models, GPT5-Codex and Sonnet 4.5, are well into the territory of being as capable as a new engineer. However, they need to be directed with extreme nuance and can't just be let loose on problems like an eager new grad. The models are smart enough to get just about anything done, but it not without a lot of teamwork. Below I'll detail some of the strategies I'm using to get good results with the current batch of cutting edge LLMs and tools.</p>

<h2>
  
  
  Managing Context
</h2>

<p>Context is worth more than gold at this point. Even with large token windows every token is poison that is ruining your results. This is from 3 major angles:</p>

<ul>
<li>Price

<ul>
<li>With current Anthropic pricing a single prompt can cost over $30</li>
</ul>


</li>

<li>Tasking Overload

<ul>
<li>Your model is thinking about its system prompt, every <code># TODO</code> it finds, all of your instructions etc. All of these compound to confuse the model.</li>
</ul>


</li>

<li>Retrieval

<ul>
<li>Every model is doing something a little different to keep all those tokens in its brain, in reality none of them are really holding hundreds of thousands of tokens in their head. Retrieval is probably the most important area of advancement for LLMs right now and is far from a solved problem.</li>
</ul>


</li>

</ul>

<h3>
  
  
  Sub-Agents - This is currently my obsession
</h3>

<p>Having your Agent call other Agents is a crazy hack. By having a fresh agent manage a task it: Gets higher quality answers, keeps your main agents context small, and saves money.</p>

<p>There are two tasks that happen a lot when doing code pairing at work with an Agent, running build systems and reading our proprietary docs via MCP. Both of these are tasks where we need a small amount of info, but the tools dump a massive amount of tokens. The average docs page is about 10k tokens, and even a small <code>make build</code> is well into the thousands or tokens. Models are really good at doing specific <code>grep</code>s or using <code>tail</code> to get exactly what they need from a document, but it doesn't always work out like that. So having a fresh Agent get told "Run make build, I edited xxx.hpp let me know if there are any errors" gets your agent the concise binary feedback of "Your change worked" or gives it details of exactly what got mad.</p>

<h3>
  
  
  Compacting Frequently
</h3>

<p>This is a feature that currently varies wildly between tools and models. Claude Code v2.0 is pretty damn good at it. I used to just manage this myself and start new models frequently but it is currently good enough that I keep the same chat going. This is something where it is more vibes than strict rules, but I tend to compact after a new feature is implemented and tests are passing. Pretty much the same milestones as when I would make a code commit.</p>

<h3>
  
  
  Planning with Precision
</h3>

<p>In a large repo you have to be extremely explicit with tasking, but you can't provide too much info. Working on a proprietary tool is really hard because the model understands literally nothing about the product, but if you just dump all of your docs and a few folders into context then you're essentially lobotomizing your model.</p>

<p>You need to be exact with telling the Agent exactly where to look and what path to take for implementation. If you don't know those answers then you probably need to use a few different Agents to get the relevant info, and help you build a prompt for your Agent that is going to do the executing.</p>

<h3>
  
  
  Clean Workspace
</h3>

<p>Poisoning your model with bad prompts is a very big issue. I had a system level prompt that said <code>Use uv for all Python commands</code> and my model was thinking to itself about this rule after every single step. We weren't even in a Python repo anymore. Try to avoid any system level prompts and work on improving your prompts or AGENTS.md in individual repos.</p>

<p>I was working in an old repo with a deprecated tool that had an upgrade warning on every build so my model was certain that was the issue and that everything would work with an upgrade. So, it started modifying my system and installing the latest update. In this case I had to make a wrapper around the function that hid the issue.</p>

<h2>
  
  
  Meta Prompting
</h2>

<p>This is kind of just sub-Agents but on a higher level. I've had a lot of success having one Agent working with me on the big picture, and coming up with prompts for a worker agent together. This is especially useful if a task is way too large for a single model to tackle at once.</p>

<p>In practice this usually has me talking to two Agents in separate terminals, one Agent is a manager of sorts and knows at a high level what we are doing. The other window gets reset often as we hit context limits or start down a completely wrong path. I'll collaborate with the manager Agent on what prompt we should provide, and when the worker model gets stuck we will take a higher level approach of figuring out what context it needs to go down the right path. This is a highly iterative process.</p>

<p>I think the current batch of models are good enough that you could do this fully programmatically and have a manager Agent that you talk to that fully breaks down and distributes tasks but I haven't gotten to test that with the latest models. Sonnet 4 wasn't good enough but Opus 4.1 was close.</p>

<h2>
  
  
  Self-Check Loop
</h2>

<p>Your Agent needs to be able to check on its own work. This is a key aspect that separates Agents from chatbots. You need to have a clear path laid out for it to build and test your code or else it is just going to build junk. Tests are also amazing documentation. If you or a different Agent can write tests before implementation begins then not only do you have a tight loop but code is far more concise than English.</p>

<h2>
  
  
  The Best Model and Tool Changes Daily
</h2>

<p>The combo of LLM and <a href="https://brain.ansonbiggs.com/Agentic-Tools" rel="noopener noreferrer">Agentic Tool</a> makes a massive difference in the quality of answers. Anthropic recently put out a <a href="https://www.anthropic.com/engineering/a-postmortem-of-three-recent-issues" rel="noopener noreferrer">post-mortem</a> where they admitted that bugs in their infra made their models dumber. So, if one day something works and the next it doesn't, it is probably not your fault. If someone comes out with a new update you should probably try it for yourself to see which combination of mode/tool/task really excels.</p>

<h2>
  
  
  Conclusion
</h2>

<p>Keep your context close and your sub-Agents closer. The best you can do is keep your context as small as possible, and don't be afraid to fully restart if the current path isn't great. I keep thinking that some of this stuff won't matter with the next era of models, but every time they ship I find myself tracking more things. I've certainly been displaced from some lines of work by Agents but we are a long way away from them building cool shit at scale.</p>

