---
Title: I Stopped Fighting My AI How Kiro's agent hooks and steering files fixed my biggest frustration with AI coding tools
Description: 
Author: Ibrahim Pima
Date: 2025-11-26T20:59:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been using AI coding assistants for about 8 months now. Cursor, GitHub Copilot, the usual suspects.</p>

<p>hey're all impressive, but they share one fatal flaw that's been driving me insane:</p>

<p><strong>They forget everything.</strong></p>

<p>You explain your project architecture. The AI generates some code. Five minutes later, you're explaining the same architecture again. Every session starts from scratch. Every feature request requires re-explaining your context.</p>

<p>I found myself maintaining a text file of prompts to copy-paste at the start of each session:</p>

<ul>
<li>"We use Zustand for state management, not Redux"</li>
<li>"All API calls go through our custom fetcher with retry logic"
</li>
<li>"Components follow the compound component pattern"</li>
<li>"Test files go in <code>__tests__</code> not next to the component"</li>
</ul>

<p>I was spending 10% of my coding time just explaining my project to the AI. Again and again and again.</p>

<p>Then I tried <a href="https://kiro.dev" rel="noopener noreferrer">Kiro</a>, and it fundamentally changed how I think about AI-assisted development.</p>

<h2>
  
  
  The Context Problem
</h2>

<p>Let me show you what I mean with a real example.</p>

<p>Last month I was building a dashboard for monitoring crypto wallets. Standard stuff: React frontend, Node backend, PostgreSQL database.</p>

<p>With my previous AI tool, here's how a typical session would go:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Me: "Add a new endpoint for fetching transaction history"

AI: *generates endpoint with inline SQL queries*

Me: "No, we use Prisma for database access"

AI: *rewrites with Prisma*

Me: "And all endpoints need rate limiting"

AI: *adds rate limiting*

Me: "And proper error handling with our custom error classes"

AI: *adds error handling*
</code></pre>

</div>



<p>Four back-and-forth messages just to get code that follows my project's patterns. Then 20 minutes later:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Me: "Add an endpoint for fetching wallet balances"

AI: *generates endpoint with inline SQL queries again*
</code></pre>

</div>



<p>We're back to square one.</p>

<h2>
  
  
  Enter Steering Files
</h2>

<p>Kiro solves this with something called <strong>steering files</strong> - persistent markdown documents that live in your project at <code>.kiro/steering/</code>.</p>

<p>Here's what I put in my steering file for that dashboard project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Database Access</span>

All database operations use Prisma. Never write raw SQL queries.

Example:
</code></pre>

</div>

<p><br>
typescript<br>
const transactions = await prisma.transaction.findMany({<br>
  where: { walletId },<br>
  orderBy: { timestamp: 'desc' }<br>
});<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
# API Patterns

All endpoints must include:
- Rate limiting via express-rate-limit
- Error handling with AppError class
- Request validation with Zod schemas
- Proper HTTP status codes

# Testing Standards

- Test files in `__tests__/` directory
- Use Vitest, not Jest
- Mock Prisma with prisma-mock
- Minimum 80% coverage
</code></pre>

</div>



<p>You write this once. Kiro reads it. Forever.</p>

<p>Now when I ask for a new endpoint, Kiro generates code that already follows these patterns. No more re-explaining. No more "actually we do it this way."</p>

<p>The AI finally <strong>maintains context</strong> instead of just <strong>having context</strong> for a single conversation.</p>

<h2>
  
  
  Agent Hooks: The Game Changer
</h2>

<p>But steering files were just the beginning. The feature that actually changed my workflow was <strong>agent hooks</strong>.</p>

<p>Agent hooks are event-driven automations written in natural language. They're like GitHub Actions but for your local development environment, powered by AI.</p>

<p>Here's a hook I set up in about 30 seconds:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">trigger</span><span class="pi">:</span> <span class="s">onSave</span>
<span class="na">pattern</span><span class="pi">:</span> <span class="s2">"</span><span class="s">**/*.tsx"</span>
<span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
  <span class="s">When a React component is saved:</span>
  <span class="s">1. Check if a corresponding test file exists in __tests__</span>
  <span class="s">2. If not, create one with basic render tests</span>
  <span class="s">3. If it exists, update it to cover any new props or functions</span>
  <span class="s">4. Run the test to verify it passes</span>
</code></pre>

</div>



<p>That's it. Plain English. No complex scripting.</p>

<p>Now every time I save a React component, Kiro automatically:</p>

<ul>
<li>Creates or updates the test file</li>
<li>Adds tests for new props/functions</li>
<li>Runs the tests</li>
<li>Shows me the results</li>
</ul>

