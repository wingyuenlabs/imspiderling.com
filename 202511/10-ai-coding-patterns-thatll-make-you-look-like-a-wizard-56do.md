---
Title: 10 AI Coding Patterns That'll Make You Look Like a Wizard 🧙‍♂️
Description: 
Author: Daniel Shashko
Date: 2025-11-04T21:58:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most developers or technical marketers I see are either copying garbage code without thinking, or avoiding AI entirely because "it doesn't understand my problem 🤥". Both approaches miss the point. AI won't write your app for you, but it can handle the parts of coding that make you want to quit and cry.</p>

<p>The ten techniques below work because they're based on how you actually build software and are tested by me and many others I learned from. </p>

<h2>
  
  
  Make AI Explain Your Code Back to You
</h2>

<p>This one changed everything for me. Instead of asking AI to fix broken code, I started explaining my code to it first and asking it to tell me what it does.</p>

<p>It's <a href="https://rubberduckdebugging.com/" rel="noopener noreferrer">rubber duck debugging</a>, but the duck actually talks back 🤪</p>

<p>When you hit a bug, write out what you're trying to accomplish and what your code actually does. Then ask AI to explain your logic back to you in plain language. Nine times out of ten, when the AI restates your code, you'll spot the flaw immediately.</p>

<p><strong>The Reverse Explanation Pattern:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>I'm trying to calculate the total price with tax. Here's my function:

[paste your code]

Can you explain back to me, in plain English, what this function does step by step? Don't fix anything, just tell me what the code is actually doing.
</code></pre>

</div>



<p>When the AI says "this function multiplies price by 0.08 then adds it to itself," you'll realize you forgot to add the original price. The bug becomes obvious because someone (or something) else is reading your intent.</p>

<p>I use this for almost every bug now. It catches logic errors way faster than staring at the screen for an hour.</p>

<p><strong>⏱️ Time Saved:</strong> 30 minutes to 2 hours per bug</p>

<h2>
  
  
  Generate Database Schemas That Actually Work
</h2>

<p>Setting up databases is one of those things that's never creative but always takes forever (at least for me). You know what you need: tables, relationships, indexes, etc. It's just annoying to write it all out.</p>

<p>AI can generate complete database schemas if you give it the full picture upfront. Don't just say "make me a user table." Describe your entire data model, how pieces relate, what needs to be fast, and what needs to be secure.</p>

<p><strong>Schema Generation Checklist:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>📊 Entities: List every type of thing (users, posts, comments, etc.)
🔗 Relationships: One-to-many? Many-to-many? Be explicit
🔒 Constraints: Required fields, unique values, foreign keys
⚡ Performance: What queries will run most often?
🗂️ Indexing: Which columns need indexes for speed?
📝 Audit: Do you need created_at, updated_at, deleted_at?
</code></pre>

</div>



<p>For a blog platform side project, I asked for tables for users, posts, comments, and tags. Told it users write posts, posts have comments, and posts can have multiple tags. Needed it optimized for showing recent posts by author since that's the main query. Asked for <a href="https://www.postgresql.org/" rel="noopener noreferrer">PostgreSQL</a> with proper relationships and timestamps on everything.</p>

<p>I got back complete table definitions with all the connections set up correctly. Ran it, worked immediately. Saved hours of looking up syntax (especially for a newbie like me).</p>

<p><strong>⏱️ Time Saved:</strong> 2 to 4 hours</p>

<h2>
  
  
  Ask AI to Break Down Your Approach First
</h2>

<p>This is the opposite of what most people do. Instead of asking AI to solve your problem, ask it to help you plan your solution.</p>

<p>Tell AI what you're building and ask it to question your approach. What edge cases might you be missing? What's a simpler way to do this? Where might this break at scale?</p>

<p><strong>The Interrogation Pattern:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>I'm building [feature description]. My plan is to [your approach].

Before I start coding or working on this project, what questions should I be asking myself? What edge cases am I probably not thinking about? Is there a simpler architecture I'm overlooking?
</code></pre>

