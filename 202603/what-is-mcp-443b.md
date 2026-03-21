---
Title: What is MCP?
Description: 
Author: Evan Lausier
Date: 2026-03-21T21:40:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you've been anywhere near developer Twitter or LinkedIn lately, you've probably seen this diagram floating around. Credit to <a href="https://www.linkedin.com/posts/alexxubyte_systemdesign-coding-interviewtips-share-7441153714280800256-sYCp?utm_source=social_share_send&amp;utm_medium=member_desktop_web&amp;rcm=ACoAABTQGW4BBR8GJa2Y1_38Rb1pMgrCd8S37no" rel="noopener noreferrer">Alex Xu at ByteByteGo</a> for putting it together so cleanly. It stopped me mid-scroll because it finally gave a clear visual to something I had been reading about in pieces.</p>

<p>MCP, or Model Context Protocol, is an open standard from Anthropic that gives AI models a structured way to connect to external tools, data sources, and services. Think of it as a universal adapter layer between the model and the rest of your stack. Instead of writing one-off integrations every time you want Claude to touch a database or call an API, MCP gives you a consistent protocol to build against.</p>

<p>The diagram breaks it down into two layers worth understanding. The top half shows the host architecture, where an MCP host like Claude Desktop or an IDE runs multiple MCP clients, each speaking to a dedicated MCP server that fronts a specific resource. The bottom half gets into the core building blocks, which are the five primitives the whole thing runs on.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffs8m6p2ddidvj6unq9se.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffs8m6p2ddidvj6unq9se.gif" alt=" " width="800" height="939"></a></p>

