---
Title: Building my Portfolio Site in 2 Days Using Google AI Tools: Gemini CLI, Antigravity, Conductor, and Agent Starter Pack
Description: 
Author: Darren "Dazbo" Lester
Date: 2026-01-25T21:08:12.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/new-year-new-you-google-ai-2025-12-31">New Year, New You Portfolio Challenge Presented by Google AI</a></em></p>

<h2>
  
  
  About Me
</h2>

<p>My name is Darren, aka Dazbo. I am an Enterprise Cloud Architect and a huge fan of all things Google Cloud and Google AI. I enjoy spreading the word about how cool this technology is. For example, by blogging on platforms like this.</p>

<p>I'm also a <a href="https://developers.google.com/community/experts" rel="noopener noreferrer">Google Developer Expert</a>. But don't let that fool you - I don't write much code! Certainly not in my day job; so I spend a lot of time in the evenings and weekends learning, experimenting, building and blogging. I love blogging for a few reasons:</p>

<ol>
<li>It's part of my learning process; it helps me assimilate information. (And I'm a learning addict!)</li>
<li>It's something I can come back to later. It helps me remember what I did, and why.</li>
<li>I love to share knowledge.</li>
</ol>

<p>Okay, now you know a little about me. Let's take a look at the portfolio site...</p>

<h2>
  
  
  Portfolio
</h2>

