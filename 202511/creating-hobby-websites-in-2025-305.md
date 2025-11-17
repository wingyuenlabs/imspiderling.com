---
Title: Creating hobby websites in 2025
Description: 
Author: Mark Kop
Date: 2025-11-17T21:24:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey, guys!<br>
Let me show you how I build new hobby websites with AI tools nowadays. </p>

<h2>
  
  
  🧠 Brainstorm with AI agent
</h2>

<p>Using a voice transcription AI tool, like Cursor's native recorder or "Wispr Flow", I brain dump all my ideas and requirements into text and save it in a PRD.md file or in a Google Doc. I also attach a bunch of other copy-pasted text that might be related to that project.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2khnzwxw4q4svdqj6usw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2khnzwxw4q4svdqj6usw.png" alt=" " width="800" height="352"></a></p>

<h2>
  
  
  🧱 Init a codebase
</h2>

<p>I personally go with <code>npx create-next-app@latest my-next-app --tailwind --typescript --eslint --app --turbopack<br>
--use-pnpm</code> and <code>npx shadcn@latest init</code>.</p>

<p>Then, I use an AI Agent tool, such as Claude Code, to create the project. I paste the document create before into a prompt and add some extra details, for example:</p>

<blockquote>
<p>Project in nextjs, tailwind, shadcn components (don't create them, import them using shadcn commands), @neondatabase/serverless Neon DB, <a class="mentioned-user" href="https://dev.to/stackframe">@stackframe</a>/stack Stack Auth</p>
</blockquote>

<p>Full Prompt: <a href="https://gist.github.com/Markkop/765612eecb7f6a7e1c2564eda03b228b" rel="noopener noreferrer">https://gist.github.com/Markkop/765612eecb7f6a7e1c2564eda03b228b</a></p>

<h1>
  
  
  ⚒️ Fixes and features
</h1>

<p>Now, using Claude Code and Cursor, I finish the integrations I want and the fixes needed. I also try to leverage AI at max via Playwright MCP/Chrome MCP/Cursor (browser testing) and Neon MCP (db updates).</p>

<p>This is where I most leverage my dev knowledge: I guide the AI to build the features I plan, change libraries, follow patterns and so on. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1igaxp5vehfhosqv3qcx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1igaxp5vehfhosqv3qcx.png" alt=" " width="800" height="491"></a></p>

<h1>
  
  
  🌎 Publish and Release
</h1>

<p>Finally, when I have something okayish to release, I setup Vercel and domains.</p>




<p><em>When I started writing this post, I realized there's a lot of more profound knowledge applied to the process that is too much for write in a single post, so take this more as an overview of the process</em></p>

