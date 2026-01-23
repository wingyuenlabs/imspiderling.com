---
Title: The best way to do agentic development in 2026
Description: 
Author: Chandler
Date: 2026-01-23T21:25:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>TLDR; I tried Claude Code, switched to OpenCode + Oh-My-Code, then finally switched to Conductor + Claude Code + Lots of Plugins and Skills.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa5ke7ih1q23l9txwmhmo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa5ke7ih1q23l9txwmhmo.png" alt="Opinionated meme" width="500" height="701"></a></p>

<h2>
  
  
  Claude Code is good
</h2>

<p><a href="https://code.claude.com/docs/en/overview" rel="noopener noreferrer">Claude Code</a> by itself is an amazing tool, hell my coworkers and colleagues use it constantly. I was always a little more hesitant, preferring Cursor in most cases because I wanted the freedom to edit some code myself while the AI agent was doing its thing, while also keeping close watch on the agent to make sure that the code it was producing worked well and was up to my standards (it usually was, if it passes my strict CI its probably fine).</p>

<p>The problem I always had with Claude Code wasn't the lack of editing: for a while (and still quite often now) I would run Claude Code in my VSCode/Cursor terminal while editing other files. It was price to performance.</p>

<h3>
  
  
  It was <em>just okay</em>.
</h3>

<p>While it always seemed to <em>work</em>, it never always <em>worked well</em>, nor did it ever seem to ever seem to work ever on the first try.</p>

<p>Some of this was remedied when moving from Sonnet 3.x to Sonnet 4 series, and even more when moving to the 4.5 series of models, but especially Opus 4.5. However it was never perfect.</p>

<h2>
  
  
  OpenCode + Oh-My-OpenCode
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqgliwyh0cl4ir93n5ndx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqgliwyh0cl4ir93n5ndx.png" alt="Me a few weeks ago" width="500" height="500"></a></p>