<p>My shiny new portfolio application has been deployed to Cloud Run, as required by this challenge. But please note I have set the <code>minimum instances</code> parameter to 0, meaning that if there's no recent requests for the site, then Cloud Run will have spun down to 0. This is good because it means I'm not paying for Cloud Run when it's not in use. But it also means that if it hasn't been used for a while, Cloud Run must "start up from cold", and this takes several seconds. (See more on how I'm mitigating the cold start problem later in this blog.)</p>

<p>Here's the embedded application:</p>


<div class="ltag__cloud-run">
  <iframe height="600px" src="https://dazbo-portfolio-96532439111.europe-west1.run.app">
  </iframe>
</div>


<p>Note that if you <a href="https://darrenlester.net" rel="noopener noreferrer">view this in a browser</a> on your desktop, you'll see multiple tiles in each horizontal row. But the embedded / mobile view only shows a single tile per row.</p>

<h2>
  
  
  A Few Quick Stats
</h2>

<p>I started this project on Saturday, Jan 17. It was mostly done by Sunday night, except for some tweaks. In short: <em>the bulk of this application was created in one weekend.</em> It's taken approximately 14 hours to design, document, build, test, and deploy this working application. That includes about three hours to write this blog.</p>

<p>Here are some stats:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb2x7erftaoxtd4lxl7x5.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb2x7erftaoxtd4lxl7x5.jpg" alt="A few stats" width="800" height="649"></a></p>

<p>The important takeaway here: <strong>there's no way I could have done this so quickly without massive contributions from my Google AI tools!</strong> (More on that in a minute.)</p>

<h2>
  
  
  Application Specifics
</h2>

<p>Funnily enough, building a portfolio website has been on my todo list for a little while. But when this <a href="https://dev.to/challenges/new-year-new-you-google-ai-2025-12-31">challenge</a> popped up in my feed, I obviously had to move this work to the top of my list!</p>

<h3>
  
  
  Broad Goals
</h3>

<p>The portfolio application should:</p>

<ul>
<li>Present a consolidated view of my blogs (from Medium and dev.to), my public GitHub repos, and my deployed applications.</li>
<li>Provide a chatbot interface with my persona, such that users can ask questions about my professional capabilities or my portfolio.</li>
</ul>

<h3>
  
  
  More Detailed Requirements
</h3>

<p>Let's take a look at my specific requirements...</p>

<h4>
  
  
  Functional
</h4>

<ul>
<li>Each portfolio "source" - i.e. blogs, GitHub repos, and applications - should be presented in the form of an interactive carousel on the UI.</li>
<li>There should be a tool to ingest source content into the application. I.e. we should not need to upload content manually, or update databases manually.</li>
<li>Ingestion should be idempotent.</li>
<li>AI should be used to provide automatic summaries of ingested content.</li>
<li>AI should be used to create a markdown version of source blogs. This will be used for future use cases:

<ul>
<li>As a source for embeddings, which will provide our chatbot with RAG.</li>
<li>To allow me to create markdown versions that can be used for cross-posting to blogging sites.</li>
</ul>


</li>

<li>The metadata and AI-generated summaries for the ingested content should be persisted in a database.</li>

<li>The Chatbot should adhere to the "Dazbo" persona.</li>

<li>The Chatbot should be informed by the database.</li>

<li>The Chatbot should only discuss relevant topics.</li>

<li>The application should implement SEO best practices to ensure public discoverability.</li>

</ul>

<h4>
  
  
  Non-Functional / ASRs
</h4>

<ul>
<li>The Chatbot should defend against prompt-injection attacks.</li>
<li>The application should be autoscaling and elastic, with a pay-as-you go paradigm.</li>
<li>UI should be fast, responsive and support desktop and mobile views.</li>
<li>Frontend should interact with backend via API.</li>
<li>The API must implement rate limiting, with both "global" rate limiting and more aggressive rate limiting for calls to the LLM.</li>
<li>Infrastructure resources (i.e. Cloud resources) should be deployable in a repeatable, automated way using Infrastructure-as-Code (IaC).</li>
<li>Application changes should automatically be built, tested and deployed through a CI/CD pipeline.</li>
<li>Human approval is required before deploying to live.</li>
<li>Code quality is managed through enforced linting and formatting.</li>
<li>Include safeguards to prevent unexpected Google Cloud costs.</li>
</ul>

<h4>
  
  
  Testing
</h4>

<ul>
<li>Unit tests should provide &gt;80% coverage.</li>
<li>Integration tests should be present for multiple use cases. </li>
</ul>

<h2>
  
  
  How I Built It
</h2>

<p>Here I'll give you an overview of:</p>

<ul>
<li>The development tools and processes I used</li>
<li>The design decisions and rationale</li>
<li>The architecture and tech stack</li>
</ul>

<p>Also, if you want more details, I've included more hands-on information later in this blog. But for now, the quick version...</p>

<h3>
  
  
  Tools and Development
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjnp2tq42x3dy85emwn17.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjnp2tq42x3dy85emwn17.png" alt="Google AI Tools" width="800" height="262"></a></p>

<p>Obviously, one important aspect of this challenge is that the portfolio site <strong>must be built using the help of Google AI tools.</strong> Fortunately, I've been embracing these tools for a little while now, and this challenge presented a great opportunity to describe how we can use some of these tools together in a holistic way. I'll explain why I prefer some tools for certain use cases.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Tool</th>
<th>Brief Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong><a href="https://antigravity.google/" rel="noopener noreferrer">Google Antigravity</a></strong></td>
<td>Google's agent-first next generation development environment. It combines the VS Code experience with deeply integrated agentic workflows. These workflows provide detailed implementation plans, and then provides evidence of the actions taken and functional outcomes.<br><br>We can customise its behaviour, and provide it with reusable rules, workflows (multi-step tasks) and skills (lightweight on-demand context and knowledge). And we can give it access to tools through the addition of MCP servers.<br><br> I have previously written blogs on using Antigravity, like <a href="https://medium.com/google-cloud/working-with-google-antigravity-in-wsl-944c96c949f3" rel="noopener noreferrer">this one, which talks about using Antigravity in the WSL environment</a>.</td>
</tr>
<tr>
<td><strong><a href="https://geminicli.com/" rel="noopener noreferrer">Gemini CLI</a></strong></td>
<td>Google's open-source, terminal-based AI agent, sharing much functionality and configuration with Gemini Code Assist. It is aware of your local OS environment and can execute terminal commands. It has multiple built in tools, such as Google Search, web fetch, and the ability to read and write files.<br><br>You can configure its behaviour and create repeatable commands. You can provide it with external tools (using MCP), and even package your repeatable commands and tools as sharable and easily deployable extensions. And there's already a huge <a href="https://geminicli.com/extensions/" rel="noopener noreferrer">ecosystem of extensions</a> available.<br><br>Unlike a simple chatbot, it can perform complex tasks like complex multi-file repo refactors, system troubleshooting, or even diagnose Google Cloud deployment issues through use of appropriate extensions.<br><br>I've written several blogs about Gemini CLI, like <a href="https://medium.com/google-cloud/creating-an-automated-ui-test-of-your-web-app-in-seconds-with-gemini-cli-and-browsermcp-09cf4afb8940" rel="noopener noreferrer">this one which talks about running automated UI tests with Gemini CLI</a>.</td>
</tr>
<tr>
<td><strong><a href="https://github.com/gemini-cli-extensions/gcloud" rel="noopener noreferrer">Gemini CLI Gcloud Extension</a></strong></td>
<td>Powers-up interactions with Google Cloud. With this extension in place, Gemini doesn't just know about Google Cloud, but can actually navigate and manage it for us. It is able to run <code>gcloud</code> commands, and perform diagnosis and troubleshooting. For example, I can say "Why did my Cloud Run service fail to deploy?" Gemini will then find the logs, interrogate them, tell me what went wrong, and what I need to do to fix it!</td>
</tr>
<tr>
<td><strong><a href="https://github.com/derailed-dash/adk-docs-ext" rel="noopener noreferrer">Gemini CLI ADK-Docs Extension</a></strong></td>
<td>This is an extension I created myself. The ADK is big and constantly evolving. Gemini's innate knowledge of ADK will always be stale, and this extension solves this problem by providing Gemini with guidance for how to retrieve the latest, most up-to-date and relevant ADK documentation. The quality, accuracy and usefulness Gemini CLI's responses to ADK queries is massively improved.<br><br>You can find my extension <a href="https://github.com/derailed-dash/adk-docs-ext" rel="noopener noreferrer">here</a>, and you can you see it is recommended on the official ADK documentation site, <a href="https://google.github.io/adk-docs/tutorials/coding-with-ai/#usage-in-development-tools" rel="noopener noreferrer">here</a>.</td>
</tr>
<tr>
<td><strong><a href="https://developers.googleblog.com/conductor-introducing-context-driven-development-for-gemini-cli/" rel="noopener noreferrer">Gemini CLI Conductor Extension</a></strong></td>
<td>A powerful extension for Gemini CLI that introduces Context-Driven Development (CDD) to your terminal. It shifts AI interaction away from ephemeral, "forgetful" chats and into persistent, repo-based artifacts. By forcing a <em>"measure twice, code once"</em> philosophy, Conductor ensures the agent understands your project’s vision, guidelines, and tech stack before a single line of code is written. It excels at both "greenfield" and "brownfield" projects, acting less like a noob vibe agent and more like a senior engineer who respects your architectural standards and goals.<br><br>The workflow is centered around <em>Tracks</em> — specific objectives that Conductor breaks down into formal specifications (<code>spec.md</code>) and implementation plans (<code>plan.md</code>). As it executes a plan, Conductor provides a high level of accountability: it tracks progress with granular task updates, creates and runs tests to verify each step, enforces human verification, and even manages your Git history by creating commits with detailed notes for every completed action. This creates a permanent, auditable trail of "how and why" changes were made, making your project context a managed, shareable asset.<br><br>Check out  a previous blog I've written about this extension, called <a href="https://medium.com/google-cloud/trying-out-the-new-conductor-extension-in-gemini-cli-0801f892e2db" rel="noopener noreferrer">Trying Out the New Conductor Extension in Gemini CLI — We’re Gonna Add Auth to Our Full Stack</a>.</td>
</tr>
<tr>
<td><strong><a href="https://developers.google.com/gemini-code-assist/docs/review-github-code" rel="noopener noreferrer">Gemini Code Assist on GitHub</a></strong></td>
<td>Enterprise-grade integration that brings Gemini's power directly into your GitHub repos. Unlike simple "PR summarizers," this integration acts as an extremely skilled senior engineer that performs tasks like issues triage and code reviews. It performs these tasks with full awareness of the entire repo.<br><br>And it is trivial to integrate this capability into your CI/CD workflows. You create a PR, and Gemini Code Assist automatically performs the in-depth review. It then makes recommendations, and categorises them based on importance and severity. For example, if you push code that introduces a prompt injection vulnerability, then the Gemini PR review will tell you, and suggest how you can fix it. You can then make your fixes, commit them, and then re-run the review by simply commenting on your PR with the instruction <code>/gemini review</code>. Cool, right?</td>
</tr>
<tr>
<td><strong><a href="https://gemini.google/overview/image-generation/" rel="noopener noreferrer">Nano Banana Pro</a></strong></td>
<td>Insanely powerful image generation. I can't live without it!</td>
</tr>
</tbody>
</table></div>

<p>Great, so we've got all these AI tools. So how to decide which to use for a particular scenario? </p>

<p>Here's my general approach...</p>

<h4>
  
  
  Use Agent-Starter-Pack to Create Your Project Folder
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxui2tpmiieveibadzu1u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxui2tpmiieveibadzu1u.png" alt="Agent Starter Pack" width="800" height="401"></a></p>

<p>Start with ASP to create your initial project workspace. It creates a folder with a working agent, Python <code>pyproject.toml</code> (to manage dependencies and testing configuration), initial <code>README.md</code>, initial <code>.gitignore</code>, a <code>Makefile</code>, Terraform IaC, CI/CD pipeline, and so much more.</p>

<p>For me, this saves quite a bit of time that would otherwise be spent in initial project creation.</p>

<h4>
  
  
  Provide Reusable Context for Gemini
</h4>

<p>Create a project-specific <strong><code>GEMINI.md</code></strong> file. This provides persistent context to Gemini, and it is used by both Antigravity and Gemini CLI. Note that Agent-Starter-Pack creates this file for us, but I always just replace its contents with what I want.</p>

<h4>
  
  
  Making Significant Changes
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmeja0m8q2m4ldx4xsr5e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmeja0m8q2m4ldx4xsr5e.png" alt="Gemini CLI Conductor" width="800" height="400"></a></p>

<p>When I want to add a new feature or do something significant, I'm using <strong>Gemini CLI with the <em>Conductor</em> extension</strong>. In case you're wondering, I tend to run Gemini CLI from my editor's integrated terminal window.</p>

<p>When we first use Conductor for a given project, Conductor gathers loads of information about the project. For example, your goals, how you want the application to behave, the tech stack you want to use, and so on. This context is persisted and then available for <em>every</em> major task (which Conductor calls a "track") that you want to implement.</p>

<p>When we initiate a new track, Conductor asks a bunch of relevant context-aware questions. It allows free-form answers, but always provides some ideas of good answers that we can select from.</p>

<p>It always creates tests before implementing changes, and then re-runs those tests when the changes have been implemented.</p>

<p>Another VERY useful feature: it tracks each-and-every step by updating the track's <code>plan.md</code> file. This has two very powerful benefits:</p>

<ol>
<li>You can shutdown your machine, come back later, and continue where you left off.</li>
<li>Because the state is persisted and automatically checked-in to GitHub, you can continue your development on another machine. I'm always swapping between my desktop and laptop, so I find this invaluable. This saves several minutes of context building with each development session.</li>
</ol>

<p>Also, since my Gemini CLI is empowered with the <code>gcloud</code> extension and my <code>ADK Docs</code> extension, it has deep knowledge of how to work with Google Cloud and ADK. So, for example, if I have any Google Cloud deployment or IAM issues, I know Gemini CLI will help me diagnose and fix in no time at all.</p>

<h4>
  
  
  For Simpler Changes
</h4>

<p>For less significant changes, I tend to use the Agent built-in to <strong>Antigravity</strong>. Sure, Antigravity still plans, executes and verifies. But if I don't need state to be persisted between sessions (or machines) and if I don't want the overhead of using test-driven development, I'll avoid using Conductor. Conductor is <em>great</em>. But it's much slower than just using the agent in Antigravity.</p>

<h4>
  
  
  Code Reviews
</h4>

<p>Having <strong>Gemini Code Assist active as a CI/CD workflow in GitHub</strong> is incredibly useful. When I'm finished with my current branch (because I always use a separate branch for features) and I raise my PR, Gemini provides a comprehensive code review in the PR itself! I find this a very useful line of defence.  It often picks up stuff that Gemini doesn't spot <em>inside</em> my development environment.</p>

<h4>
  
  
  Documentation
</h4>

<p>I've maintained some key documents throughout, and the rules in my <code>GEMINI.md</code> help ensure they are kept current with each and every change that Gemini helps me with. E.g.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[previous content]

## Key Docs

- @README.md - repo overview and dev guidance
- @GEMINI.md - guidance for you, the agent
- @conductor/product.md - an overview of this portfolio site as a "product"
- @conductor/tech-stack.md - an overview of the tech stack
- @TODO.md - list of tasks to complete
- @Makefile - dev commands
- @docs/design-and-walkthrough.md - design and walkthrough, including design decisions and implementation
- @docs/testing.md - testing docs, including descriptions of all tests
- @deployment/README.md - deployment docs

## Rules

[previous rules]
- ALWAYS use the adk-docs-mcp tools to answer questions about building agents with ADK. If you can't use this MCP, you MUST alert me rather than falling back to what you know.
- Key docs should be updated as you make changes.
- Always include top-of-file docstrings in every Python file you create or edit. This should include a description of what the file does, why it exists, and how it works.

[Rest of this doc]
</code></pre>

</div>



<h4>
  
  
  TODO.md
</h4>

<p>One thing I always create at the <em>very start</em> of a project is a <code>TODO.md</code>. Initially, the <code>TODO.md</code> I created for this project looked like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>- [ ] Create README.md and GEMINI.md
- [ ] Create design-and-walkthrough doc
- [ ] Create testing doc
- [ ] Implement Conductor
- [ ] Add Gemini PR GitHub Actions
- [ ] Ensure Firestore in Terraform deployment; remove any Cloud SQL
- [ ] Add Google Cloud billing alerts and killswitch
- [ ] Establish how to ingest/point to source blogs, repos, etc
- [ ] Add GCS bucket for static assets
- [ ] Implement backend services
- [ ] Implement FastAPI
- [ ] Deploy with Terraform
- [ ] Create React UI with carousel for blogs, GitHub repos, applications, etc.
- [ ] Build container image and test locally.
- [ ] Deploy and test on Cloud Run.
- [ ] Add conversational agent using Dazbo persona. Store the persona as a Google Secret.
- [ ] Persona can answer portfolio questions and questions about me.
- [ ] Add rate limiting.
- [ ] Map to my domain name.
- [ ] UI experimentation / aesthetics.
- [ ] Add AI summarisation of ingested material.
- [ ] Add AI creation of markdown and generation of keywords.
- [ ] Add SEO.
- [ ] Implement RAG with Vector Search
</code></pre>

</div>



<p>It then grew as the project moved towards completion.</p>

<h3>
  
  
  Design Decisions
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8uh22ryo1vuug9dl6k51.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8uh22ryo1vuug9dl6k51.jpg" alt="Design Decisions" width="800" height="446"></a></p>

<p>I made several of these decisions up-front. In some cases, AI (particularly the Conductor extension) helped me make a selection from options. And in some cases, design decisions evolved and changed as the application was created.</p>

<p>Here's a summary of some of the more significant decisions:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Decision</th>
<th>Rationale</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Gemini for LLM</strong></td>
<td>Google's best-in-class multimodal model with its famous 1M+ token context window. I'm using <code>gemini-3-flash-preview</code> at this time. Being fast is more important than powerful reasoning capabilities. (Not that Gemini 3 Flash can't do powerful reasoning!!)</td>
</tr>
<tr>
<td><strong>ADK for agent framework</strong></td>
<td>Provides a production-grade foundation for agent orchestration. Provides the ability to orchestrate across multiple agents, manage context and artifacts, provides agentic evaluation tools, and provides convenient developer tools for interacting with agents.</td>
</tr>
<tr>
<td><strong>AI-Powered Summary Creation</strong></td>
<td>Use Gemini to generate concise technical summaries from ingested blogs. Because why do these by hand?</td>
</tr>
<tr>
<td><strong>AI-Powered Markdown Creation</strong></td>
<td>Use Gemini to generate structured Markdown from raw blog HTML. Because Gemini is really good at this sort of thing!</td>
</tr>
<tr>
<td><strong>Terraform for infrastructure deployment</strong></td>
<td>Provides declarative Infrastructure as Code (IaC), allowing automated, repeatable, versioned deployment of my Google configuration, IAM and services.</td>
</tr>
<tr>
<td><strong>Google Cloud Build for CI/CD</strong></td>
<td>A fully managed, serverless CI/CD platform that integrates seamlessly with both Google Cloud services and with GitHub. So any changes pushed to GitHub automatically result in a new image builds, automated testing, and deployment to Cloud Run.</td>
</tr>
<tr>
<td><strong>Cloud Run for the Application</strong></td>
<td>A fully managed serverless container runtime platform that scales to zero (which is very cost-effective) and handles autoscaling automatically. It also supports custom domains without the need for a Load Balancer. Note that to mitigate the "cold start" that results from me allowing Cloud Run to scale to 0, I'm using <code>CPU boost</code> to speed up startup.</td>
</tr>
<tr>
<td><strong>Cloud Run Domain Mapping</strong></td>
<td>Allows me to map my custom domain directly to my Cloud Run service, removing the need for a Load Balancer.</td>
</tr>
<tr>
<td><strong>Unified Container Image</strong></td>
<td>Packaging the frontend, API, backend, and agent into a single container allows atomic deployments, and greatly simplifies the overall solution and deployment process.</td>
</tr>
<tr>
<td><strong>Unified Origin Architecture</strong></td>
<td>Serving React static assets directly from the FastAPI backend (acting as the origin) completely eliminates CORS complexity in production and simplifies cookie handling.</td>
</tr>
<tr>
<td><strong>Firestore for the Database</strong></td>
<td>A serverless, autoscaling, NoSQL document database chosen for its flexibility with semi-structured data (blogs, projects). Also, we can use it to store embeddings when we implement RAG later.<br><br>It has a generous free tier which I don't expect to exceed. Consequently, for the relatively low demands of this application, it will be significantly cheaper than deploying, say, a Cloud SQL Postgres database, where we have to pay for the always-on infrastructure and the storage.</td>
</tr>
<tr>
<td><strong>Cloud Storage (GCS)</strong></td>
<td>Scalable, serverless, no-ops object storage, well-suited for storage of unstructured data. I will use it to store my small number of static assets, e.g. images used by the UI.</td>
</tr>
<tr>
<td><strong><code>/api</code> Prefix</strong></td>
<td>Establishes a strict routing namespace: <code>/api</code> for backend services; all other routes fallback to the SPA (<code>index.html</code>).</td>
</tr>
<tr>
<td><strong>ADK <code>InMemorySessionService</code></strong></td>
<td>Sessions are designed to be ephemeral and I have no need for any sort of session persistence or HA. An in-memory store offers the lowest possible latency and simplest implementation without needing external persistence like Redis.</td>
</tr>
<tr>
<td><strong>FastAPI for backend</strong></td>
<td>Chosen for its high-performance async capabilities, automatic OpenAPI documentation, and native Pydantic integration, ensuring strict type validation across the API surface.</td>
</tr>
<tr>
<td><strong>In-Memory Rate Limiting</strong></td>
<td>Implemented via <code>slowapi</code> to provide essential DoS protection and cost control for the LLM. At our current scale, this avoids the operational overhead of a dedicated Redis cluster.</td>
</tr>
<tr>
<td><strong><code>uv</code> for Package Management</strong></td>
<td>Replaces both <code>pip</code> and <code>poetry</code> with a single, ultra-fast (Rust-based) tool for dependency resolution and environment management, ensuring deterministic builds.</td>
</tr>
<tr>
<td><strong>Hybrid Ingestion for Medium</strong></td>
<td>Medium has an RSS feed that exports blogs. However, the RSS feed only returns the last 10 blogs. To work around this limitation I'm combining RSS feed and the ability to read a Medium Zip Archive.</td>
</tr>
<tr>
<td><strong>React for frontend</strong></td>
<td>The industry standard for dynamic UIs. Its declarative component model efficiently handles complex states (like real-time chat and dynamic content filters) and benefits from a massive ecosystem.</td>
</tr>
<tr>
<td><strong>Vite for frontend build</strong></td>
<td>Offers instant Hot Module Replacement and optimized production builds using Javascript ES modules, significantly outperforming legacy Webpack-based tools in developer experience and build speed. Efficient delivery to the client</td>
</tr>
<tr>
<td><strong>React 19 Native Metadata</strong></td>
<td>Leverages built-in hoisting for <code>&lt;title&gt;</code> and <code>&lt;meta&gt;</code> tags, eliminating the need for external libraries like <code>react-helmet</code>.</td>
</tr>
<tr>
<td><strong>Use budget alerts and my central "Killswitch" project</strong></td>
<td>My <a href="https://medium.com/google-cloud/how-to-avoid-a-massive-cloud-bill-41a76251caba" rel="noopener noreferrer">Killswitch project</a> automatically disconnects billing from a project, if that project exceeds its spend limit. I use this mechanism to keep a lid on my project spend and avoid unexpected cloud bills.</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  Deployment Architecture
</h4>

<p>This is covered in the design decisions, but to summarise and recap:</p>

<ul>
<li>The frontend, API and backend are deployed into a single container, hosted on Google Cloud Run.</li>
<li>Ingested content, extracted metadata and AI-generated enrichment is stored in Firestore.</li>
<li>Static assets are stored in Google Cloud Storage.</li>
<li>Logging is sent to Google Cloud Logging.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu54slcfspfydgzffn7gr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu54slcfspfydgzffn7gr.png" alt="Deployment Architecture" width="800" height="484"></a></p>

<h4>
  
  
  Application Architecture
</h4>

<p>The application contains two fundamental components:</p>

<p><strong>The containerised user-facing application</strong>, composed of:</p>

<ul>
<li>The React UI (frontend)</li>
<li>The FastAPI layer</li>
<li>The service-oriented async backend</li>
<li>The Gemini agent chatbot</li>
</ul>

<p><strong>The Ingestion Toolkit</strong> - a CLI that allows me to load source material into the database. It is intended for use <em>only by me</em>, and not by end users. For this reason, it is not packaged into the container.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxx0x7984wqguu8nwuzhl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxx0x7984wqguu8nwuzhl.png" alt="Software Architecture" width="800" height="646"></a></p>

<h2>
  
  
  Recap
</h2>

<p>Well, that's it for the overview of my new portfolio application! I've made use of a bunch of Google AI tools to create it, and this has made it possible to build the application in a couple of days, rather than months!</p>

<p>In particular, the AI tools have:</p>

<ul>
<li>Helped me create the initial project scaffold quickly.</li>
<li>Helped me create a full-stack portfolio application, with React/Vite frontend, API, Python backend and Gemini-based agentic components.</li>
<li>Ensured continuous alignment to my goals and requirements, architectural standards and coding rules.</li>
<li>Ensured that I've keep my documentation complete and up-to-date.</li>
<li>Ensured that I have an extensive high-coverage testing suite, which is run as part of my CI/CD pipeline.</li>
</ul>

<p>I should add that I have virtually zero knowledge of building frontends with React. Creating this responsive and cool looking UI by hand would have taken me a lot of effort, time and learning. Yeah - I love to learn. <em>But there's only so many hours in a day, and sometimes I just want to leverage AI so I don't have to!</em></p>

<p>Well, that's it for the brief! But if you're interested in more detail, feel free to check out the <em>Deep Dive</em> below.</p>

<h2>
  
  
  What's Next for the Application?
</h2>

<p>My ingestion tool creates AI summaries and stores the entire blogs in the database in markdown. The next obvious step is to take these summaries and markdown, and create embeddings from them. We can store these embeddings in the same Firestore database, index them, and then use them to allow my chatbot to perform RAG. That way, the chatbot will be able to have much more meaningful understanding of the details of my blogs and repos.</p>

<h2>
  
  
  What I'm Most Proud Of
</h2>

<p>From this portfolio experience?</p>

<p>I guess...</p>

<ul>
<li>That this isn't just a vibe solution. This is (I hope) well-architected,   well-documented, production-ready, with clean, maintainable code.</li>
<li>That I've managed to pull together (hopefully) a coherent story for how we can make use of a collection of Google AI tools; with clear guidance on which tools are suited for a particular use case. When there are so many tools, it can be hard to know which ones to use.</li>
<li>That it looks so cool!</li>
</ul>

<h2>
  
  
  AI Comedy Moments
</h2>

<p>Whilst helping me build this portfolio, Gemini CLI found itself occasionally frustrated. Here's a couple of screenshots that made me laugh:</p>

<p><strong>I am an absolute muppet</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fot8pkfqxzyau00plnbiw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fot8pkfqxzyau00plnbiw.png" alt="Muppet" width="753" height="251"></a></p>

<p>And then later, when it made the same mistake again:</p>

<p><strong>I am going to smash my keyboard</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4p1zdx71ujin9rowimy3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4p1zdx71ujin9rowimy3.png" alt="Smash keyboard" width="669" height="125"></a></p>

<p>It seems that Gemini's internal dialog is pretty similar to mine!!</p>

<h2>
  
  
  Diving Deeper
</h2>

<p>Consider everything from here onwards as erm, <em>bonus content</em>.</p>

<p>I'll provide some more detail for a few steps in this journey. It's not going to be a <em>complete walkthrough</em>, but it should hopefully be useful if you want to replicate parts of the journey.</p>

<p>Let's go!</p>

<h3>
  
  
  Create a Google Cloud Project
</h3>

<p>First we need a Google Cloud project to host our Cloud Run service. I went ahead and created this in the Google Cloud console. </p>

<p>Next we need to configure our <code>gcloud</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">GOOGLE_CLOUD_PROJECT</span><span class="o">=</span><span class="s2">"my-project-name"</span>
gcloud auth login <span class="nt">--update-adc</span>
gcloud config <span class="nb">set </span>project <span class="nv">$GOOGLE_CLOUD_PROJECT</span>
gcloud auth application-default set-quota-project <span class="nv">$GOOGLE_CLOUD_PROJECT</span>
</code></pre>

</div>



<p>Next let's enable a minimal set of APIs, in order to:</p>

<ul>
<li>Use Gemini API</li>
<li>Make use of Gemini Cloud Assist</li>
<li>Use Cloud Build to push images to Google Artifact Registry, and to deploy to Cloud Run</li>
<li>Enable Secret Manager</li>
</ul>

<p>Here's how:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gcloud services <span class="nb">enable</span> <span class="nt">--project</span><span class="o">=</span><span class="nv">$GOOGLE_CLOUD_PROJECT</span> <span class="se">\</span>
  artifactregistry.googleapis.com <span class="se">\</span>
  cloudbuild.googleapis.com <span class="se">\</span>
  secretmanager.googleapis.com <span class="se">\</span>
  run.googleapis.com <span class="se">\</span>
  logging.googleapis.com <span class="se">\</span>
  aiplatform.googleapis.com <span class="se">\</span>
  serviceusage.googleapis.com <span class="se">\</span>
  storage.googleapis.com <span class="se">\</span>
  cloudtrace.googleapis.com <span class="se">\</span>
  geminicloudassist.googleapis.com
</code></pre>

</div>



<h3>
  
  
  Create an API Key
</h3>

<p>I'm going to use a Gemini API key in order to use the Gemini API. An easy way to do this is to use <a href="https://aistudio.google.com/" rel="noopener noreferrer">Google AI Studio</a>.</p>

<ol>
<li>Open <a href="https://aistudio.google.com/api-keys" rel="noopener noreferrer">https://aistudio.google.com/api-keys</a>. </li>
<li>From there we import our newly created Google Cloud project. Then, create a new Gemini API key, connected to that project.</li>
</ol>

<h3>
  
  
  Create Your Local Development Workspace
</h3>

<p>Now we create a local project folder. A really quick way to get up and running is with the <a href="https://github.com/GoogleCloudPlatform/agent-starter-pack" rel="noopener noreferrer">Agent Starter Pack</a>. This creates a project for us, deploys a basic agent from a template, sets up initial tests, creates Jupyter notebooks for experimentation, sets up CI/CD, and so on:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>uvx agent-starter-pack create dazbo_portfolio <span class="nt">-a</span> adk
</code></pre>

</div>



<p>The <code>agent-starter-pack</code> prompts for input at various stages. These are the choices I selected:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Consideration</th>
<th>Selected</th>
<th>Rationale</th>
</tr>
</thead>
<tbody>
<tr>
<td>Agent template</td>
<td><code>adk</code></td>
<td>Provides an out-of-the-box Gemini chat agent, using Google ADK</td>
</tr>
<tr>
<td>Deployment target</td>
<td>Google Cloud Run</td>
<td>Fully-managed, serverless, no-ops, elastic container hosting; and required for this challenge</td>
</tr>
<tr>
<td>Session type</td>
<td><code>in_memory</code></td>
<td>I have no need to persist sessions; this option is the most simple to implement, requiring no additional services</td>
</tr>
<tr>
<td>CI/CD</td>
<td>Google Cloud Build</td>
<td>Out-of-the-box CI/CD pipeline</td>
</tr>
<tr>
<td>Region</td>
<td><code>europe-west1</code></td>
<td>Close to home so low latency; also, I know I won't have any Cloud Build quota issues with this region</td>
</tr>
</tbody>
</table></div>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi2vb1uz33l3gjccs0b3z.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi2vb1uz33l3gjccs0b3z.png" alt="Project created" width="800" height="233"></a></p>

<h3>
  
  
  Environment Setup
</h3>

<p>I like to create a <code>.env.template</code>. This includes the environment variables I want, and it's safe to check-in.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">REPO</span><span class="o">=</span><span class="s2">"repo-name"</span>
<span class="nb">export </span><span class="nv">SERVICE_NAME</span><span class="o">=</span><span class="s2">"service-name"</span>

<span class="nb">export </span><span class="nv">GOOGLE_CLOUD_PROJECT</span><span class="o">=</span><span class="s2">"&lt;your-gcp-project&gt;"</span>
<span class="nb">export </span><span class="nv">GOOGLE_CLOUD_REGION</span><span class="o">=</span><span class="s2">"&lt;your-region&gt;"</span> <span class="c"># For Google Cloud resources</span>
<span class="nb">export </span><span class="nv">GOOGLE_CLOUD_LOCATION</span><span class="o">=</span><span class="s2">"global"</span> <span class="c"># For Gemini model - you can use "global"</span>

<span class="c"># Firestore</span>
<span class="nb">export </span><span class="nv">FIRESTORE_DATABASE_ID</span><span class="o">=</span><span class="s2">"(default)"</span>

<span class="c"># For CI/CD with Cloud Build SA</span>
<span class="nb">export </span><span class="nv">CB_SA_EMAIL</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">PROJECT_NUMBER</span><span class="k">}</span><span class="s2">@cloudbuild.gserviceaccount.com"</span>

<span class="c"># Agent SA</span>
<span class="nb">export </span><span class="nv">SERVICE_SA</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">SERVICE_NAME</span><span class="k">}</span><span class="s2">-app"</span>
<span class="nb">export </span><span class="nv">SERVICE_SA_EMAIL</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">SERVICE_SA</span><span class="k">}</span><span class="s2">@</span><span class="k">${</span><span class="nv">GOOGLE_CLOUD_PROJECT</span><span class="k">}</span><span class="s2">.iam.gserviceaccount.com"</span>

<span class="c"># Agent</span>
<span class="nb">export </span><span class="nv">GEMINI_API_KEY</span><span class="o">=</span><span class="s2">"&lt;your-gemini-api-key&gt;"</span>
<span class="nb">export </span><span class="nv">LOG_LEVEL</span><span class="o">=</span><span class="s1">'DEBUG'</span>
<span class="nb">export </span><span class="nv">APP_NAME</span><span class="o">=</span><span class="s2">"app_name"</span>
<span class="nb">export </span><span class="nv">AGENT_NAME</span><span class="o">=</span><span class="s2">"chat_agent_name"</span>
<span class="nb">export </span><span class="nv">GOOGLE_GENAI_USE_VERTEXAI</span><span class="o">=</span><span class="s2">"False"</span> <span class="c"># Use Vertex AI for auth, not API key</span>
<span class="nb">export </span><span class="nv">MODEL</span><span class="o">=</span><span class="s2">"gemini-3-flash-preview"</span>

<span class="c"># URLs</span>
<span class="nb">export </span><span class="nv">BASE_URL</span><span class="o">=</span><span class="s2">"https://&lt;your-domain&gt;"</span>
<span class="nb">export </span><span class="nv">MEDIUM_PROFILE</span><span class="o">=</span><span class="s2">"https://medium.com/@user-name"</span>
<span class="nb">export </span><span class="nv">DEVTO_PROFILE</span><span class="o">=</span><span class="s2">"https://dev.to/user-name"</span>
</code></pre>

</div>



<p>I then typically make a copy called <code>.env</code> that is not checked-in, and populate with my required values.</p>

<p>I've also created a <code>scripts\setup-env.sh</code> to automate setting up the environment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># This script is meant to be sourced to set up your development environment.</span>
<span class="c"># It configures gcloud, installs dependencies, and activates the virtualenv.</span>
<span class="c">#</span>
<span class="c"># Usage:</span>
<span class="c">#   source ./setup-env.sh [--noauth]</span>
<span class="c">#</span>
<span class="c"># Options:</span>
<span class="c">#   --noauth: Skip gcloud authentication.</span>

<span class="c"># --- Color and Style Definitions ---</span>
<span class="nv">RESET</span><span class="o">=</span><span class="s1">'\033[0m'</span>
<span class="nv">BOLD</span><span class="o">=</span><span class="s1">'\033[1m'</span>
<span class="nv">RED</span><span class="o">=</span><span class="s1">'\033[0;31m'</span>
<span class="nv">GREEN</span><span class="o">=</span><span class="s1">'\033[0;32m'</span>
<span class="nv">YELLOW</span><span class="o">=</span><span class="s1">'\033[0;33m'</span>
<span class="nv">BLUE</span><span class="o">=</span><span class="s1">'\033[0;34m'</span>

<span class="c"># --- Parameter parsing ---</span>
<span class="nv">AUTH_ENABLED</span><span class="o">=</span><span class="nb">true

</span><span class="k">while</span> <span class="o">[[</span> <span class="nv">$# </span><span class="nt">-gt</span> 0 <span class="o">]]</span><span class="p">;</span> <span class="k">do
    case</span> <span class="s2">"</span><span class="nv">$1</span><span class="s2">"</span> <span class="k">in</span>
        <span class="nt">--noauth</span><span class="p">)</span>
            <span class="nv">AUTH_ENABLED</span><span class="o">=</span><span class="nb">false
            shift</span>
            <span class="p">;;</span>
        <span class="k">*</span><span class="p">)</span>
            <span class="nb">shift</span>
            <span class="p">;;</span>
    <span class="k">esac</span>
<span class="k">done

</span><span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BLUE</span><span class="k">}${</span><span class="nv">BOLD</span><span class="k">}</span><span class="s2">--- ☁️  Configuring Google Cloud environment ---</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>

<span class="c"># 1. Check for .env file</span>
<span class="k">if</span> <span class="o">[</span> <span class="o">!</span> <span class="nt">-f</span> .env <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="k">${</span><span class="nv">RED</span><span class="k">}</span><span class="s2">❌ Error: .env file not found.</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>
    <span class="nb">echo</span> <span class="s2">"Please create a .env file with your project variables and run this command again."</span>
    <span class="k">return </span>1
<span class="k">fi</span>

<span class="c"># 2. Source environment variables and export them</span>
<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"Sourcing variables from </span><span class="k">${</span><span class="nv">BLUE</span><span class="k">}</span><span class="s2">.env</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2"> file..."</span>
<span class="nb">set</span> <span class="nt">-a</span> <span class="c"># automatically export all variables (allexport = on)</span>
<span class="nb">source</span> .env
<span class="nb">set</span> +a <span class="c"># disable allexport mode</span>

<span class="c"># 3. Authenticate with gcloud and configure project</span>
<span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$AUTH_ENABLED</span><span class="s2">"</span> <span class="o">=</span> <span class="nb">true</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="s2">🔐 Authenticating with gcloud and setting project to </span><span class="k">${</span><span class="nv">BOLD</span><span class="k">}</span><span class="nv">$GOOGLE_CLOUD_PROJECT</span><span class="s2">...</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>
    gcloud auth login <span class="nt">--update-adc</span> 2&gt;&amp;1 | <span class="nb">grep</span> <span class="nt">-v</span> <span class="nt">-e</span> <span class="s1">'^$'</span> <span class="nt">-e</span> <span class="s1">'WSL'</span> <span class="nt">-e</span> <span class="s1">'xdg-open'</span> <span class="c"># Suppress any annoying WSL messages</span>
    gcloud config <span class="nb">set </span>project <span class="s2">"</span><span class="nv">$GOOGLE_CLOUD_PROJECT</span><span class="s2">"</span>
    gcloud auth application-default set-quota-project <span class="s2">"</span><span class="nv">$GOOGLE_CLOUD_PROJECT</span><span class="s2">"</span>
<span class="k">else
    </span><span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="k">${</span><span class="nv">YELLOW</span><span class="k">}</span><span class="s2">Skipping gcloud authentication as requested.</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>
    gcloud config <span class="nb">set </span>project <span class="s2">"</span><span class="nv">$GOOGLE_CLOUD_PROJECT</span><span class="s2">"</span>
<span class="k">fi

</span><span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="k">${</span><span class="nv">BLUE</span><span class="k">}</span><span class="s2">--- Current gcloud project configuration ---</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>
gcloud config list project
<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BLUE</span><span class="k">}</span><span class="s2">------------------------------------------</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>

<span class="nb">export </span><span class="nv">PROJECT_NUMBER</span><span class="o">=</span><span class="si">$(</span>gcloud projects describe <span class="nv">$GOOGLE_CLOUD_PROJECT</span> <span class="nt">--format</span><span class="o">=</span><span class="s2">"value(projectNumber)"</span><span class="si">)</span>
<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BOLD</span><span class="k">}</span><span class="s2">PROD_PROJECT_NUMBER:</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">  </span><span class="nv">$PROJECT_NUMBER</span><span class="s2">"</span>
<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="k">${</span><span class="nv">BLUE</span><span class="k">}</span><span class="s2">------------------------------------------</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>

