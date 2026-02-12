---
Title: After 20 Years Managing Dev Teams, I Built a Full SaaS Alone With AI
Description: 
Author: Jacobo Moreno
Date: 2026-02-12T22:04:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>In September 2025, I sat down at a desk in Vancouver, opened a terminal, and started building ioZen. No dev team. No co-founder writing code beside me. No Slack channels with dozens of engineers debating architecture decisions.</p>

<p>Just me, a clear vision, and AI.</p>

<p>Four months later, we're launching the beta of a platform that replaces bureaucratic forms with intelligent conversations. It has AI-powered intake flows, visual workflow boards, a CRM, marketing attribution, embeddable widgets, and a document extraction engine.</p>

<p>A well-funded startup with a full engineering team would have taken 12 to 18 months to ship this scope.</p>

<p>I'm still processing what happened. I want to tell you the real story.</p>

<h2>
  
  
  The road to Vancouver
</h2>

<p>I need to go back to explain how I got here.</p>

<p>For over 20 years, I built technology companies in Latin America. I founded PATIOTuerca (Ecuador's largest automotive marketplace) and co-founded Vive1 (real estate), Evaluar.com (leading HR tech serving millions of users), and Taxo (tax technology for Mexico and Ecuador). I led engineering organizations across Mexico, Argentina, Peru, Ecuador, and beyond. Across all of these ventures, I managed teams of dozens of developers building complex, high-stakes systems.</p>

<p>I know what it takes to ship software at scale. I've lived through waterfall-to-agile transitions, microservices migrations, monolith rewrites, cloud infrastructure debates. Every flavor of technical debt you can imagine. I've recruited senior devs, built hiring pipelines, run sprint ceremonies, negotiated with vendors. I've done architecture reviews at 2 AM before critical launches.</p>

<p>That world was my entire identity for two decades.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fwte1v5pd70yxdqtfyt.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fwte1v5pd70yxdqtfyt.jpeg" alt="Speaking at BBVA Open Innovation. The world I knew for 20 years." width="800" height="534"></a></p>

<blockquote>
<p>Speaking at BBVA Open Innovation. The world I knew for 20 years.</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhvmp5pmxn6d6ths532km.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhvmp5pmxn6d6ths532km.jpeg" alt=" " width="800" height="600"></a></p>

<blockquote>
<p>Presenting at AWS Cloud Experience. Cloud architecture experience.</p>
</blockquote>

<p>Then, in early 2025, I made a decision that changed everything.</p>

<p>I stepped away from Taxo. My wife and I moved to Vancouver, Canada. It was a new beginning in every sense: new country, new language (English isn't my native tongue), new environment, new financial reality. I left behind the comfort zone of my network in Ecuador, the teams I had built, the companies that defined me.</p>

<p>I arrived in Vancouver with a clear mind and an empty calendar.</p>

<h2>
  
  
  The uncomfortable truth about starting over
</h2>

<p>I won't deny it. Those first months in Vancouver were hard.</p>

<p>I had income from advisory roles and board seats, but nothing like what I earned before. I was in a city where nobody knew my track record. I didn't have the grounding of my professional network, the daily rhythm of managing teams, the identity that comes from being "the hardcore tech nerd" in the room.</p>

<p>What I did have was time. And an idea that had been growing in my head for years.</p>

<p>Throughout my career, I watched the same pattern destroy value everywhere: bureaucracy. Every organization I touched was drowning in forms, approvals, paperwork, and process overhead. The tools designed to fix this (form builders, workflow engines, CRM systems) create their own bureaucracy. They make the problem prettier, not smaller.</p>

<p>I kept thinking: what if we could kill the forms entirely? What if instead of building another form, you could have a conversation that gets the right information, then routes it, processes it, and delivers an outcome automatically?</p>

<p>That was the seed idea for ioZen.</p>

<h2>
  
  
  The experiment
</h2>

<p>Here I want to be brutally honest. I didn't plan to build ioZen alone.</p>

<p>The original plan was to find a technical co-founder. Maybe hire a small team. Raise a pre-seed round. Do it the way I'd always done it: architect the system, recruit the talent, manage the execution.</p>

<p>But something happened while that plan was materializing. I started prototyping with AI coding tools. Not because I believed I could build the whole thing, but because I'm an engineer at heart, and I wanted to validate my ideas fast.</p>

<p>I've been coding since I was a teenager. In 1996, at 16, I sold my first website. Over the years, as I moved into leadership, I coded less and managed more. But the mental models and the logic of building software never left, they sharpened. System design, data architecture, API patterns, cloud infrastructure, security considerations, code patterns. All of that was still there, accumulating over 20 years of shipping real products, but far from the syntax and the thrill of coding.</p>

<p>So I started. Cursor open. Claude in another window. A blank Git repository.</p>

<p>And something unexpected happened.</p>

<p>The AI didn't just autocomplete my code. It became a real right hand. I could describe an architectural pattern I wanted, say a queue-based workflow engine with state transitions, and the AI understood what I meant. Not because it had opinions about architecture. But because I had opinions, and it could execute on them at a speed no junior or mid-level developer could match. It even surprised me by making brilliant decisions or challenging assumptions in ways I had only experienced with my best engineers.</p>

<p>This wasn't about AI replacing developers. It was about AI <em>amplifying</em> someone who already knew what to build and how to build it.</p>

<h2>
  
  
  What experience actually means in the age of AI
</h2>

<p>This is the part nobody talks about when they say "AI will replace developers."</p>

<p>AI is incredible at generating code. It's fast, tireless, and gets better every month. But code is not the same as software. Software is decisions. Architecture decisions. Data model decisions. Security decisions. UX decisions. Hard trade-off decisions that come from building things that broke, from scaling systems that buckled, from shipping products that users rejected or never used.</p>

<p>When I sat down to build ioZen, I wasn't starting from zero. I was starting from twenty years of hardships, failed products, and successful releases accumulated.</p>

<p>I knew why I wanted a particular database schema because I had lived through what a bad database design does at scale. I knew which authentication patterns to use because I'd had to deal with the mess of getting them wrong. I knew how to structure a monorepo, how to design an API that could evolve gracefully, how to implement role-based access control that wouldn't become a maintenance nightmare.</p>

<p>The AI gave me speed. My experience gave the AI direction.</p>

<p>Without the experience, the AI would have generated functional but fragile code. Without the AI, my experience would have produced maybe the same code in 10 or 100 times the time, and I would have needed a whole team.</p>

<p>Together? Something new happened.</p>

<h2>
  
  
  Four months, one person
</h2>

<p>Let me be more specific about what we built between September 2025 and January 2026:</p>

<ul>
<li><p><strong>IntakeBots</strong>: conversational AI agents that replace forms. They ask follow-up questions, handle branching logic, extract data from documents, and adapt based on previous answers. Forms that think.</p></li>
<li><p><strong>Process Boards</strong>: visual workflow management inspired by Kanban, purpose-built for intake-driven processes. Stages, assignments, automations, status tracking. All connected to the intake data.</p></li>
<li><p><strong>Contact Records</strong> and lightweight CRM that auto-populates from intake conversations. Every interaction, document, and status change is captured and searchable.</p></li>
<li><p><strong>AI Field Intelligence</strong>: AI to classify, extract, summarize, and validate incoming data. Not bolted-on AI. Native intelligence in data fields.</p></li>
<li><p><strong>Embeddable Widgets</strong> so you can deploy any IntakeBot as a chatbot widget on any website or as a direct link. Custom branding, zero-friction integration.</p></li>
<li><p><strong>Marketing Attribution</strong> to track which campaigns, sources, and referrers drive conversions, with built-in UTM tracking.</p></li>
<li><p><strong>Multi-workspace Architecture</strong> with full multi-tenancy, team management, role-based access, and workspace isolation.</p></li>
<li><p><strong>Developer Tools</strong>: webhooks, embed codes, API access, and complete documentation.</p></li>
<li><p><strong>Marketing Website</strong>: high SEO performance, refined design, modern animations. Also built with AI.</p></li>
</ul>

<p>One person. Four months. A complete product-led SaaS platform with Stripe integration and a marketing website. And 100% functional in English and Spanish.</p>

<p>I'm not listing this to boast. If you told me two years ago that a single founder could ship this scope in this timeline, I would not have believed you. And I was the person who was supposed to know what's possible.</p>

<h2>
  
  
  What AI development actually looks like
</h2>

<p>People hear "I built it with AI" and imagine someone typing prompts into ChatGPT and pasting code into an IDE. It's more than that.</p>

<p>AI engineering is a new discipline. It has its own patterns, its own problems, its own art.</p>

<p>Here's what my days actually looked like:</p>

<p><strong>Architecture sessions.</strong> I'd spend time thinking about system design: on paper, in my head, walking around Vancouver. The AI didn't help here. This was pure experience and neurons. What should the data model look like? Where do we need real-time updates vs. batch processing? How do we handle concurrent workflow state changes?</p>

<p><strong>Implementation sprints.</strong> Once I had a clear architectural vision, the AI became my execution engine. I could describe what I needed (a specific component, an API endpoint, a database migration) and iterate rapidly. What would take a developer a week, I could review, refine, and ship in hours.</p>

<p><strong>Code review of my own AI-generated code.</strong> This is critical and underrated. AI generates code that works. It doesn't always generate code that's <em>right</em>. I spent significant time reading, questioning, and restructuring what the AI produced. My experience has been most valuable here: recognizing patterns that would cause problems at scale, catching security oversights, ensuring consistency across the codebase, validating test coverage (this is key), automatic hooks before commits, ensuring good CI/CD with quality gates.</p>

<p><strong>Debugging sessions.</strong> AI is great at writing code. It's less great at understanding why something doesn't work in a specific context. Debugging complex interactions between services, tracing state management bugs, understanding why a particular edge case behaves unexpectedly, iterating until it's right. That required human judgment.</p>

<p><strong>Whole days of refactoring.</strong> Something I learned over the years: it's unlikely you'll build a quality product without several refactors along the way. No matter how well-conceived your architecture or how good your code. Technical debt is the biggest poison to software quality. For the first time in my life, I gave myself the luxury of doing deep refactors before going to production. I must have done a dozen deep refactors of what I built over these months. Unthinkable in my past life, because doing a refactor at this level in production can take weeks or months.</p>

<p>The ratio was roughly: 30% thinking, 20% directing the AI, 25% reviewing and testing, 15% debugging, 10% manual coding for the tricky parts. And some days, 100% refactoring with the same AI and test coverage.</p>

<p>It's not just dictating prompts and done. It's a new way of building that requires a different kind of discipline and determination.</p>

<h2>
  
  
  The tech stack (for those curious)
</h2>

<ul>
<li>
<strong>Frontend</strong>: Next.js, React, TypeScript, Tailwind CSS</li>
<li>
<strong>Backend</strong>: Node.js, PostgreSQL, Supabase</li>
<li>
<strong>AI</strong>: Claude (Anthropic) via Cursor IDE</li>
<li>
<strong>Hosting</strong>: Vercel (frontend), Supabase (backend + auth + realtime)</li>
<li>
<strong>Payments</strong>: Stripe</li>
<li>
<strong>Marketing site</strong>: Astro, also built with AI</li>
</ul>

<h2>
  
  
  Why this matters beyond my story
</h2>

<p>I'm not writing this as a humble-brag about what one founder can do. I'm writing it because I think we're at a turning point that most people haven't fully processed.</p>

<p>For decades, the limiting factor for software startups was engineering talent. You needed a team. You needed funding to pay that team. You needed months or years of runway to build something meaningful. This created a natural barrier to entry that favored well-funded companies in major tech hubs.</p>

<p>That barrier is collapsing.</p>

<p>A single person with years of building experience and AI fluency can now build what used to require a team of 10 or 20. Not a toy project. Not an MVP held together with tape. A real, production-grade platform.</p>

<p>This doesn't mean developers are going away. Far from it. As ioZen grows, I will build a talented team. There are challenges that require multiple minds, diverse perspectives, and specialized expertise. AI amplifies capability. It doesn't replace the need for humans to collaborate, debate, and create together.</p>

<p>But the starting conditions have changed forever. The solo founder with experience and vision is now a viable technical team. The barrier to entry for building ambitious software has dropped by an order of magnitude.</p>

<p>If you're an experienced professional who has been on the sidelines of the AI revolution, thinking it's for "real developers" or "younger people," you're wrong. Your experience is the most valuable input an AI can receive. The architecture patterns you learned the hard way, the production failures you survived, the systems you built and maintained for years. That knowledge, combined with AI, is powerful.</p>

<h2>
  
  
  What's next
</h2>

<p>ioZen is live. The platform is real. The mission is real.</p>

<p>We're going after bureaucracy. Starting with the forms that waste everyone's time and ending with the processes that keep organizations stuck.</p>

<p>If you've ever filled out a 47-field form and wondered "why does this exist," you understand the problem. If you've ever built a workflow that created more overhead than it eliminated, you understand it deeply.</p>

<p>Four months ago, this was an idea in a Vancouver apartment. Now it's a product. <a href="https://iozen.ai" rel="noopener noreferrer">Check it out at iozen.ai</a> if you're curious.</p>




<p><em>What do you think? I'd love honest feedback from the dev community. About the product, the approach, the tech stack, anything. And if you're building with AI too, I want to hear your story. What's working? What's not? Drop a comment or find me on <a href="https://linkedin.com/in/jacobomn" rel="noopener noreferrer">LinkedIn</a> or <a href="https://x.com/jacobomoreno" rel="noopener noreferrer">X</a>.</em></p>

<p><em>AI tools (Claude, Cursor) were used in building both the product described and in editing this article.</em></p>