<p>A friend of mine introduced me to <a href="https://github.com/code-yeongyu/oh-my-opencode" rel="noopener noreferrer">Oh-My-OpenCode</a> (and I've known about OpenCode but always preferred Claude Code) at the beginning of the year, and <strong>IT. IS. WONDERFUL!</strong></p>

<p>Feels like an actual developer that lives on your machine. Its basically a plugin that significantly improves the performance of OpenCode with subagents, and an "ultrawork" mode that allows more efficient background tasks and subtasks to be delegated to other agents.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftd2k8ucji5konomr1ftn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftd2k8ucji5konomr1ftn.png" alt="Their example screenshot of it working" width="800" height="509"></a></p>

<p>Plus, unlike Claude Code, OpenCode supports many AI providers (the main two they talk about are Codex and Claude Code.</p>

<p>Now that all seems amazing, what's the catch? <strong>Cost.</strong></p>

<h3>
  
  
  Anthropic is kinda mad.
</h3>

<p>When I first started using Oh-My-OpenCode, one of its selling points is that it allowed you to use your <a href="https://claude.com/pricing" rel="noopener noreferrer">Claude Code Pro or Max</a> subscriptions to use OpenCode, which made it <em>significantly</em> more cost effective. Since then, <a href="https://github.com/code-yeongyu/oh-my-opencode?tab=readme-ov-file#claude-oauth-access-notice" rel="noopener noreferrer">Anthropic has blocked this</a> as they claim this was outside their ToS.</p>

<p>I knew this may become an issue, and I had tons of Anthropic grant credits that expired and were burning a hole in my pocket, so I opted to use API keys from the start. </p>

<p>So what's my problem then? While OpenCode + Oh-My-OpenCode has impressive performance, there's a lot of dead time where nothing is happening. I'm just sitting there watching it code. Now I could do what I normally do and complete other non-coding tasks that are related to my work, or just procrastinate by doing non-work tasks in the background, but I'm a productivity nerd. I want to get as much done as possible in the least amount of time possible.</p>

<h2>
  
  
  Conductor - trains or music?
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feut8avqcoc1v58efjj5p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feut8avqcoc1v58efjj5p.png" alt="Example from their website" width="800" height="550"></a></p>

<p><a href="https://www.conductor.build/" rel="noopener noreferrer">Conductor</a> is a unique tool. Its currently a macOS-only (Windows and Linux coming soon) developer tool that uses Claude Code (or OpenAI Codex) to orchestrate many git worktrees of a repo to accomplish many tasks at the same time. It has <em>tight</em> integration with Linear and GitHub (Linear is especially fitting since they're one of the larger companies using it), allowing you to inject Linear (and GitHub) issues as context, provide feedback as it developers the app, run multiple Claude Code instances in a single or across multiple worktrees, handoff plans to other agents, review code, and open GitHub Pull Requests right from within the app. I don't leave the app unless I need to request a review on my PR (I should ask them for that feature...).</p>

<p>It also handles GitHub PRs amazing. One of your CI checks fail? No problem, click the "fix" button in the top right to automatically fix it. Merge conflicts? No problem, one click fix those too!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9qlktjoanv1tdxjlxekn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9qlktjoanv1tdxjlxekn.png" alt="Fix CI errors with a single click!" width="800" height="596"></a></p>

<p>Now this alone wouldn't get me to move over from Oh-My-OpenCode since the code from Claude Code was worse, however there's one more piece to this agentic puzzle that made the switch worth it.</p>

<h2>
  
  
  Superpowers (and friends)
</h2>

<p><a href="https://github.com/obra/superpowers" rel="noopener noreferrer">Superpowers</a> is a plugin for Claude Code that just seems to make it smarter, especially when running it in planning mode (which you absolutely should be doing). It asks you more intelligent questions as it goes, fleshing out implementation details to make its actual implementation better. It'll spawn subagents (just like Oh-My-OpenCode) for gathering context from all your files in your repo or other MCP servers. It can even use your testing suites to make sure its new code is passing linting, formatting, and other CI/CD. It accomplishes this by providing a number of skills (basically just fancy prompts injected into the system prompt with hyper specific instructions for tasks) which allow it to perform within spitting distance of Oh-My-OpenCode, while still being able to use your existing Claude Code Max subscription.</p>

<p>The planning part of Superpowers is definitely the reason it performs so much better </p>

<h3>
  
  
  Other Plugins
</h3>

<p>I don't just use the superpowers plugin, I also have a few others that I consider essential to making Claude Code perform at its best. </p>

<p>An example of this is the Context7 plugin. <a href="https://context7.com/" rel="noopener noreferrer">Context7</a> is a documentation provider that is specifically designed to allow AI agents find and read documentation. This allows Claude Code to search for the latest documentation and use APIs properly the first time, minimizing required iteration.</p>

<p>Another plugin I use that isn't listed in the Anthropic plugins store is <a href="https://docs.tavily.com/documentation/claude-code#npx-recommended" rel="noopener noreferrer">Tavily</a>. I use this to give Claude Code improved web search capabilities and to integrate research abilities into the repository, which has been especially useful when trying to find information about documentation that isn't yet in Context7, or to do a research task before accomplishing a complex feature like trying to find out how this is usually implemented in other projects.</p>

<h2>
  
  
  How do I get this set up?
</h2>

<h3>
  
  
  Claude Code + Plugins
</h3>

<p>First step is to <a href="https://code.claude.com/docs/en/quickstart" rel="noopener noreferrer">install Claude Code</a>. If you're on macOS or Linux, this can be done the official script.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-fsSL</span> https://claude.ai/install.sh | bash
</code></pre>

</div>



<p>Once that's done, you should log into Claude by running <code>claude</code> in your terminal then running <code>/login</code>.</p>

<p>You can leave Claude for now. Now we need to install all the good plugins. I prefer use level install for most things so that its used across all my projects, which can be done from the command line. Let's start with the easy ones.</p>

<p>You don't need to install all of these, but I am going to list all plugins I have installed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>claude plugin marketplace add obra/superpowers-marketplace
claude plugin <span class="nb">install </span>superpowers@superpowers-marketplace
claude plugin <span class="nb">install </span>context7@claude-plugins-official
claude plugin <span class="nb">install </span>playwright@claude-plugins-official
claude plugin <span class="nb">install </span>frontend-design@claude-plugins-official
claude plugin <span class="nb">install </span>feature-dev@claude-plugins-official
claude plugin <span class="nb">install </span>commit-commands@claude-plugins-official
</code></pre>

</div>



<h3>
  
  
  Tavily MCP Server
</h3>

<p>Unlike many MCP servers, you don't need to run this one locally. You start by making an account on <a href="https://www.tavily.com/" rel="noopener noreferrer">Tavily</a>, then by following their official skill instructions. Here's a TLDR if you don't want to use their docs.</p>

<p>Open your Claude settings, which is at <code>~/.claude/settings.json</code>, using your preferred text editor.</p>

<p>Once that's done, add a new section called env. You'll add your Tavily API key, (which you need an account for) Should look something like this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"env"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"TAVILY_API_KEY"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tvly-YOUR_API_KEY"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Replace <code>tvly-YOUR_API_KEY</code> with your actual API key. Once that's done, go to your terminal and run the following command, following the instructions to install the skill.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx skills add https://github.com/tavily-ai/tavily-plugins
</code></pre>

</div>



<p>Once that's done, you can call those commands you added from anywhere, or you can tell Claude to use Tavily to perform searches to research anything you want!</p>

<h3>
  
  
  Conductor
</h3>

<p>Now that we have Claude Code set up for maximal productivity, we need the program that ties it all together: <a href="https://www.conductor.build/" rel="noopener noreferrer">Conductor</a>.</p>

<p>Install it, create your account, then (ideally) link your GitHub and Linear accounts to your Conductor accounts. This will let you quickly add context via injecting issues.</p>

<p>Once that's all done you're ready to rip with Conductor! Utilize the superpower skills, research topics, link Linear and GitHub issues, <strong>always</strong> use Planning mode, and dump as much context as you can into it to make your issues work as best as possible! I highly recommend use of the <code>/research</code> command or telling it about its new skills when prompting to make it especially intelligent when accomplishing tasks!</p>

<p>The real power when using Conductor though is that its not just a pretty UI: its a force multiplier. Now you can open many projects, many repos, and work using git worktrees (parallel workspaces) to accomplish many issues quickly. I'll run parallel agents on many issues across multiple project by dumping as much context as possible. Since I've started at <a href="https://www.saphira.ai/" rel="noopener noreferrer">a new company</a> I've needed to gather lots of context very quickly from existing documents, and the fastest way to accomplish this (due to their extensive use of Google Drive) was to use <a href="https://notebooklm.google/" rel="noopener noreferrer">NotebookLM</a> to gather context on projects and information on clients and what they (both my colleagues and clients) would want. Since we also switched to Linear for Issue and Project management, I can dump additional context from there into Conductor, and build in parallel extremely fast.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbbvdgd0xuyh5dhitk9yp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbbvdgd0xuyh5dhitk9yp.png" alt="Many issues" width="441" height="390"></a></p>

<p>The only issue I've ran into is that when you run commands with Conductor, (<code>/commit</code>, any slash commands) it tends to ignore any attached issues. The solution is to attach them anyways and tell it to use the Linear/GitHub Issue to complete your task, and when it can't find it it'll ask for you to attach it again. I then attach it again to the new message, and it finds it and uses it.</p>

<p>Any questions? Leave a comment below!</p>