<span class="c"># For CI/CD with Cloud Build SA</span>
<span class="nb">export </span><span class="nv">CB_SA_EMAIL</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">PROJECT_NUMBER</span><span class="k">}</span><span class="s2">@cloudbuild.gserviceaccount.com"</span>

<span class="c"># 6. Sync Python dependencies and activate venv</span>
<span class="nb">echo</span> <span class="s2">"Syncing python dependencies with uv..."</span>
uv <span class="nb">sync</span> <span class="nt">--dev</span> <span class="nt">--extra</span> jupyter
<span class="nb">source</span> .venv/bin/activate

<span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="se">\n</span><span class="k">${</span><span class="nv">GREEN</span><span class="k">}</span><span class="s2">✅ Environment setup complete for project </span><span class="k">${</span><span class="nv">BOLD</span><span class="k">}</span><span class="nv">$GOOGLE_CLOUD_PROJECT</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}${</span><span class="nv">GREEN</span><span class="k">}</span><span class="s2">. Your shell is now configured.</span><span class="k">${</span><span class="nv">RESET</span><span class="k">}</span><span class="s2">"</span>b  
</code></pre>

</div>



<p>And I can automate running this script whenever I open my workspace, by creating this <code>.envrc</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># For use with direnv</span>
<span class="c"># - `sudo apt install direnv`</span>
<span class="c">#</span>
<span class="c"># Add hook to end of .bashrc:</span>
<span class="c"># eval "$(direnv hook bash)"</span>
<span class="c"># </span>
<span class="c"># Allow this folder:</span>
<span class="c"># - `direnv allow`</span>
<span class="c">#</span>
<span class="c"># Script as required...</span>
<span class="k">if</span> <span class="o">[</span> <span class="o">!</span> <span class="nt">-d</span> <span class="s2">".venv"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span>uv venv
<span class="k">fi</span>

<span class="c"># Check if gcloud token is still valid to avoid re-authenticating</span>
<span class="k">if </span>gcloud auth print-access-token <span class="nt">--quiet</span> <span class="o">&gt;</span> /dev/null 2&gt;&amp;1<span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"gcloud token is valid, skipping authentication."</span>
  <span class="nb">source </span>scripts/setup-env.sh <span class="nt">--noauth</span>
<span class="k">else
  </span><span class="nb">echo</span> <span class="s2">"gcloud token is not valid, re-authenticating."</span>
  <span class="nb">source </span>scripts/setup-env.sh
<span class="k">fi</span>
</code></pre>

</div>



<h3>
  
  
  Initial Test
</h3>

<p>At this point, I should have a working agentic sample application. I can test it by running <code>make playground</code>. (This is a handy <code>make</code> target that was created by the ASP.) We can see it start up:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkv5f9n7k4tlim62n561b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkv5f9n7k4tlim62n561b.png" alt="Running ADK Playground" width="800" height="249"></a></p>

<p>And then we can open our browser:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsw9h9jr1ehb8ehiqb2hv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsw9h9jr1ehb8ehiqb2hv.png" alt="ADK Web UI" width="800" height="600"></a></p>

<p>Okay, so far, so good!</p>

<h3>
  
  
  Setting up the Spec with Conductor
</h3>

<p>To setup Conductor for the first time in a repo, we run this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/conductor:setup
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6wd31uru5vtrrv0sl3cp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6wd31uru5vtrrv0sl3cp.png" alt="Brownfield setup" width="800" height="141"></a></p>

<p>It guides us with many questions. like this one to set the tone:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7uyyjkq7j1z6zdzivhvr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7uyyjkq7j1z6zdzivhvr.png" alt="Guidance questions" width="800" height="239"></a></p>

<p>It checks what kind of aesthetic I want from the frontend, and what sort of personality my chat bot will have. (I point it to my existing "Dazbo persona" which I have on GitHub.)</p>

<p>I then proposes a tech stack based on what I've already provided in the <code>README.md</code>, such as the design decisions I had already recorded. I needed to make a couple of tweaks to what Conductor suggested. For example, I want to use Google Firestore for managing portfolio content, rather than Cloud SQL Postgres.</p>

<p>Once the tech stack is confirmed, Conductor pulls in its relevant pre-canned style guides.</p>

<p>Conductor then proposes an initial track, and then commits everything done so far, as per its standard workflow.</p>

<p>At the end of this process, we now have this files in our workspace:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb4hytrqc94p1aa4q5gnx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb4hytrqc94p1aa4q5gnx.png" alt="Conductor files" width="800" height="345"></a></p>

<h3>
  
  
  Setup Automatic Gemini Reviews of GitHub PRs
</h3>

<p>A super useful thing to setup is automatic Gemini reviews of any PRs you submit in GitHub. There's a couple of ways to set this up, but since we already have Gemini CLI running, the easiest way is to run <code>/setup-github</code> from within Gemini CLI. Within a few seconds, it's created the necessary GitHub Actions for us:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4eeltfxxt9pnqts94tpo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4eeltfxxt9pnqts94tpo.png" alt="Gemini GitHub PR" width="800" height="320"></a></p>

<p>We also need to ensure we've created a GitHub repo secret called <code>GEMINI_API_KEY</code> which contains the API key we created earlier.</p>

<p>Now, if we commit changes in a new branch and then raise a PR to merge this back in to main, Gemini will automatically review it for us. Nice!</p>

<h3>
  
  
  Setup Terraform and CI/CD
</h3>

<p>I'd like the deployment of my Google Cloud resources to be consistent and repeatable, so I'm going to use Terraform. The Agent Starter Pack has created some initial Terraform for me, but I've tweaked it for my purposes.</p>

<h4>
  
  
  State Management
</h4>

<p>The first thing I want to do is store my Terraform state in a GCS bucket, rather than in a local state file. Note that we need to create the bucket first and then point Terraform to it. I add this <code>backend.tf</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># =========================================================================</span>
<span class="c1"># Terraform backend configuration</span>
<span class="c1"># =========================================================================</span>

<span class="c1"># Expects the bucket has already been created, e.g.</span>
<span class="c1"># gcloud storage buckets create gs://${GOOGLE_CLOUD_PROJECT}-tf-state</span>
<span class="c1">#</span>
<span class="c1"># To migrate from local state to this GCS backend state, simply re-run terraform init</span>
<span class="c1"># and answer 'yes' to the prompt to copy the existing state to the new backend.</span>

<span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">backend</span> <span class="s2">"gcs"</span> <span class="p">{</span>
    <span class="nx">bucket</span> <span class="p">=</span> <span class="s2">"dazbo-portfolio-tf-state"</span> <span class="c1"># variables not supported here</span>
    <span class="nx">prefix</span> <span class="p">=</span> <span class="s2">"terraform/state"</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  Add Firestore to Terraform Config
</h4>

<p>I want my Terraform to create my Google Firestore database. I could have just done added this additional Terraform configuration by hand, but I've decided to let Conductor do it for me. So I created a dedicated Conductor track.</p>

<p>Conductor adds this section to my <code>storage.tf</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"google_firestore_database"</span> <span class="s2">"database"</span> <span class="p">{</span>
  <span class="nx">project</span>                     <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">project_id</span>
  <span class="nx">name</span>                        <span class="p">=</span> <span class="s2">"(default)"</span>
  <span class="nx">location_id</span>                 <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">region</span>
  <span class="nx">type</span>                        <span class="p">=</span> <span class="s2">"FIRESTORE_NATIVE"</span>
  <span class="nx">delete_protection_state</span>     <span class="p">=</span> <span class="s2">"DELETE_PROTECTION_DISABLED"</span>
  <span class="nx">depends_on</span> <span class="p">=</span> <span class="p">[</span><span class="nx">resource</span><span class="p">.</span><span class="nx">google_project_service</span><span class="p">.</span><span class="nx">deploy_project_services</span><span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<h4>
  
  
  CI/CD
</h4>

<p>We can obviously just deploy our Cloud Run service manually once the application is built. But I might as well get the CI/CD running so that any code changes I make are automatically pushed to Cloud Run.</p>

<p>The goal here is for code changes pushed to GitHub to automatically call a trigger that runs a Google Cloud Build workflow. The Cloud Build workflow will build the container image, push it to Google Container Registry, and from there, deploy it to Google Cloud Run.</p>

<p>ASP has already created initial Cloud Build configuration files, and initial Terraform <code>build_triggers.tf</code>. I just need to tweak them a bit for my purposes.</p>

<h4>
  
  
  Create a Connection from Cloud Build to GitHub
</h4>

<p>In order for GitHub to be able trigger our Cloud Build, we need to establish a connection between them.</p>

<p>We can set this up in the Cloud Console: open Cloud Build, then Repositories, then "Connect repository". Select "Edit repositories on GitHub" which takes you to Settings &gt; Applications &gt; Cloud Build. From here, we should add our new GitHub repo. Once this is saved in GitHub, we can select it back in the Cloud Console.</p>

<p>Now, update <code>deployment/terraform/vars/env.tfvars</code> with the appropriate configuration information, including:</p>

<ul>
<li>
<code>host_connection_name</code> - as shown in Cloud Build &gt; Repositories</li>
<li>
<code>github_pat_secret_id</code> - which will be the name of the appropriate secret that has been created in Secret Manager. It will be called something like: <code>&lt;host_connection_name&gt;-gihub-oauthtoken-1a2b3c</code>
</li>
</ul>

<p>I've created <code>env.tfvars.template</code> that describes the variables I need:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Project name used for resource naming</span>
project_name <span class="o">=</span> <span class="s2">"&lt;project-name&gt;"</span>

<span class="c"># Your Google Cloud Project ID for resource deployment</span>
project_id <span class="o">=</span> <span class="s2">"&lt;your-project-id&gt;"</span>

<span class="c"># Your Google Cloud project ID that will be used to host the Cloud Build pipelines.</span>
cicd_runner_project_id <span class="o">=</span> <span class="s2">"&lt;cicd-runner-project-id&gt;"</span>

<span class="c"># Name of the host connection you created in Cloud Build</span>
connection_already_exists <span class="o">=</span> <span class="nb">true</span> <span class="c"># Do not try to create a new connection</span>
host_connection_name <span class="o">=</span> <span class="s2">"&lt;gh-conn-name&gt;"</span>
github_pat_secret_id <span class="o">=</span> <span class="s2">"&lt;gh-conn-secret-id&gt;"</span>

repository_owner <span class="o">=</span> <span class="s2">"&lt;gh-username&gt;"</span>

<span class="c"># Name of the repository you added to Cloud Build</span>
repository_name <span class="o">=</span> <span class="s2">"&lt;gh-repo-name&gt;"</span>

<span class="c"># The Google Cloud region you will use to deploy the infrastructure</span>
region <span class="o">=</span> <span class="s2">"&lt;gcp-region&gt;"</span>

<span class="c"># Service Configuration</span>
service_name <span class="o">=</span> <span class="s2">"&lt;service-name&gt;"</span> <span class="c"># Note hyphens</span>
app_name <span class="o">=</span> <span class="s2">"&lt;service_name&gt;"</span> <span class="c"># Note underscore</span>
agent_name <span class="o">=</span> <span class="s2">"&lt;service_name&gt;_chat_agent"</span>
google_genai_use_vertexai <span class="o">=</span> <span class="s2">"true"</span>
model <span class="o">=</span> <span class="s2">"&lt;model&gt;"</span>
location <span class="o">=</span> <span class="s2">"global"</span>
app_domain_name  <span class="o">=</span> <span class="o">[</span><span class="s2">"some-domain.com"</span>, <span class="s2">"www.some-domain.com"</span><span class="o">]</span>
</code></pre>

</div>



<p>I don't need to create the Cloud Build triggers; my Terraform will do that for me when I run it later.</p>

<h4>
  
  
  Run the Terraform
</h4>

<p>Of course, we can run Terraform the manual way, by running <code>terraform init</code>, <code>terraform plan</code> and <code>terraform apply</code> commands. But I've created convenient <code>Makefile</code> targets. So we can just run this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># First check it with make tf-plan</span>
make tf-apply
</code></pre>

</div>



<p>After the Terraform apply, our resources have been successfully created. Let's verify the Firestore database has been created:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo6ksla90b4h4venrm3d3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo6ksla90b4h4venrm3d3.png" alt="Firestore created" width="800" height="144"></a></p>

<p>Great!</p>

<p>And now let's check our Cloud Build triggers:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuph8blhjprtd77vulo7u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuph8blhjprtd77vulo7u.png" alt="Cloud Build triggers created" width="800" height="240"></a></p>

<h3>
  
  
  Portfolio Resource Ingestion Strategy
</h3>

<p>Let's have a look at Gemini CLI Conductor executing a significant change for us. In Gemini CLI, we run the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>/conductor:newTrack
</code></pre>

</div>



<p>Conductor asks me what I want to achieve. I tell it:</p>

<p><em>My application needs a way to ingest my upstream resources. I want to store the ingested resource metadata in Google Firestore, and any associated images in GCS. I want the application to be able to trawl my sources, such as GitHub, Medium, and <code>dev.to</code>, using the relevant user ID for me. But for simplicity, I've decided I want this to be handled "out-of-band" to the main application. I.e. I'll just run this ingestion process on-demand, as required.</em></p>

<p>After a number of questions and checks, Conductor creates this initial <code>spec.md</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Specification: Portfolio Resource Ingestion Strategy</span>

<span class="gu">## Overview</span>
This track defines and implements the system for bringing portfolio resources—including blog posts, GitHub repositories, websites, and applications—into the Dazbo Portfolio application. It focuses on a decoupled, "out-of-band" ingestion process via CLI scripts that populate Google Firestore and handle static assets via Google Cloud Storage.

<span class="gu">## Goals</span>
<span class="p">-</span> Establish a "Source of Truth" in Firestore for all showcaseable items.
<span class="p">-</span> Provide a hybrid ingestion mechanism (automated connectors + manual entries).
<span class="p">-</span> Enable on-demand triggering via a dedicated CLI tool.
<span class="p">-</span> Support metadata-only entries for paywalled or external content.

<span class="gu">## Functional Requirements</span>

<span class="gu">### 1. Ingestion Connectors</span>
<span class="p">-</span> <span class="gs">**GitHub Connector:**</span> Fetch public repositories, metadata (stars, description), and primary languages.
<span class="p">-</span> <span class="gs">**Medium Connector:**</span> Fetch blog post metadata (titles, links, summaries).
<span class="p">-</span> <span class="gs">**Dev.to Connector:**</span> Fetch blog post metadata via the Dev.to API.
<span class="p">-</span> <span class="gs">**Custom/Manual Connector:**</span> Support for entries not tied to a specific platform (e.g., private projects, paywalled articles, standalone websites).

<span class="gu">### 2. Ingestion Triggering</span>
<span class="p">-</span> <span class="gs">**CLI Tooling:**</span> A set of Python scripts in <span class="sb">`scripts/`</span> or a dedicated module (e.g., <span class="sb">`app/tools/ingest.py`</span>) to trigger syncs manually from a developer environment.

<span class="gu">### 3. Static Asset Management</span>
<span class="p">-</span> <span class="gs">**Image Storage:**</span> All portfolio-related images (thumbnails, profile pics, screenshots) will be stored in a <span class="gs">**Public Google Cloud Storage (GCS) Bucket**</span>.
<span class="p">-</span> <span class="gs">**Asset Referencing:**</span> Firestore documents will store the public URLs of these GCS objects.

<span class="gu">### 4. Data Architecture</span>
<span class="p">-</span> <span class="gs">**Decoupled Logic:**</span> The ingestion logic will be kept separate from the main FastAPI request-response cycle to minimize dependencies in the production runtime.
<span class="p">-</span> <span class="gs">**Schema Support:**</span> Extend/Verify <span class="sb">`Project`</span> and <span class="sb">`Blog`</span> models to support "manual" flags and "metadata-only" status.

<span class="gu">## Non-Functional Requirements</span>
<span class="p">-</span> <span class="gs">**Resilience:**</span> Handle API rate limiting and transient network errors gracefully.
<span class="p">-</span> <span class="gs">**Maintainability:**</span> Use a modular "Connector" pattern to allow adding new sources easily.
<span class="p">-</span> <span class="gs">**Idempotency:**</span> Ensure that repeated ingestion runs do not create duplicate entries in Firestore.

<span class="gu">## Acceptance Criteria</span>
<span class="p">-</span> [ ] A Python script exists that can sync GitHub, Medium, and Dev.to content to Firestore.
<span class="p">-</span> [ ] Ability to manually add a "Custom" resource entry via a configuration file (e.g., YAML) processed by the CLI.
<span class="p">-</span> [ ] Images successfully uploaded to GCS are accessible via public URL in the app.
<span class="p">-</span> [ ] Integration tests verify that ingestion does not create duplicate entries.

<span class="gu">## Out of Scope</span>
<span class="p">-</span> A full-blown web-based Admin Dashboard (GUI).
<span class="p">-</span> Admin API endpoints.
<span class="p">-</span> Automated crawling of paywalled content body (metadata/stubs only).
<span class="p">-</span> Real-time Webhook-based syncing.
<span class="p">-</span> LinkedIn Connector.
</code></pre>

</div>



<p>I confirm I'm happy with this spec. Conductor then goes on to create an execution plan, as <code>plan.md</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Implementation Plan: Portfolio Resource Ingestion Strategy</span>

<span class="gu">## Phase 1: Foundation &amp; Data Models</span>
<span class="p">-</span> [ ] Task: Update Pydantic models to support ingestion metadata
<span class="p">    -</span> [ ] Add <span class="sb">`source_platform`</span> and <span class="sb">`is_manual`</span> fields to <span class="sb">`Project`</span> and <span class="sb">`Blog`</span> models
<span class="p">    -</span> [ ] Add support for "metadata-only" entries
<span class="p">    -</span> [ ] Update Firestore service layer to handle these new fields
<span class="p">-</span> [ ] Task: Verify and Update Terraform for GCS
<span class="p">    -</span> [ ] Check <span class="sb">`deployment/terraform/storage.tf`</span> for the public assets bucket definition
<span class="p">    -</span> [ ] Ensure the bucket has correct public access policies (if intended for public serving)
<span class="p">    -</span> [ ] Apply Terraform changes if necessary (update <span class="sb">`tech-stack.md`</span> if new resources are added)
<span class="p">-</span> [ ] Task: Implement Google Cloud Storage (GCS) Utility
<span class="p">    -</span> [ ] Create a utility class for uploading and retrieving URLs for images in GCS
<span class="p">    -</span> [ ] Write unit tests for GCS utility with mocked storage client
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 1: Foundation &amp; Data Models' (Protocol in workflow.md)

<span class="gu">## Phase 2: Ingestion Connectors</span>
<span class="p">-</span> [ ] Task: Implement GitHub Connector
<span class="p">    -</span> [ ] Create <span class="sb">`GitHubConnector`</span> to fetch repos using <span class="sb">`PyGithub`</span> or <span class="sb">`httpx`</span>
<span class="p">    -</span> [ ] Implement mapping from GitHub API response to <span class="sb">`Project`</span> model
<span class="p">    -</span> [ ] Write unit tests with mocked API responses
<span class="p">-</span> [ ] Task: Implement Medium Connector
<span class="p">    -</span> [ ] Create <span class="sb">`MediumConnector`</span> to fetch post metadata (titles, links, summaries)
<span class="p">    -</span> [ ] Implement mapping to <span class="sb">`Blog`</span> model
<span class="p">    -</span> [ ] Write unit tests with mocked responses
<span class="p">-</span> [ ] Task: Implement Dev.to Connector
<span class="p">    -</span> [ ] Create <span class="sb">`DevToConnector`</span> to fetch posts via API
<span class="p">    -</span> [ ] Implement mapping to <span class="sb">`Blog`</span> model
<span class="p">    -</span> [ ] Write unit tests with mocked API responses
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 2: Ingestion Connectors' (Protocol in workflow.md)

<span class="gu">## Phase 3: CLI Ingestion Tool</span>
<span class="p">-</span> [ ] Task: Implement CLI Harness
<span class="p">    -</span> [ ] Create a script in <span class="sb">`app/tools/ingest.py`</span> using <span class="sb">`typer`</span> or <span class="sb">`argparse`</span>
<span class="p">    -</span> [ ] Implement the command logic to orchestrate connectors
<span class="p">    -</span> [ ] Add logic to ensure idempotency (prevent duplicates in Firestore)
<span class="p">-</span> [ ] Task: Implement YAML-based Manual Entry Support
<span class="p">    -</span> [ ] Define YAML schema for manual resource entries
<span class="p">    -</span> [ ] Add logic to the CLI to parse YAML and insert entries into Firestore
<span class="p">    -</span> [ ] Write unit tests for YAML parsing and ingestion logic
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 3: CLI Ingestion Tool' (Protocol in workflow.md)

<span class="gu">## Phase 4: Integration &amp; Documentation</span>
<span class="p">-</span> [ ] Task: End-to-End Integration Testing
<span class="p">    -</span> [ ] Create integration tests that run the full ingestion flow against a local Firestore emulator or mock
<span class="p">    -</span> [ ] Verify that images are correctly referenced and metadata is accurate
<span class="p">-</span> [ ] Task: Update Documentation
<span class="p">    -</span> [ ] Update <span class="sb">`README.md`</span> and <span class="sb">`docs/design-and-walkthrough.md`</span> with the new ingestion architecture
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 4: Integration &amp; Documentation' (Protocol in workflow.md)
</code></pre>

</div>



<p>Excellent! I then ask Conductor to go ahead and do this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>/conductor:implement
</code></pre>

</div>



<p>Conductor goes ahead with the implementation. It creates tests first, and verifies they fail. Then it writes the code. Then it re-runs the tests and confirm they pass. Then it asks for manual verification from me.</p>

<p>Now this has been implemented, I can try it out:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>uv run python <span class="nt">-m</span> app.tools.ingest <span class="nt">--help</span>
</code></pre>

</div>



<p>The output looks like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fte77ttn0xam2djza7is3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fte77ttn0xam2djza7is3.png" alt="Ingestion tool" width="800" height="274"></a></p>

<p>Let's load in my GitHub projects:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>uv run python <span class="nt">-m</span> app.tools.ingest <span class="nt">--github-user</span> derailed-dash
</code></pre>

</div>



<p>This reads in my public repos:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flduzsnk8bso00uqmkkz0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flduzsnk8bso00uqmkkz0.png" alt="Ingesting my repos" width="800" height="370"></a></p>

<p>And if we look in Firestore, we can see entries have been created:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx3jutnosvxgul76kgbuq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx3jutnosvxgul76kgbuq.png" alt="Repos in Firestore" width="800" height="465"></a></p>

<p>Later I spotted that it's ingested repos that I've forked from upstream. I don't want this; I only want it to ingest <strong>MY</strong> public repos. I fixed this later.</p>

<h4>
  
  
  Raising the PR
</h4>

<p>Having commited my changes to my dedicated branch and pushed to GitHub, I then create a PR in GitHub.</p>

<p>Let's see what Gemini makes of it in GitHub. First, we get Gemini's overview of my PR:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkzav4f73otem495e6wvc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkzav4f73otem495e6wvc.png" alt="Gemini GitHub PR review" width="800" height="566"></a></p>

<p>Okay, this is a good summary. Now let's what problems it's found:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqgf4eapiln7weqnn15q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqgf4eapiln7weqnn15q.png" alt="Gemini GItHub PR recommendations" width="800" height="340"></a></p>

<p>Ooh, this is a good catch. It's definitely conceivable that I could have blogs, repos and applications with common names. So I go ahead and fix that straight away. There were a few other issues detected, which I've fixed, committed and pushed.</p>

<p>Then we can re-run the Gemini check of the PR by adding a comment of <code>/gemini review</code> in GitHub.</p>

<h3>
  
  
  Add the UI
</h3>

<p>Here I've setup another Conductor Track, to create our React UI.</p>

<p>I've asked for a clean UI, using Vite to implement a super-fast static React UI. I've asked for carousels for my blogs, repos and sites/applications, and I've asked for a button that will trigger an overlay for the chatbot.</p>

<p>After the functionality has been implemented, we have a working UI that looks like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F74mvgsvq43oogxevb613.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F74mvgsvq43oogxevb613.png" alt="First iteration of the UI" width="800" height="560"></a></p>

<p>Not bad for so little work!</p>

<h3>
  
  
  Raise the UI PR
</h3>

<p>Let's see what the Gemini review makes of this one:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frlv03np402dsus0b3sfk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frlv03np402dsus0b3sfk.png" alt="Gemini PR review" width="800" height="472"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffcdxjo63aemvupjiqgg0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffcdxjo63aemvupjiqgg0.png" alt="Gemini PR review" width="800" height="354"></a></p>

<p>I fix a few issues that are flagged by Gemini, and we're ready to move on to the next track.</p>

<h3>
  
  
  Creating the Container Image
</h3>

<p>Here I create a new Conductor track to build the Docker image. I already had a <code>Dockerfile</code>, but I need to incorporate the new frontend. Also, I want a two phase Dockerfile in order to create a lightweight container image that starts fast. Here's the resulting <code>Dockerfile</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="c"># Stage 1: Build the React frontend</span>
<span class="k">FROM</span><span class="w"> </span><span class="s">node:20-slim</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">frontend-builder</span>

<span class="k">WORKDIR</span><span class="s"> /app/frontend</span>
<span class="k">COPY</span><span class="s"> frontend/package.json frontend/package-lock.json ./</span>
<span class="k">RUN </span>npm <span class="nb">install</span>
<span class="k">COPY</span><span class="s"> frontend/ ./</span>
<span class="k">RUN </span>npm run build

<span class="c"># Stage 2: Final production image</span>
<span class="k">FROM</span><span class="s"> python:3.12-slim</span>

<span class="k">RUN </span>pip <span class="nb">install</span> <span class="nt">--no-cache-dir</span> <span class="nv">uv</span><span class="o">==</span>0.8.13

<span class="k">WORKDIR</span><span class="s"> /code</span>

<span class="c"># Copy backend requirements and install</span>
<span class="k">COPY</span><span class="s"> ./pyproject.toml ./README.md ./uv.lock ./</span>
<span class="k">RUN </span>uv <span class="nb">sync</span> <span class="nt">--frozen</span>

<span class="c"># Copy backend code</span>
<span class="k">COPY</span><span class="s"> ./app ./app</span>

<span class="c"># Copy built frontend assets from Stage 1</span>
<span class="k">COPY</span><span class="s"> --from=frontend-builder /app/frontend/dist /code/frontend/dist</span>

<span class="k">ARG</span><span class="s"> COMMIT_SHA=""</span>
<span class="k">ENV</span><span class="s"> COMMIT_SHA=${COMMIT_SHA}</span>

<span class="k">ARG</span><span class="s"> AGENT_VERSION=0.0.0</span>
<span class="k">ENV</span><span class="s"> AGENT_VERSION=${AGENT_VERSION}</span>

<span class="k">EXPOSE</span><span class="s"> 8080</span>

<span class="k">CMD</span><span class="s"> ["uv", "run", "uvicorn", "app.fast_api_app:app", "--host", "0.0.0.0", "--port", "8080"]</span>
</code></pre>

</div>



<p>And I need to add a couple of targets to my <code>Makefile</code> so we can both build the container, and launch it.</p>

<h3>
  
  
  Raising the PR
</h3>

<p>Once again, let's see what Gemini says:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8znvfpwk3atzwdnh21vk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8znvfpwk3atzwdnh21vk.png" alt=" " width="800" height="663"></a></p>

<p>Gemini didn't find any other issues. BUT, the CI/CD failed because one of my integration tests was broken. Oops! I had forgotten to update one of the API endpoints. Easily fixed.</p>

<p>Onwards!</p>

<h3>
  
  
  Deploy to Google Cloud Run
</h3>

<p>Here I made a few updates to my Cloud Build CI/CD configuration, and my Terraform triggers. In particular, to make sure I'm passing in all the required environment variables to deploy my Cloud Run service.</p>

<p>I also updated my <code>Makefile</code> to replicate the Cloud Build workflow. So it now looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Build and deploy the agent to Cloud Run (Manual / Development)</span>
<span class="c"># Builds directly from source; not from Google Artifact Registry</span>
deploy-cloud-run:
    gcloud run deploy <span class="si">$(</span>SERVICE_NAME<span class="si">)</span> <span class="se">\</span>
        <span class="nt">--source</span> <span class="nb">.</span> <span class="se">\</span>
        <span class="nt">--memory</span> <span class="s2">"4Gi"</span> <span class="se">\</span>
        <span class="nt">--project</span> <span class="si">$(</span>GOOGLE_CLOUD_PROJECT<span class="si">)</span> <span class="se">\</span>
        <span class="nt">--region</span> <span class="si">$(</span>GOOGLE_CLOUD_REGION<span class="si">)</span> <span class="se">\</span>
        <span class="nt">--service-account</span><span class="o">=</span><span class="s2">"</span><span class="nv">$$</span><span class="s2">SERVICE_SA_EMAIL"</span> <span class="se">\</span>
        <span class="nt">--max-instances</span><span class="o">=</span>1 <span class="se">\</span>
        <span class="nt">--cpu-boost</span> <span class="se">\</span>
        <span class="nt">--allow-unauthenticated</span> <span class="se">\</span>
        <span class="nt">--set-env-vars</span><span class="o">=</span><span class="s2">"COMMIT_SHA=</span><span class="si">$(</span>shell git rev-parse HEAD<span class="si">)</span><span class="s2">,APP_NAME=</span><span class="si">$(</span>APP_NAME<span class="si">)</span><span class="s2">,AGENT_NAME=</span><span class="si">$(</span>AGENT_NAME<span class="si">)</span><span class="s2">,MODEL=</span><span class="si">$(</span>MODEL<span class="si">)</span><span class="s2">,GOOGLE_GENAI_USE_VERTEXAI=</span><span class="si">$(</span>GOOGLE_GENAI_USE_VERTEXAI<span class="si">)</span><span class="s2">,GOOGLE_CLOUD_LOCATION=</span><span class="si">$(</span>GOOGLE_CLOUD_LOCATION<span class="si">)</span><span class="s2">,LOG_LEVEL=DEBUG"</span> <span class="se">\</span>
        <span class="nt">--labels</span><span class="o">=</span>dev-tutorial<span class="o">=</span>devnewyear2026 <span class="se">\</span>
        <span class="si">$(</span><span class="k">if</span> <span class="si">$(</span>IAP<span class="si">)</span>,--iap<span class="si">)</span>
</code></pre>

</div>



<p>The service deployed without incident and I was able to run the portfolio application from the public Cloud Run URL. But the service was not showing the information stored in Firestore.</p>

<p>I immediately <em>knew</em> what the problem was: I haven't granted the Cloud Run service account the right role to use Firestore! But rather than just fixing it myself, I wanted to see how good Gemini CLI is at diagnosing the issue, given that I have the <code>gcloud</code> extension installed.</p>

<p>So, I asked Gemini CLI to diagnose my access issue. It went ahead and used the gcloud MCP server to read the Cloud Run service logs directly. It immediately detected the 403 errors in the logs, and then went ahead and added the missing <code>datastore.user</code> role to this variable in my <code>variables.tf</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"app_sa_roles"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"List of roles to assign to the application service account"</span>
  <span class="nx">type</span>    <span class="p">=</span> <span class="nx">list</span><span class="p">(</span><span class="nx">string</span><span class="p">)</span>
  <span class="nx">default</span> <span class="p">=</span> <span class="p">[</span>
    <span class="s2">"roles/aiplatform.user"</span><span class="p">,</span>
    <span class="s2">"roles/logging.logWriter"</span><span class="p">,</span>
    <span class="s2">"roles/cloudtrace.agent"</span><span class="p">,</span>
    <span class="s2">"roles/storage.objectAdmin"</span><span class="p">,</span>
    <span class="s2">"roles/serviceusage.serviceUsageConsumer"</span><span class="p">,</span>
    <span class="s2">"roles/datastore.user"</span><span class="p">,</span>
  <span class="p">]</span>
<span class="p">}</span>
</code></pre>

</div>



<p>I ran my <code>make tf-apply</code> and all was fixed. Woop!</p>

<h3>
  
  
  Implementing the Chatbot
</h3>

<p>I created a new Conductor track for this. We already have a bare-bones agent and a UI widget. What this track needs to do is:</p>

<ul>
<li>Add a secret to Secret Manager, for my chatbot's personality.</li>
<li>Inject the secret as an environment variable into Cloud Run.</li>
<li>Retrieve the secret in the agent code.</li>
<li>Add tools in order to query my portfolio data from Firestore.</li>
<li>Wire the UI widget to the agent.</li>
</ul>

<p>But I <em>didn't</em> need to tell Conductor all of this. I just told it that the personality will be stored in Google Secret Manager. Conductor then went ahead and worked out what it needed to do. Check out the plan it built:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Implementation Plan - Chatbot Implementation</span>

This plan outlines the steps to implement the "Dazbo" portfolio chatbot, including backend agent logic, infrastructure updates for secret management, and frontend integration.

<span class="gu">## Phase 1: Infrastructure &amp; Secret Management</span>
<span class="p">-</span> [ ] Task: Create Google Secret for Persona Style
<span class="p">    -</span> [ ] Create <span class="sb">`dazbo-system-prompt`</span> secret in Google Secret Manager
<span class="p">    -</span> [ ] Populate with the Dazbo persona and system prompt content
<span class="p">-</span> [ ] Task: Update Terraform Configuration
<span class="p">    -</span> [ ] Define <span class="sb">`google_secret_manager_secret`</span> in <span class="sb">`deployment/terraform/storage.tf`</span> (or dedicated file)
<span class="p">    -</span> [ ] Update <span class="sb">`google_cloud_run_v2_service`</span> in <span class="sb">`deployment/terraform/service.tf`</span> to inject the secret as an environment variable named <span class="sb">`DAZBO_SYSTEM_PROMPT`</span>
<span class="p">    -</span> [ ] Update <span class="sb">`app_sa_roles`</span> in <span class="sb">`deployment/terraform/variables.tf`</span> to include <span class="sb">`roles/secretmanager.secretAccessor`</span>
<span class="p">-</span> [ ] Task: Apply Infrastructure Changes
<span class="p">    -</span> [ ] Run <span class="sb">`make tf-apply`</span> to provision resources and update Cloud Run
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 1: Infrastructure' (Protocol in workflow.md)

<span class="gu">## Phase 2: Agent Tooling &amp; Logic [checkpoint: 47cbdae]</span>
<span class="p">-</span> [ ] Task: Implement Portfolio Search Tool [2e50bd9]
<span class="p">    -</span> [ ] Write unit tests for <span class="sb">`search_portfolio`</span> tool
<span class="p">    -</span> [ ] Implement <span class="sb">`search_portfolio`</span> in <span class="sb">`app/agent.py`</span> or a new tools module
<span class="p">    -</span> [ ] Tool should query Firestore <span class="sb">`projects`</span> and <span class="sb">`blogs`</span> collections based on query/tags
<span class="p">-</span> [ ] Task: Implement Content Detail Tool [b8078da]
<span class="p">    -</span> [ ] Write unit tests for <span class="sb">`get_content_details`</span> tool
<span class="p">    -</span> [ ] Implement <span class="sb">`get_content_details`</span> to fetch a full document from Firestore by ID
<span class="p">-</span> [ ] Task: Refine Agent Persona &amp; System Prompt Handling [69691f7]
<span class="p">    -</span> [ ] Update <span class="sb">`app/config.py`</span> to include the environment variable name
<span class="p">    -</span> [ ] Modify <span class="sb">`app/agent.py`</span> to read the system prompt from the <span class="sb">`DAZBO_SYSTEM_PROMPT`</span> environment variable at runtime
<span class="p">    -</span> [ ] Ensure <span class="sb">`InMemorySessionService`</span> is correctly integrated for history persistence
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 2: Agent Logic' (Protocol in workflow.md)

<span class="gu">## Phase 3: Backend API &amp; Streaming</span>
<span class="p">-</span> [ ] Task: Implement Streaming Endpoint in FastAPI
<span class="p">    -</span> [ ] Write integration tests for streaming chat endpoint
<span class="p">    -</span> [ ] Update <span class="sb">`app/fast_api_app.py`</span> to include an SSE endpoint for the agent
<span class="p">    -</span> [ ] Ensure the agent's stream is correctly piped to the SSE response
<span class="p">-</span> [ ] Task: Verify Backend End-to-End
<span class="p">    -</span> [ ] Run <span class="sb">`make local-backend`</span>
<span class="p">    -</span> [ ] Use <span class="sb">`curl`</span> to verify the streaming response from the API
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 3: Backend Streaming' (Protocol in workflow.md)

<span class="gu">## Phase 4: Frontend Integration</span>
<span class="p">-</span> [ ] Task: Connect ChatWidget to Backend
<span class="p">    -</span> [ ] Implement SSE listener in <span class="sb">`frontend/src/components/ChatWidget.tsx`</span>
<span class="p">    -</span> [ ] Update UI state to handle streaming chunks and display message history
<span class="p">    -</span> [ ] Add "typing" indicator and auto-scroll to bottom
<span class="p">-</span> [ ] Task: Verify UI/UX
<span class="p">    -</span> [ ] Run <span class="sb">`make react-ui`</span> and <span class="sb">`make local-backend`</span>
<span class="p">    -</span> [ ] Confirm chat feels responsive and correctly reflects the Dazbo persona
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 4: Frontend Integration' (Protocol in workflow.md)

<span class="gu">## Phase 5: Documentation &amp; Roadmap</span>
<span class="p">-</span> [ ] Task: Update Roadmap &amp; Design Docs
<span class="p">    -</span> [ ] Append future RAG/Vector Search tasks to <span class="sb">`TODO.md`</span>
<span class="p">    -</span> [ ] Update <span class="sb">`docs/design-and-walkthrough.md`</span> with the RAG roadmap details (Vertex AI + Firestore Vector Search)
<span class="p">-</span> [ ] Task: Conductor - User Manual Verification 'Phase 5: Documentation' (Protocol in workflow.md)
</code></pre>

</div>



<p>Here's a snippet that shows our agent now has access to these new portfolio tools:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">root_agent</span> <span class="o">=</span> <span class="nc">Agent</span><span class="p">(</span>
    <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">root_agent</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">You are Dazbo</span><span class="sh">'</span><span class="s">s helpful assistant. You can search for content in his portfolio</span><span class="sh">"</span><span class="p">,</span> 
    <span class="n">model</span><span class="o">=</span><span class="nc">Gemini</span><span class="p">(</span>
        <span class="n">model</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="n">model</span><span class="p">,</span>
        <span class="n">retry_options</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">HttpRetryOptions</span><span class="p">(</span><span class="n">attempts</span><span class="o">=</span><span class="mi">3</span><span class="p">),</span>
    <span class="p">),</span>
    <span class="n">instruction</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="n">dazbo_system_prompt</span><span class="p">,</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[</span><span class="n">search_portfolio</span><span class="p">,</span> <span class="n">get_content_details</span><span class="p">],</span>
<span class="p">)</span>
</code></pre>

</div>



<p>We can test the chatbot using the "ADK Web" UI:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdot95309dytvzoqi89j1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdot95309dytvzoqi89j1.png" alt=" " width="800" height="428"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqgxcggktfhnhhiiz02qo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqgxcggktfhnhhiiz02qo.png" alt=" " width="800" height="648"></a></p>

<p>We can see it's using the right tools and retrieving the right content. Nice!</p>

<h3>
  
  
  Adding the Chatbot to the UI
</h3>

<p>After making all the required changes and pushing, here's the PR summary from Gemini:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4e3fyrp0e98ft8skuubq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4e3fyrp0e98ft8skuubq.png" alt=" " width="800" height="583"></a></p>

<p>And it raises this concern about prompt injection attacks:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9j4o4vag4im9zmp5i3dp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9j4o4vag4im9zmp5i3dp.png" alt=" " width="800" height="301"></a></p>

<p>Ooh, that's a good one. I added some defensive code (with Gemini's help, of course) and then created a prompt injection test.</p>