</div>



<p>I was planning to build a simple feature for handling file uploads on a personal project (the blog platform from before). Asked AI what could go wrong with my approach. It caught that I wasn't checking file size limits or validating file types. Would have been a security nightmare in production.</p>

<p>Use this for any feature that feels like it might get complicated or if you always "forget" something. The AI won't always be right, but it'll force you to defend your choices, which often reveals flaws you didn't see.</p>

<p><strong>⏱️ Time Saved:</strong> 4 to 8 hours (by avoiding wrong implementations)</p>

<h2>
  
  
  Generate Migration Scripts Without the Headache
</h2>

<p>Database migrations are error-prone and boring. One typo in a migration and you're rolling back in production at 2am.</p>

<p>AI is perfect for generating migration scripts because it can be paranoid about edge cases in ways humans forget to be.</p>

<p>Give it your current schema and your target schema, and ask for migration scripts that handle existing data, add rollback commands, and preserve data integrity.</p>

<p><strong>Migration Request Template:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Current schema: [paste current tables]
Target schema: [describe changes]

Generate migration scripts that:
- Add new columns with appropriate defaults
- Migrate existing data safely
- Create rollback scripts
- Add necessary indexes
- Handle foreign key constraints
- Work with [your DB engine]

Include warnings for any data loss scenarios.
</code></pre>

</div>



<p>Needed to split a <code>users</code> table into <code>users</code> and <code>user_profiles</code> to separate auth from profile data. Described the split and asked for both up and down migrations. AI generated scripts that created the new table, copied data over, set up foreign keys, added indexes, and included a rollback that would merge everything back. Even included warnings about potential data loss if email fields were longer than 255 characters. For best practices, see <a href="https://www.harness.io/harness-devops-academy/database-rollback-strategies-in-devops" rel="noopener noreferrer">Database Rollback Strategies in DevOps</a>.</p>

<p>Tested the migration on a copy of production data. Worked perfectly. Would have taken me all afternoon to write and test manually.</p>

<p><strong>⏱️ Time Saved:</strong> 1 to 3 hours</p>

<h2>
  
  
  Build Multi-Step Workflows in One Shot
</h2>

<p>Once you understand individual prompting patterns, you can chain them together to generate entire features.</p>

<p>The trick is structuring your prompt as a sequence: models first, then business logic, then API, then UI, then tests. Each piece builds on the previous one.</p>

<p><strong>Feature Blueprint Format:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Build a [feature name] with the following stack: [technologies]

Step 1 - Data Model:
[describe your entities and relationships]

Step 2 - Business Logic:
[describe the core functionality]

Step 3 - API Endpoints:
[list endpoints with methods and requirements]

Step 4 - Frontend:
[describe the UI components needed]

Step 5 - Tests:
[what needs test coverage]

Generate code for each step that works together as a cohesive feature.
</code></pre>

</div>



<p>I needed a content translation system. Described the setup: Streamlit for the UI, Playwright to scrape web pages, AI models for translation, Bright Data's SERP API to find relevant links to add, and Pyppeteer for simple browser actions like screenshots.</p>

<p>AI generated the <strong>complete</strong> pipeline in about 30 minutes of back-and-forth. The scraping worked immediately, translation was solid, and the UI had all the controls I needed. Code wasn't perfect (had to fix some error handling and rate limiting), but it was ~90% done and properly structured.</p>

<p>This works best for self-contained tools that don't need to integrate deeply with existing systems. For modifying production apps, it's better to do it piece by piece.</p>

<p>My own example:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4bvdnl6mue0zeltco3h4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4bvdnl6mue0zeltco3h4.png" alt=" " width="800" height="477"></a></p>

<p><strong>⏱️ Time Saved:</strong> 4 hours to 2 days</p>

<h2>
  
  
  Let AI Write Your Documentation
</h2>

<p>Writing docs is important and nobody wants to do it. AI is actually pretty good at this if you give it the code and tell it who the audience is.</p>

