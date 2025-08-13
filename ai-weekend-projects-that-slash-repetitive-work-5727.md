---
Title: AI Weekend Projects That Slash Repetitive Work
Description: Lets build some project that is not just a entry in your resume like, YouTube clone or Reddit clone.
If you’ve been following my articles, you know I spend a few weekends each month on what I call “Pe...
Author: hiruthicSha
Date: 2025-08-12T06:04:24.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Lets build some project that is not just a entry in your resume like, YouTube clone or Reddit clone.</strong></p>

<p>If you’ve been following my articles, you know I spend a few weekends each month on what I call “<a href="https://hiruthicsha.medium.com/code-coffee-and-a-weekend-my-personal-hackathon-ritual-c6d8d48894da" rel="noopener noreferrer">Personal Hackathons</a>”.<br>
These are my sprint sessions to build something that either:</p>

<ul>
<li>Becomes a product I can monetize, or</li>
<li>Eliminates friction from my daily workflows.</li>
</ul>

<p>Here are some projects I’ve already built for myself, and a few more I’m planning for the coming weeks.</p>

<h2>
  
  
  Environment Status updates in Messaging Channels
</h2>

<p><strong>Build a tool that periodically monitors a service and takes action when certain conditions are met, for example:</strong></p>

<ul>
<li>If the response code isn’t 200, notify the DevOps team and possibly a potential fix aswell using an LLM.</li>
<li>If latency exceeds 250ms, alert the development team and automatically create a ticket in your project management tool.
-If the issue is simple, trigger a background coding agent to work on the task and raise a PR.</li>
</ul>

<p>This isn’t just for your organization, you can use it for your own hosted services as well.</p>

<h3>
  
  
  What to learn
</h3>

<ul>
<li>A bit of scripting (Bash/Python/Node.js)</li>
<li>REST APIs for your deployment environment.</li>
<li>Power automate/n8n if your organization have the subscription.</li>
<li>Prompt engineering.</li>
</ul>

<h3>
  
  
  How to approach
</h3>

<ul>
<li>Set up a bot or webhook for the messaging medium that your organization uses.</li>
<li>Create a small service that fetches environment status from your infra (K8s cluster, VM health checks, API uptime).</li>
<li>Post updates on-demand or at intervals to a dedicated Teams channel.</li>
<li>Use the issue and code repository context, ask a coding agent to work on the task.</li>
</ul>

<h3>
  
  
  Potential
</h3>

<ul>
<li>Saves “Hey, is staging up?” Slack/Teams chatter.</li>
<li>Can be extended to trigger alerts on failures.</li>
<li>Please make sure your team’s in the loop and that you’re within organizational policy, before the IT team catches you and makes that face.</li>
</ul>

<h3>
  
  
  CI/CD Pipeline Highlighter/Alert
</h3>

<p><strong>This is similar to the previous tool. Build a tool that can detect pipeline completions or failure and trigger an alert to the respective person.</strong></p>

<p>I have built this exact thing but only for highlighting, but you can extend it to alert. Below I have mentioned each of the learning path with “[Alerting]” or “[Highlighting]”</p>

<h3>
  
  
  What to learn
</h3>

<ul>
<li>[Alerting] Your DevOps provider API (Azure, GitHub or others)</li>
<li>[Highlighting] Browser extension development.</li>
<li>[Highlighting] Basic HTML/CSS for visual cues.</li>
</ul>

<h3>
  
  
  How to approach
</h3>

<ul>
<li>[Alerting] Fetch pipeline run data via Azure DevOps API.</li>
<li>[Highlighting] Highlight failed tasks, bottlenecks, or flaky tests visually.</li>
<li>[Highlighting] Build as a browser extension.</li>
<li>[Highlighting] String parsing through HTML to find the relevent emails.</li>
</ul>

<h3>
  
  
  Potential
</h3>

<ul>
<li>Immediate clarity on where a build went wrong.</li>
<li>A massive time-saver during release crunch.</li>
<li>Shows skill in automation + front-end + Browser extension development.</li>
</ul>

<h2>
  
  
  WebUI for Ollama
</h2>

<p><strong>Ollama is a local LLM model runner which works in command line interfaces. These models will output text in md format when it needs to structure the output but the command line doesn’t render the markdown content.</strong></p>

<p>BTW, there is already a tool that does this <a href="https://github.com/open-webui/open-webui" rel="noopener noreferrer">https://github.com/open-webui/open-webui</a>. But what’s the fun in that.</p>

<h2>
  
  
  What to learn
</h2>

<ul>
<li>Working with Ollama API</li>
<li>React/Next.js or any front-end framework you like or want to learn.
Prompt engineering fundamentals</li>
</ul>

<h3>
  
  
  How to approach
</h3>

<ul>
<li>Spin up a small web app that interacts with your local Ollama server.</li>
<li>Add features like conversation history, model switching, and file uploads.</li>
<li>Keep it lightweight and internal for personal productivity.</li>
</ul>

<h3>
  
  
  Potential
</h3>

<ul>
<li>A custom AI playground tailored to your needs.</li>
<li>Gateway to experimenting with AI tools without leaving the browser.</li>
</ul>

<h2>
  
  
  Retrain an LLM to Talk Like You
</h2>

<p><strong>This one’s pretty self-explanatory. I’ve wanted to build it since my first year of university, after reading a post about a Chinese guy who created a tool to reply to his girlfriend, and she didn’t realize for an entire year.</strong></p>

<p>I’m still not sure how he pulled that off without the advancements we have in LLMs today (and I don’t recall seeing the word LLM in that post either), but still… it’s a pretty cool project to try.</p>

<h3>
  
  
  What to learn
</h3>

<ul>
<li>Fine-tuning LLMs</li>
<li>Dataset creation &amp; cleaning</li>
<li>Messaging API integrations (WhatsApp via Twilio, Telegram bots, Signal API) to read and respond to the texts.</li>
</ul>

<h3>
  
  
  How to approach
</h3>

<ul>
<li>Collect your writing/chat history (with consent where required).</li>
<li>Fine-tune a small open-source model to mimic your tone and style.</li>
<li>Deploy the model on a server and connect it to WhatsApp or other messaging platforms.</li>
</ul>

<h3>
  
  
  Potential
</h3>

<ul>
<li>Your own AI assistant that sounds like you.</li>
<li>Can help with drafting messages, reminders, or automating repetitive responses.</li>
<li>An advanced AI + automation project that’s a great portfolio talking point.</li>
</ul>

<blockquote>
<p>Some of the projects mentioned here involve data you may or may not have permission to access. Always communicate with the right stakeholders and make sure you’re following applicable policies and laws.</p>
</blockquote>

<p>Stay Curious. Adios 👋</p>

<p><em>Cover image generated with ChatGPT using DALL.E</em></p>