<p>Let's have a look at the Chatbot in action:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdd284kqr0wtk3a4zq332.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdd284kqr0wtk3a4zq332.png" alt=" " width="395" height="603"></a></p>

<h3>
  
  
  Implement Rate Limiting
</h3>

<p>I wanted API rate limiting for the usual reasons. For example, to prevent abuse, and to avoid unexpected costs. I don't want a denial-of-service attack to cost me loads of money!</p>

<p>I ask Conductor to implement the rate limiting. It offers a few ways to do this, but I've gone with the <code>slowapi</code> library. I included global rate limiting for the API, and more stringent rate limit for any calls that interact with Gemini.</p>

<h3>
  
  
  Working Around the Medium RSS Issue
</h3>

<p>It turns out the Medium RSS feed only returns the 10 latest blogs. This is no good! I want my portfolio to show <em>all</em> my blogs. Medium doesn't offer an API to retrieve blogs, so I'm going to have to go old-school and scrape.</p>

<p>Additionally, I would like to:</p>

<ol>
<li>Use Gemini to create summaries of each blog read. I will store the summary in Firestore.</li>
<li>Suggest tags for each ingested blog.</li>
<li>Convert HTML pages into markdown, with some specific layout rules. (I will re-use this code elsewhere, to help me with cross-posting to blogging sites.)</li>
</ol>