<p>Don't ask for generic documentation. Specify the audience, the format, what needs examples, and what needs warnings.</p>

<p><strong>Documentation Pattern:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Create documentation for [code/API/component] targeted at [audience].

Include:
- Brief overview of what it does
- Setup/installation steps
- Usage examples (at least 3 realistic scenarios)
- Common pitfalls and how to avoid them
- API reference (if applicable)
- Links to related docs

Format: [Markdown/Docstring/JSDoc/etc.]
</code></pre>

</div>



<p>Built a custom event tracking system that sends user behavior data to our analytics stack. Needed a documentation so other marketers could add new events without breaking anything. Gave AI the code and asked for docs with examples of tracking button clicks, form submissions, and page views.</p>

<p>Got back a documentation that actually made sense to non-developers. First time it's ever happened to me.</p>

<p><strong>⏱️ Time Saved:</strong> 1 to 3 hours</p>

<h2>
  
  
  Use AI for "What's Wrong With This" Reviews
</h2>

<p>When your code works but feels off, ask AI to review it like a senior engineer would. Not just for bugs, but for code smell, performance issues, security problems, and maintainability.</p>

<p><strong>Code Review Prompts:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Review Type</th>
<th>Prompt</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Security</strong></td>
<td>"Review this code for security vulnerabilities. Look for <a href="https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF" rel="noopener noreferrer">SQL injection</a>, XSS, insecure dependencies, and auth bypass issues."</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>"Analyze this code for performance problems. Identify <a href="https://www.google.com/search?q=https://www.pingcap.com/article/how-to-efficiently-solve-the-n1-query-problem" rel="noopener noreferrer">N+1 queries</a>, unnecessary loops, and expensive operations."</td>
</tr>
<tr>
<td><strong>Maintainability</strong></td>
<td>"Review this code for maintainability. Point out complex logic, poor naming, missing error handling, and tight coupling."</td>
</tr>
<tr>
<td><strong>Best Practices</strong></td>
<td>"Review this against [language/framework] best practices. What patterns am I violating? What's going to confuse the next dev?"</td>
</tr>
</tbody>
</table></div>

<p>I had an API endpoint that worked fine in testing but felt slow. Asked AI to review it for performance and security. It caught that I was making 50 separate database calls inside a loop when I could do it in one query, spotted a missing rate limit, and found a security hole in how I was handling user input.</p>

<p><strong>⏱️ Time Saved:</strong> 2 to 6 hours (by catching issues early)</p>

<h2>
  
  
  Generate Environment-Specific Configs
</h2>

<p>Docker files, CI/CD configs, environment variable files, they're all important but boring. Each one has specific syntax that I often forget.</p>

<p>AI can generate complete config files for your stack if you tell it what you're deploying and where.</p>

<p><strong>Config Generation Needs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🚀 Stack: Framework, database, services you're using
🌍 Environments: Development, staging, production requirements
🔧 Build process: Dependencies, build steps, optimization
🧪 Testing: How tests should run in CI
📦 Artifacts: What gets deployed, where it goes
🔒 Secrets: How environment variables get managed
</code></pre>

</div>



<p>I saw a LinkedIn post about someone needing a complete Docker setup for a Next.js app with PostgreSQL. Asked for a <a href="https://docs.docker.com/build/building/multi-stage/" rel="noopener noreferrer">Dockerfile with multi-stage builds</a> for production, docker-compose for local dev with hot reloading, and a <a href="https://www.google.com/search?q=https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nextjs" rel="noopener noreferrer">GitHub Actions workflow</a> that runs tests, builds the image, and pushes to AWS ECR.</p>

<p>And guess what? He got back everything he needed with comments explaining each step. The Dockerfile was optimized for layer caching, docker-compose had correct volume mounts, and the GitHub workflow included parallel test runs. </p>

<p><strong>Disclaimer: The above section is based on someone else's story.</strong></p>

<p><strong>⏱️ Time Saved:</strong> 2 to 4 hours</p>

