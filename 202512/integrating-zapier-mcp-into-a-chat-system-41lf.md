---
Title: Integrating Zapier MCP into a Chat System
Description: 
Author: Harsh Kumar
Date: 2025-12-21T21:44:04.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>While working on a chat-based automation prototype, I got the opportunity to explore integrating a Zapier MCP server into a chat-based system. The goal was to allow users to trigger real-world actions directly from chat without writing custom workflows for every use case.</p>

<p>This post explains how I approached the integration, the architecture I followed, how I designed the UI, and the problems I faced while working with Zapier MCP.</p>




<h2>
  
  
  Understanding the architecture
</h2>

<p>Before writing any code, I focused on understanding the core components involved:</p>

<ul>
<li>
<strong>Tools</strong>: Actions such as sending emails, updating spreadsheets, or creating tasks.</li>
<li>
<strong>MCP server</strong>: A server that exposes tools in a standardized way. In my case, this was Zapier MCP.</li>
<li>
<strong>MCP client</strong>: The LLM (Claude) that can discover and call available tools.</li>
<li>
<strong>MCP host</strong>: The application that connects user input, the LLM, and the MCP server.</li>
</ul>

<p>Once these roles were clear, the overall system design became easier to reason about. MCP works as a bridge between LLMs and external tools, avoiding the need for manual integrations.</p>




<h2>
  
  
  Overall flow
</h2>

<p>The flow I followed in the system was straightforward:</p>

<ol>
<li>The user provides input</li>
<li>The LLM discovers relevant tools</li>
<li>The LLM maps the user intent to tool instructions</li>
<li>Tools are executed via Zapier MCP</li>
<li>The LLM formats the final response</li>
</ol>

<p>This can be summarized as:</p>

<p>User intent → tool discovery → tool execution → formatted response</p>




<h2>
  
  
  Tool discovery and execution
</h2>

<p>In practice, the process looked like this:</p>

<ul>
<li>Available tools and user input are sent to Claude</li>
<li>Claude returns a list of relevant tools</li>
<li>The discovered tools are sent back to Claude to map instructions</li>
<li>Zapier MCP executes the instructions</li>
<li>Claude formats the tool responses into a final output</li>
</ul>

<p>This approach required multiple LLM calls, which later became one of the main drawbacks.</p>




<h2>
  
  
  Setting up Zapier MCP
</h2>

<p>Setting up Zapier MCP was relatively simple.</p>

<p>I created a Zapier account, selected the tools I needed, enabled the Zapier MCP server, and copied the MCP server URL and API key. This MCP endpoint was then connected to Claude using the Anthropic Messages API.</p>

<p>Once connected, I could query the system to discover available tools, and Zapier MCP handled the execution layer reliably.</p>




<h2>
  
  
  UI design
</h2>

<p>The UI was intentionally minimal.</p>

<p>It consisted of a chat input, a panel showing active tool execution states, and a final section displaying completed results. The goal was to make tool usage transparent so users could see what was running and when execution was complete.</p>

<p>This helped reduce the feeling of a black box system.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3tvh2n592js8p72xzn9r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3tvh2n592js8p72xzn9r.png" alt=" " width="800" height="356"></a></p>




<h2>
  
  
  What worked well
</h2>

<p>Zapier MCP was easy to integrate and worked well for quick experimentation. It removed the need to manually connect multiple APIs and allowed the LLM to focus on intent and orchestration.</p>

<p>For fast iteration and prototyping, this setup was effective. I briefly tried n8n as well, but it felt heavier for quick experiments, so Zapier was a better fit for my workflow.</p>




<h2>
  
  
  Limitations and frustrations
</h2>

<p>There were a few clear limitations.</p>

<h3>
  
  
  Multiple LLM calls
</h3>

<p>To produce a single clean response, the system required separate LLM calls for tool discovery, instruction mapping, and response formatting. Reducing these steps without adding extra glue code would improve the developer experience.</p>

<h3>
  
  
  Closed source nature
</h3>

<p>Zapier is closed source, which limits visibility into internal execution. Debugging and customization beyond the exposed interface are restricted.</p>

<h3>
  
  
  Limited control
</h3>

<p>While the abstraction is useful for speed, it also means giving up fine-grained control over execution logic, which can be an issue for complex workflows.</p>




<h2>
  
  
  Conclusion
</h2>

<p>Integrating Zapier MCP into a chat system was a solid experience overall.</p>

<p>It works well for rapid development, experimentation, and proof-of-concept builds. However, for advanced workflows that require deep control and visibility, additional layers or alternative approaches may be needed.</p>

<p>This experiment helped me better understand how LLM-driven tool orchestration can be built using MCP-based systems.</p>