<p>I created a Conductor track to do this, and guided it in creating a plan and spec, as usual. One really cool thing that Conductor tells me is that Medium offers the ability to export all my blogs in a single zip. I didn't know this! This zip contains all my posts in HTML format. Well, it's not an API, but it's very useful.</p>

<p>Here's the <code>spec.md</code> from Conductor:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Specification: Comprehensive Medium Blog Ingestion</span>

<span class="gu">## Overview</span>
The goal of this track is to overcome the 10-post limitation of the Medium RSS feed and enhance the portfolio's content richness. This involves a hybrid ingestion system (RSS + Zip Export), paywall detection, and a new processing pipeline that converts HTML content to structured Markdown and generates AI-powered summaries.

<span class="gu">## Functional Requirements</span>
<span class="p">-</span> <span class="gs">**Hybrid Ingestion Engine:**</span>
<span class="p">    -</span> Support fetching the latest 10 posts via the standard Medium RSS feed.
<span class="p">    -</span> Implement a parser for Medium export archives (<span class="sb">`posts.zip`</span>).
<span class="p">    -</span> The archive parser must extract blog metadata (title, date, content/summary, URL) from HTML files located within the <span class="sb">`posts/`</span> directory of the zip.
<span class="p">-</span> <span class="gs">**Content Processing &amp; AI Enrichment:**</span>
<span class="p">    -</span> <span class="gs">**HTML to Markdown Conversion:**</span>
<span class="p">        -</span> Convert the blog post HTML content into clean Markdown.
<span class="p">        -</span> <span class="gs">**Formatting Rules:**</span>
<span class="p">            -</span> Title: H1 (<span class="sb">`#`</span>)
<span class="p">            -</span> Headings: H2 (<span class="sb">`##`</span>)
<span class="p">            -</span> Subheadings: H3 (<span class="sb">`###`</span>)
<span class="p">        -</span> <span class="gs">**Frontmatter:**</span> Include YAML frontmatter with <span class="sb">`subtitle`</span> and <span class="sb">`tags`</span> (if available).
<span class="p">    -</span> <span class="gs">**AI Summarization:**</span>
<span class="p">        -</span> Generate a concise summary of the entire blog post using the project's Gemini agent.
<span class="p">    -</span> <span class="gs">**Storage:**</span> Store the generated Markdown and AI Summary in the <span class="sb">`Blog`</span> model in Firestore.
<span class="p">-</span> <span class="gs">**Paywall Identification:**</span>
<span class="p">    -</span> Implement heuristic analysis to detect paywalled content (e.g., "Member-only story" markers).
<span class="p">    -</span> Update the <span class="sb">`Blog`</span> model to include a <span class="sb">`is_private`</span> boolean field.
<span class="p">-</span> <span class="gs">**Duplicate Management &amp; Idempotency:**</span>
<span class="p">    -</span> Detect duplicates across RSS and Zip sources using the canonical URL (fallback to Title).
<span class="p">    -</span> <span class="gs">**Priority:**</span> RSS Feed metadata takes precedence for basic fields (date, title), but the Zip export (processed into Markdown) serves as the source for the full content body.
<span class="p">-</span> <span class="gs">**CLI Enhancement:**</span>
<span class="p">    -</span> Update the <span class="sb">`ingest`</span> CLI tool to accept a <span class="sb">`--medium-zip`</span> parameter.
<span class="p">-</span> <span class="gs">**UI Presentation:**</span>
<span class="p">    -</span> Display the <span class="gs">**AI-generated summary**</span> in the portfolio interface.
<span class="p">    -</span> Provide a clear link to the original Medium post for full reading.
<span class="p">    -</span> Display a "Member-only" badge for paywalled content.
<span class="p">-</span> <span class="gs">**Documentation:**</span>
<span class="p">    -</span> Update <span class="sb">`docs/design-and-walkthrough.md`</span> to reflect the new hybrid ingestion architecture, design decisions, and a detailed walkthrough of the mechanism.