<h2>
  
  
  Ask AI to Generate Only The "Boring" Parts
</h2>

<p>One of the best uses of AI is admitting which parts of your job are genuinely choking your soul.</p>

<p>Form validation? Have AI write it. Type definitions? AI. <a href="https://www.typescriptlang.org/docs/handbook/2/objects.html" rel="noopener noreferrer">Converting JSON to TypeScript interfaces</a>? AI. Boilerplate CRUD endpoints? AI.</p>

<p>Keep the interesting problems for yourself. Let AI handle the stuff that makes you zone out.</p>

<p><strong>High-Value Automation:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✓ Form validation with error messages
✓ TypeScript type definitions from JSON
✓ CRUD endpoints (basic create/read/update/delete)
✓ Data transformation utilities
✓ Mock data generators for testing
✓ Regex patterns for common validations
✓ Basic error handling wrappers
✓ Logging setup and configuration
</code></pre>

</div>



<p>This is where AI really shines. Not replacing you, just doing the things you'd do anyway but faster.</p>

<p><strong>⏱️ Time Saved:</strong> 15 minutes to 2 hours per task</p>

<h2>
  
  
  Create Reusable Code Patterns
</h2>

<p>If you find yourself writing the same kind of code repeatedly, have AI generate a template you can reuse.</p>

<p>This works great for things like API error handlers, database query wrappers, <a href="https://srivastavayushmaan1347.medium.com/understanding-middlewares-a-comprehensive-guide-with-practical-examples-c80383f888d5" rel="noopener noreferrer">authentication middleware</a>, logging, or anything else that follows the same pattern every time.</p>

<p><strong>Pattern Template Request:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Create a reusable template for [pattern type] that:

Base requirements:
- [list core functionality]

Must handle:
- [list edge cases]

Should be customizable by:
- [parameters/options]

Generate TypeScript/Python/etc. with inline comments explaining how to customize it for different use cases.
</code></pre>

</div>



<p>I kept writing the same code to connect to a <a href="https://www.salesforce.com/eu/crm/what-is-crm/" rel="noopener noreferrer">CRM</a> API every time we built a new automation. Asked AI to create a reusable connector that takes care of authentication, retries failed requests, and formats the data correctly. Said that I wanted detailed error messages during testing but simple ones in production.</p>

<p>Got back a wrapper function I now use for all integrations. Changed a few field names to match our CRM structure, but the connection logic worked perfectly.</p>

<p>Now whenever I need to pull CRM data, I copy the template and customize it in 2 minutes instead of fighting with API documentation.</p>

<p><strong>⏱️ Time Saved:</strong> 30 minutes to 2 hours (accumulated over multiple uses)</p>

<h2>
  
  
  What This Actually Means
</h2>

<p>The pattern here is obvious: AI is best at structured, repetitive work with clear requirements. It's not good at creative problem-solving (yet).</p>

<p>That means your job changes, no matter if you are a dev or a marketer. Instead of writing every line of code, you're making architectural decisions, explaining requirements, reviewing generated code, and solving the interesting problems.</p>

<p>The boring parts (boilerplate, config files, CRUD, type definitions - you name it) happen faster. The creative parts (system design, user experience, performance optimization) get more of your time.</p>

<h3>
  
  
  Start This Week
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>When</th>
<th>Do This</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Today</strong></td>
<td>Use the "explain back" technique on your current bug</td>
</tr>
<tr>
<td><strong>Tomorrow</strong></td>
<td>Generate a database schema or migration script</td>
</tr>
<tr>
<td><strong>This Week</strong></td>
<td>Try the multi-step workflow for a small feature</td>
</tr>
<tr>
<td><strong>Next Week</strong></td>
<td>Review what saved you the most time and do more of that</td>
</tr>
</tbody>
</table></div>

<p>You don't need to use AI for everything. Use it for the parts that make sense for your workflow. If something works better the old way, keep doing it the old way. Thanks for reading!</p>

