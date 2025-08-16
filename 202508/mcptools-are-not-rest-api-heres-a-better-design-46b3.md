---
Title: MCP/Tools Are Not REST API: Here's a Better Design
Description: 
Author: Peter Mbanugo
Date: 2025-08-16T21:29:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>Large language models (LLM) without access to current data are almost useless. They need enough data and context to do the right thing, and that's where tool calling becomes leverage. The rise of MCP has unlocked more possibilities for AI agents to access current information and therefore provide useful results.</p>

<blockquote>
<p>Tools are functions or APIs that models or agents can call to perform tasks.</p>
</blockquote>

<p>A good practice for tool design is to provide a clear description of what each tool does and when to use it. You’ll find this info in some good resources about tools and function calling, but I’d like to address something else: <strong>Tools are not REST APIs</strong>.</p>

<p>What do I mean?</p>

<p>A lot of companies now offer MCP for their products and services. Most of the time, it's just a direct abstraction of their REST or GraphQL API. I don't mind cases with just a few tools to call; it becomes annoying when an MCP exposes 20 or more tools. Too many tool options make it hard for an agent to decide on the right tool to call. For example:</p>

<ul>
<li>📂 MCP Server: github-mcp-server-official

<ul>
<li>📜 get_notification_details Get detailed information for a specific GitHub notification...</li>
<li>📜 get_pull_request Get details of a specific pull request in a GitHub repository.</li>
<li>📜 get_pull_request_comments Get comments for a specific pull request.</li>
<li>📜 get_pull_request_diff Get the diff of a pull request.</li>
<li>📜 get_pull_request_files Get the files changed in a specific pull request.</li>
<li>📜 get_pull_request_reviews Get reviews for a specific pull request.</li>
<li>📜 get_pull_request_status Get the status of a specific pull request.</li>
<li>📜 get_secret_scanning_alert Get details of a specific secret scanning alert in a GitHub r...</li>
<li>📜 get_tag Get details about a specific git tag in a GitHub repository</li>
<li>📜 get_workflow_run Get details of a specific workflow run</li>
<li>📜 get_workflow_run_logs Download logs for a specific workflow run (EXPENSIVE: downl...</li>
<li>📜 get_workflow_run_usage Get usage metrics for a workflow run</li>
<li>📜 list_branches List branches in a GitHub repository</li>
<li>📜 list_code_scanning_alerts List code scanning alerts in a GitHub repository.</li>
</ul>


</li>

</ul>

<p>I recently added the official GitHub MCP server to VS Code; when I opened the <strong>Configure Tools</strong> dialog, the list was overwhelming. The list you saw is just a fraction of the tools available through that MCP. They've improved the UI since then, but there are still a few questions:</p>

<ul>
<li>Why can't some of the <code>get_pull_request_*</code> tools be combined into one? (Same for other similar tools in that MCP.)</li>
<li>Why aren't comments and status included in the <code>get_pull_request</code> tool instead of being separate tools?</li>
</ul>

<p>I think models will find it hard to select the right tools without explicit instructions to use a particular tool. If it's hard for me or the model to pick the right tool because of <em><strong>information overload</strong></em>, the tools become useless.</p>

<p>I believe a lot of thought went into GitHub's MCP design, and it may be unfair to single them out. Let's use a generic example that shows how to avoid designing your tools like a REST API.</p>

<h2>
  
  
  Tools for Stock Data
</h2>

<p>Assuming you want to build an MCP to provide core stock data to an agent, here’s an easy way you’d design the tools:</p>

<ul>
<li>
<code>get_intraday_stock_data</code>: For retrieving intraday stock data.</li>
<li>
<code>get_daily_stock_data</code>: For retrieving daily stock data.</li>
<li>
<code>get_adjusted_daily_stock_data</code>: For retrieving daily adjusted stock data.</li>
<li>
<code>get_weekly_stock_data</code>: For retrieving weekly stock data.</li>
<li>
<code>get_adjusted_weekly_stock_data</code>: For retrieving weekly adjusted stock data.</li>
<li>
<code>get_monthly_stock_data</code>: For retrieving monthly stock data.</li>
<li>
<code>get_adjusted_monthly_stock_data</code>: For retrieving monthly adjusted stock data.</li>
<li>
<code>get_stock_quote</code>: For retrieving a single stock quote from the Quote Endpoint.</li>
</ul>

<p>This design follows the typical REST endpoints you'd find with stock data providers.</p>

<p>That's too many tools for this use case. You can do better: what if you condensed it down to one or two tools?</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft8gh6sft6r4exjolpi3u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft8gh6sft6r4exjolpi3u.png" alt="A visual design on how to go from 8 tools, to 2. We settled on two tools, namely: get_stock_data, and get_stock_quote" width="800" height="420"></a></p>

<p>There you go — from eight tools to two. This works because the parameters for retrieving stock data are similar, and you likely only need a <em>switch statement</em> to choose which internal function or REST endpoint to call.</p>

<h2>
  
  
  Wrap Up
</h2>

<p>REST and GraphQL have specific constraints when it comes to designing APIs. REST, for example, emphasizes hypermedia and scalable web architectures. The constraints for function/tool calling in LLMs are different, which necessitate a different design approach.</p>

<p>You should think about how to simplify the number of tools and their arguments the next time you're designing or building tools. Having similarly named tools with almost identical arguments or descriptions can make it tricky for models to decide what to call. More importantly, providing too many tools can increase the risk of your model selecting an incorrect or suboptimal tool.</p>

<p><em>P.S. If you want to read more of my AI-related posts, subscribe to my newsletter which is at the bottom of <a href="https://pmbanugo.me/" rel="noopener noreferrer">this page</a>.</em></p>