<span class="gu">## Non-Functional Requirements</span>
<span class="p">-</span> <span class="gs">**Performance:**</span> Zip parsing and AI summarization should handle rate limits gracefully.
<span class="p">-</span> <span class="gs">**Maintainability:**</span> Modular parser architecture.

<span class="gu">## Acceptance Criteria</span>
<span class="p">-</span> [ ] Ingestion with <span class="sb">`posts.zip`</span> populates Firestore with historical posts.
<span class="p">-</span> [ ] Blog content is stored as Markdown in Firestore (for future use/RAG).
<span class="p">-</span> [ ] Each blog entry has an AI-generated summary stored in Firestore.
<span class="p">-</span> [ ] The Portfolio UI displays the AI Summary and links to the full post.
<span class="p">-</span> [ ] Member-only stories are correctly flagged and badged in the UI.
<span class="p">-</span> [ ] <span class="sb">`docs/design-and-walkthrough.md`</span> is updated with the new ingestion details.
</code></pre>

</div>



<p>I spotted a number of issues with the initial implementation. For example:</p>

<ul>
<li>It takes a lot of time to process a zip with many blog files, so we need some sort of progress indicator.</li>
<li>The Medium blog export includes comments and replies to comments. We don't want to import these as blogs, so the application needs to ignore these.</li>
<li>We also need to skip files that are unpublished drafts.</li>
</ul>

