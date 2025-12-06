---
Title: How can I connect a local MCP server (Python) to ChatGPT? - “Unsafe URL” issue
Description: 
Author: Makki
Date: 2025-12-06T21:45:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hello,<br>
I’m running a local MCP server using the official Python SDK (mcp, Anthropic).<br>
The server works and exposes the MCP endpoint at:</p>

<p><a href="http://localhost:8000/mcp" rel="noopener noreferrer">http://localhost:8000/mcp</a></p>

<p>When I try to add this endpoint inside ChatGPT under:</p>

<p>Apps &amp; Connectors → Add Connector → MCP Server URL</p>

<p>ChatGPT rejects it with the message:</p>

<p>Unsafe URL</p>

<p>My ChatGPT account does not have the “Developer Tools → MCP Servers” menu that some documentation mentions. I only have the “Apps &amp; Connectors (Beta)” section.</p>

<p>My questions:<br>
Is it currently possible to connect a local MCP server (HTTP / localhost) to ChatGPT?</p>

<p>Does “Apps &amp; Connectors” support MCP, or only remote HTTPS endpoints?</p>

<p>If ChatGPT cannot connect to localhost, what is the correct way to test MCP locally — should I implement my own MCP client or host the server publicly over HTTPS?</p>

<p>My goal is to have an AI (ChatGPT) call MCP tools such as a Selenium test via my local MCP server. Any clarification on the current limitations or correct workflow would be appreciated.</p>

