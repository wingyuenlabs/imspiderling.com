---
Title: How I added experimental MCP Apps support to Apollo MCP Server with Goose
Description: 
Author: Amanda
Date: 2026-01-10T21:27:22.000Z
Robots: noindex,nofollow
Template: index
---
<p>The challenge: contribute to my companies codebase in a language I do not write using an agentic dev workflow.</p>

<p>As a developer advocate, I'm often creating demo and educational code versus contributing to our product codebases. Over the holiday break I wanted to challenge myself to fully lean into agentic coding by adding experimental support for MCP Apps draft spec to the Apollo MCP Server (<a href="https://github.com/amandamartin-dev/apollo-mcp-server-testing" rel="noopener noreferrer">repo here</a>).</p>

<p>It was time to test everything I teach. in a real world scenario in a language I do not write...Rust. </p>

<p>The Apollo MCP Server is open source and is used as a way to expose GraphQL operations as MCP tools without writing any additional code. We recently added support for the OpenAI Apps SDK so evolving into MCP UI apps was a great next step to challenge myself on.</p>

<h2>
  
  
  Constraints
</h2>

<p>The agent had to add experimental support for MCP Apps without breaking existing functionality or interfering with the OpenAI Apps implementation. To validate my changes, I also needed a separate repository with a prototype MCP UI App for my Luma Community Analytics tool.</p>

<h3>
  
  
  The Tools
</h3>

<p>These are the tools and techniques I leveraged to make this a successful build.</p>

<ol>
<li>goose CLI and Desktop</li>
<li>Markdown files to provide instructions and important information to the agent including MCP Spec details and general project goals and rules</li>
<li>Apollo MCP Server</li>
<li>MCP Jam </li>
</ol>

<h2>
  
  
  The Build
</h2>

<p>I needed to work with the agent to get to a point where we had a flow that  followed a research/plan → code in chunks → test → report deviations → repeat process. This was the core idea I had to make this a successful test.</p>

<p>My research started with a simple prompt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"research all files in the apollo-mcp-server local repo to understand the OpenAI apps implementation and the details of the draft MCP UI Apps spec. Then create a plan to add expirimental support for MCP apps that preserves the open ai SDK version"
</code></pre>

</div>



<p>This wasn't a magical plan that was perfect the first time. Iteration on this plan was the most "human in the loop" work that I completed for this project. In my experience spending time on a planning phase is very important when working in an existing codebase especially in the case of adding a new/experimental feature that should not impact the rest of the project.</p>

<p>One of the features of my plan that increased the success of my agent was to have each chunk of the plan followed by full testing of the code. This way, errors were caught along the way and fixed rather than having a mess at the end. I also requested that the final report output have any details where the agent needed to deviate from the plan based on errors or other information.</p>

<p>Once it was complete, I built the rust binary and added it to an MCP Apps project for testing.</p>

<p>Did it work the first time? No.</p>

<p>However, I was able to quickly debug with goose to find the few loose ends that did not get caught such as a query parameter used in our OpenAI SDK version that was not actually MCP App compliant but didn't flag on a test.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4gykguepsyl3dqknz4he.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4gykguepsyl3dqknz4he.png" alt="the offending query param code" width="800" height="376"></a></p>

<p>With the app working, I was able to do a preliminary test in MCP Jam which is an alternative to the MCP Inspector. It's a great experience and the regular inspector does not currently support apps.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F36ezyxdj9ojz5933mcqu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F36ezyxdj9ojz5933mcqu.png" alt="initial load test in mcp jam" width="800" height="534"></a></p>

<p>At this point... I had to wait.  Remember MCP Apps is in draft and the agent ecosystem hadn't begun to support it yet. The Goose team at this point was almost there but it was the holiday break so I actually shut my laptop. (go touch grass?)</p>

<h2>
  
  
  The final test!
</h2>

<p>Goose team released draft spec support on ...  I immediately rushed to test in the Goose desktop agent but ran into a few issues.  </p>

<p>Back to goose again to debug!</p>

<p>Goose helped me discover 2 small bugs in the early release of MCP Apps support in goose and I was able to report quickly to the team so they could patch.</p>

<p>You read that right. I made goose debug goose!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feu98s5iz06nq5rvgxwvt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feu98s5iz06nq5rvgxwvt.png" alt="spiderman pointing as himself meme" width="567" height="320"></a></p>

<p>Ultimately I ended up with the early prototype of my Luma events dashboard which will be a great tool for folks internally here at Apollo to understand community metrics for events I host.  </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft6nkkjgowp2rb8pjfuxd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft6nkkjgowp2rb8pjfuxd.png" alt="Screenshot of goose desktop with a running mcp app" width="800" height="551"></a></p>

<h2>
  
  
  Learnings
</h2>

<ol>
<li>You don't have to be an expert in a language to contribute a meaningful feature to a codebase</li>
<li>Plan, Iterate, Plan</li>
<li>Agents working in small chunks and always testing means they can catch errors early, fix and move on while you grab a coffee</li>
<li>Is perfection the goal? In my experience, not yet but I was able to move so much faster as a developer. The server updates and new app were completed in half a day with me starting from zero knowledge of the draft spec or codebase.</li>
</ol>

<p><em>Note: After I completed this I saw a post by Angie Jones for a Research -&gt; Plan -&gt; Implement flow with recipes that she released over the holiday break too.  If I were starting this flow again today, this is where I would start.  <a href="https://block.github.io/goose/docs/tutorials/rpi/" rel="noopener noreferrer">Check the docs here</a>.</em></p>

<p>Can you do this? Yes! I'd love to hear your experience testing this workflow or other agentic coding flows you are testing.</p>