<p>This ends up being a pretty big PR in the end:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frgsz0zgtk2t9zji81imw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frgsz0zgtk2t9zji81imw.png" alt=" " width="800" height="618"></a></p>

<p>After we run the new ingestion tool, we can see Firestore has been properly updated with over 100 blogs from Medium, including AI-generated summaries and AI-extracted tags:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkfu0w14tz5wna523ie8r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkfu0w14tz5wna523ie8r.png" alt=" " width="800" height="500"></a></p>

<h3>
  
  
  Domain Name Mapping
</h3>

<p>My goal is to map my Cloud Run URL to my custom domain.</p>

<p>But at this point in the day, I've run out of Gemini quota! Sad times. So I'm doing the next few steps without any AI assistance.</p>

<p>First, I need to verify my domain name. I'm following the <a href="https://support.google.com/webmasters/answer/9008080#txt_instructions&amp;zippy=%2Chtml-file-upload%2Cdomain-name-provider%2Cmanual-domain-name-provider-instructions" rel="noopener noreferrer">guidance here</a> in order to create a <code>TXT</code> record that I can configure with my DNS registrar.</p>

<p>Next, I add this block to my <code>services.tf</code> Terraform:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Create domain mappings for all listed domains</span>
<span class="nx">resource</span> <span class="s2">"google_cloud_run_domain_mapping"</span> <span class="s2">"app_prod_domain_mapping"</span> <span class="p">{</span>
  <span class="nx">for_each</span> <span class="p">=</span> <span class="nx">toset</span><span class="p">(</span><span class="nx">var</span><span class="p">.</span><span class="nx">app_domain_name</span><span class="p">)</span>
  <span class="nx">name</span>     <span class="p">=</span> <span class="nx">each</span><span class="p">.</span><span class="nx">key</span>
  <span class="nx">project</span>  <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">project_id</span>
  <span class="nx">location</span> <span class="p">=</span> <span class="nx">google_cloud_run_v2_service</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">location</span>
  <span class="nx">metadata</span> <span class="p">{</span>
    <span class="nx">namespace</span> <span class="p">=</span> <span class="nx">data</span><span class="p">.</span><span class="nx">google_project</span><span class="p">.</span><span class="nx">project</span><span class="p">.</span><span class="nx">project_id</span>
  <span class="p">}</span>
  <span class="nx">spec</span> <span class="p">{</span>
    <span class="nx">route_name</span> <span class="p">=</span> <span class="nx">google_cloud_run_v2_service</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">name</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>I add the matching variable in <code>variables.tf</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"app_domain_name"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"A list of domain names to be mapped to the service"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">list</span><span class="p">(</span><span class="nx">string</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>And then I add the comma-separated list of domains to my <code>env.tfvars</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">app_domain_name</span>  <span class="err">=</span> <span class="p">[</span><span class="s2">"darrenlester.net"</span><span class="p">,</span> <span class="s2">"www.darrenlester.net"</span><span class="p">]</span>
</code></pre>

</div>



<p>Now I apply this Terraform configuration and the domain mappings are created. We can verify that this has applied in the Google Cloud Console, by looking at <code>Domain Mappings</code> in the Console:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnc68s3l4c7daikm9rpks.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnc68s3l4c7daikm9rpks.png" alt=" " width="800" height="226"></a></p>

<p>I need to grab the <code>A</code> (IP4) and <code>AAAA</code> (IP6) records for the domain and add them to the DNS configuration of my DNS registar. And I need to grab the <code>CNAME</code> for the subdomain (i.e. <code>www</code>) and add this to the DNS registrar also.</p>

<p>Shortly after doing this, Google provisions SSL certificates for the domains. It took about 30 minutes.</p>

<h3>
  
  
  UI Aesthetics
</h3>

<p>The UI is okay, but it's not great. I need to add some style and polish!</p>

<p>I started by creating a banner image using Nano Banana Pro. I gave it a few images I wanted to use as subjects. I provided by profile pic, obviously! </p>

<p>Then I told the agent in Antigravity to help me experiment with the UI, and with these requirements:</p>

<ul>
<li>Add the new banner</li>
<li>Add my profile pic to the left of the banner</li>
<li>Use black background with dark theme</li>
</ul>

<p>It made some excellent initial UI changes. There were a few aesthetic issues to work through, but eventually I ended up with a result that I was happy with:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb1fsc4gvbrq2h6q9kl1h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb1fsc4gvbrq2h6q9kl1h.png" alt=" " width="800" height="577"></a></p>

<p>(I've iterated the UI a couple of times since then.)</p>

<h2>
  
  
  Okay, You Get The Idea
</h2>

<p>I could show some more hands-on stuff from this journey. But I'm knackered, and I'm sure you are too. But I hope you've found these examples interesting and useful!</p>

<h2>
  
  
  Before You Go
</h2>

<p>Please engage with this post - even if it's just a like. And please follow me so you don't miss my future content. Thanks!</p>