<p>I haven't manually created a test file in a week. The hook just handles it.</p>

<h3>
  
  
  Real-World Hook Examples
</h3>

<p>Here are some other hooks I've set up:</p>

<p><strong>Auto-update API documentation:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">trigger</span><span class="pi">:</span> <span class="s">onSave</span>
<span class="na">pattern</span><span class="pi">:</span> <span class="s2">"</span><span class="s">src/api/**/*.ts"</span>
<span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
  <span class="s">When an API route file changes:</span>
  <span class="s">1. Extract endpoint, method, parameters, response types</span>
  <span class="s">2. Update the corresponding section in docs/API.md</span>
  <span class="s">3. Ensure the example requests are current</span>
</code></pre>

</div>



<p><strong>Security check before commit:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">trigger</span><span class="pi">:</span> <span class="s">onManual</span>
<span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
  <span class="s">Before committing, scan for:</span>
  <span class="s">- Hardcoded API keys or secrets</span>
  <span class="s">- Console.log statements in production code</span>
  <span class="s">- TODO comments with no corresponding issue</span>
  <span class="s">- Files larger than 500 lines that should be split</span>
</code></pre>

</div>



<p><strong>Keep dependencies in sync:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">trigger</span><span class="pi">:</span> <span class="s">onSave</span>
<span class="na">pattern</span><span class="pi">:</span> <span class="s2">"</span><span class="s">package.json"</span>
<span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
  <span class="s">When package.json changes:</span>
  <span class="s">1. Check if any new dependencies need configuration</span>
  <span class="s">2. Update the development setup docs</span>
  <span class="s">3. Verify no conflicting versions</span>
</code></pre>

</div>



<p>Each hook took less than a minute to set up. Together they've probably saved me 10+ hours over the past two weeks.</p>

<h2>
  
  
  Specs: Planning Before Coding
</h2>

<p>The third piece of Kiro's workflow is <strong>specs</strong> - AI-generated specification documents that break down features before you code them.</p>

<p>Instead of going straight from idea to code, Kiro helps you:</p>

<ol>
<li>Define requirements in EARS notation (more on this later)</li>
<li>Generate a design document with data flows and interfaces</li>
<li>Break the feature into sequenced tasks</li>
<li>Link each task back to requirements</li>
</ol>

<p>Here's what this looked like for my transaction history feature:</p>

<p>I started with: "Add transaction history with filtering and export"</p>

<p>Kiro generated a spec that included:</p>

<ul>
<li>User requirements ("User can filter by date range, amount, type")</li>
<li>System design (data flow diagrams, API contracts)</li>
<li>Task breakdown (8 sequential tasks with dependencies)</li>
<li>Test criteria for each task</li>
</ul>

<p>The spec caught issues I would've missed:</p>

<ul>
<li>Need to handle pagination for large transaction lists</li>
<li>Export needs to work with filters applied</li>
<li>Date ranges need timezone handling</li>
<li>What happens with failed transactions?</li>
</ul>

<p>Without the spec, I would've coded for 3 hours and then realized "oh crap, I forgot about timezones."</p>

<p>With the spec, these edge cases were handled upfront.</p>

<h2>
  
  
  What EARS Notation Actually Is
</h2>

<p>Quick sidebar: EARS (Easy Approach to Requirements Syntax) is a format for writing clear, unambiguous requirements.</p>

<p>Instead of:</p>

<blockquote>
<p>"The system should allow users to export transactions"</p>
</blockquote>

<p>EARS format:</p>

<blockquote>
<p>"When the user clicks export, the system shall generate a CSV file containing all displayed transactions"</p>
</blockquote>

<p>It's more explicit about triggers, conditions, and actions. Makes it way harder for the AI (or human developers) to misinterpret what you want.</p>

<p>Kiro generates these automatically from your natural language description, then lets you refine them before coding starts.</p>

<h2>
  
  
  The Development Flow
</h2>

<p>Here's how my workflow changed:</p>

<p><strong>Before Kiro:</strong></p>

<ol>
<li>Think of feature</li>
<li>Start coding</li>
<li>Realize I need to explain context to AI</li>
<li>Copy-paste my project patterns</li>
<li>Generate code</li>
<li>Fix all the things that don't match my patterns</li>
<li>Manually write tests</li>
<li>Manually update docs</li>
<li>Commit</li>
</ol>

<p><strong>With Kiro:</strong></p>

<ol>
<li>Describe feature to Kiro</li>
<li>Review/refine the generated spec</li>
<li>Approve spec</li>
<li>Kiro implements following the spec</li>
<li>Hooks automatically update tests and docs</li>
<li>Review and commit</li>
</ol>

<p>Steps 3-6 from the old flow are just... gone. Handled by steering + hooks.</p>

<h2>
  
  
  Real Impact: A Case Study
</h2>

<p>Last week I needed to add a notification system to the dashboard. Users wanted alerts for:</p>

<ul>
<li>Large transactions</li>
<li>Failed transactions
</li>
<li>Low wallet balances</li>
<li>Unusual activity</li>
</ul>

<p><strong>Traditional AI assistant approach:</strong></p>

<ul>
<li>Would've taken me ~6 hours</li>
<li>Would've required constant hand-holding</li>
<li>Would've missed edge cases</li>
<li>Tests written last (if at all)</li>
<li>Documentation updated manually (if I remembered)</li>
</ul>

<p><strong>With Kiro:</strong></p>

<ul>
<li>Created a spec: 15 minutes</li>
<li>Refined requirements: 10 minutes</li>
<li>Kiro implemented: 2 hours</li>
<li>Hooks kept tests/docs in sync automatically</li>
<li>Total: ~2.5 hours</li>
</ul>

<p>But more importantly: <strong>the code was consistent with my existing patterns from the start</strong>. No "actually we use our custom notification service" corrections. No forgetting to add rate limiting. No missing tests.</p>

<p>The steering file had all the context. The hooks enforced the patterns. The spec caught the edge cases.</p>

<h2>
  
  
  Setting Up Your First Hook
</h2>

<p>If you want to try this, here's the simplest possible hook to start with:</p>

<ol>
<li>Open Kiro (it's built on VS Code, so feels familiar)</li>
<li>Click the Kiro ghost icon in the sidebar</li>
<li>Go to "Agent Hooks"</li>
<li>Click the + button</li>
<li>Type in natural language what you want:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>When I save a file, check for console.log statements
and comment them out with a note that they should
be removed before production
</code></pre>

</div>



<p>That's it. Kiro converts your natural language to a hook configuration and starts running it.</p>

<p>Try it with something simple first:</p>

<ul>
<li>"When I save, run prettier"</li>
<li>"When I create a .tsx file, add my standard imports"</li>
<li>"When I modify package.json, update the README dependencies section"</li>
</ul>

<p>Once you see how easy it is, you'll start thinking of hooks for everything.</p>

<h2>
  
  
  The MCP Integration
</h2>

<p>One more thing: Kiro has native Model Context Protocol (MCP) support.</p>

<p>This means you can connect external tools and data sources:</p>

<ul>
<li>Your company's internal documentation</li>
<li>Your database schema</li>
<li>Your API specs from Postman/Swagger</li>
<li>Web search for looking up library docs</li>
<li>Slack for checking team decisions</li>
</ul>

<p>I connected our company's Notion wiki as an MCP server. Now when Kiro generates code, it references our actual internal patterns and decisions, not just generic best practices.</p>

<h2>
  
  
  What About Terminal Issues?
</h2>

<p>Full transparency: Kiro's terminal integration isn't perfect yet. Sometimes commands don't execute reliably, which can be frustrating if you're trying to run tests or start servers directly.</p>

<p>But honestly? The hooks + steering + specs workflow is so much better than what I was using before that the terminal issues are a minor annoyance, not a dealbreaker.</p>

<p>Plus, Kiro is still early (backed by AWS, actively developing). These rough edges are getting smoothed out.</p>

<h2>
  
  
  Should You Try It?
</h2>

<p>Here's my honest take:</p>

<p><strong>Try Kiro if:</strong></p>

<ul>
<li>You're tired of re-explaining your project to your AI</li>
<li>You have patterns/standards you want enforced automatically</li>
<li>You work on projects that need consistent structure</li>
<li>You're building something more complex than a prototype</li>
</ul>

<p><strong>Stick with Cursor/Copilot if:</strong></p>

<ul>
<li>You just need quick code generation</li>
<li>You mostly do one-off scripts or small projects</li>
<li>You don't care about maintaining consistency</li>
<li>You need rock-solid stability right now</li>
</ul>

<p>For me, the context persistence + automation was worth switching. I'm spending way less time fighting with my AI assistant and way more time actually building.</p>

<h2>
  
  
  Try It Yourself
</h2>

<p>Kiro is free while in early access. Download it at <a href="https://kiro.dev" rel="noopener noreferrer">kiro.dev</a>.</p>

<p>Start with:</p>

<ol>
<li>Create one steering file with your project's patterns</li>
<li>Set up one simple hook (like the console.log checker above)</li>
<li>Try creating a spec for your next feature</li>
</ol>

<p>See if it clicks for you like it did for me.</p>




<p><strong>Have you tried Kiro? What's your biggest frustration with AI coding assistants?</strong> Drop a comment below.</p>




