---
Title: How to Setup CodeRabbit: Complete Step-by-Step Guide (2026)
Description: 
Author: Rahul Singh
Date: 2026-03-26T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Why setup CodeRabbit for AI code review
</h2>

<p><strong>Code review bottlenecks slow down every engineering team.</strong> Pull requests sit waiting for hours or days while senior engineers context-switch between their own work and reviewing other people's code. When reviews finally happen, time pressure means reviewers focus on the obvious issues and miss the subtle bugs, security gaps, and logic errors that cause production incidents later.</p>

<p><a href="https://dev.to/tool/coderabbit">CodeRabbit</a> solves this by providing instant AI-powered code review on every pull request. Within minutes of opening a PR, CodeRabbit analyzes the diff in the context of your full repository and posts human-like review comments covering logic errors, security vulnerabilities, performance anti-patterns, and code style issues. It has reviewed over 13 million pull requests across more than 2 million connected repositories, making it the most widely adopted AI code review tool available.</p>

<p>The best part is that you can set it up in under 10 minutes with zero CI/CD configuration. CodeRabbit installs as a native GitHub App (or GitLab, Azure DevOps, and Bitbucket integration) and runs on its own infrastructure. There are no GitHub Actions workflows to write, no Docker containers to manage, and no API keys to configure.</p>

<p>This guide walks through every step of setting up CodeRabbit - from creating your account to configuring advanced review behavior with <code>.coderabbit.yaml</code> to onboarding your entire team.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwmt6h59ydofxtvw1vwh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwmt6h59ydofxtvw1vwh.png" alt="CodeRabbit screenshot"></a></p>

<h2>
  
  
  What you need before you start
</h2>

<p><strong>Before beginning the setup, confirm you have the following:</strong></p>

<ul>
<li>A GitHub, GitLab, Azure DevOps, or Bitbucket account with at least one repository</li>
<li>Admin or owner permissions on the repositories where you want to install CodeRabbit (required to authorize the app)</li>
<li>At least one open pull request or the ability to create one for testing (optional but recommended for verifying the installation)</li>
</ul>

<p>No other prerequisites are needed. CodeRabbit does not require a CI/CD pipeline, a Docker installation, API keys, or any local tooling. The entire setup happens through the browser.</p>

<h2>
  
  
  Step 1: Sign up for a CodeRabbit account
</h2>

<p><strong>Navigate to coderabbit.ai and create your account.</strong> Here is how:</p>

<ol>
<li>Open <a href="https://coderabbit.ai" rel="noopener noreferrer">coderabbit.ai</a> in your browser</li>
<li>Click the "Get Started Free" button on the homepage</li>
<li>Choose your Git platform - GitHub, GitLab, Azure DevOps, or Bitbucket</li>
<li>Authorize CodeRabbit to access your account when prompted by your Git platform's OAuth screen</li>
</ol>

<p>For GitHub specifically, you will see a standard GitHub OAuth authorization page asking you to grant CodeRabbit permission to read your profile information. Click "Authorize CodeRabbit" to proceed.</p>

<p>After authorization, you land on the CodeRabbit dashboard. This is your central control panel for managing repositories, viewing review activity, and adjusting organization-level settings.</p>

<p>The free tier requires no credit card and gives you unlimited public and private repositories with AI-powered PR summaries and review comments. Rate limits of 200 files per hour and 4 PR reviews per hour apply, but for most small to mid-size teams, these limits are rarely hit in practice.</p>

<h2>
  
  
  Step 2: Install the CodeRabbit GitHub App
</h2>

<p><strong>The GitHub App installation is what connects CodeRabbit to your repositories.</strong> This is separate from the OAuth sign-in you completed in Step 1 - the OAuth grants CodeRabbit access to your identity, while the App installation grants it access to your repositories and pull requests.</p>

<ol>
<li>From the CodeRabbit dashboard, click "Add Repositories" or "Install GitHub App"</li>
<li>GitHub will display the App installation page showing the permissions CodeRabbit needs:

<ul>
<li>
<strong>Read access</strong> to repository contents, metadata, and pull requests</li>
<li>
<strong>Write access</strong> to pull request comments and checks (so it can post review comments)</li>
<li>
<strong>Webhook events</strong> for pull request creation and updates (so it knows when to review)</li>
</ul>
</li>
<li>Choose your installation scope:

<ul>
<li>
<strong>All repositories</strong> - CodeRabbit will automatically review PRs on every repository in the organization or account, including future repositories</li>
<li>
<strong>Only select repositories</strong> - Choose specific repositories from a list; you can add more later</li>
</ul>
</li>
<li>Click "Install" to complete the installation</li>
</ol>

<p>For organizations, you may need an organization owner to approve the installation. If you see a "Request" button instead of "Install," the app request will be sent to your organization owner for approval.</p>

<p>Once installed, the CodeRabbit App appears in your GitHub organization settings under Settings &gt; GitHub Apps (or in your personal account under Settings &gt; Applications &gt; Installed GitHub Apps).</p>

<h2>
  
  
  Step 3: Select and configure repositories
</h2>

<p><strong>After installing the App, configure which repositories CodeRabbit should actively review.</strong> While the App installation grants access, the CodeRabbit dashboard lets you fine-tune which repositories are active.</p>

<ol>
<li>Return to the CodeRabbit dashboard at <a href="https://app.coderabbit.ai" rel="noopener noreferrer">app.coderabbit.ai</a>
</li>
<li>Navigate to the "Repositories" tab</li>
<li>You will see all repositories that the GitHub App has access to</li>
<li>Toggle repositories on or off as needed</li>
<li>For each active repository, you can configure basic settings:

<ul>
<li>
<strong>Review language</strong> - the language CodeRabbit writes its comments in (defaults to English)</li>
<li>
<strong>Auto-review</strong> - whether CodeRabbit reviews PRs automatically or only when requested (defaults to automatic)</li>
<li>
<strong>Draft PR handling</strong> - whether to review draft PRs (defaults to skip)</li>
</ul>
</li>
</ol>

<p>For teams just getting started, enable CodeRabbit on one or two repositories first. This lets you evaluate the review quality and tune the configuration before rolling it out broadly.</p>

<h2>
  
  
  Step 4: Trigger your first AI-powered PR review
</h2>

<p><strong>Open a pull request to see CodeRabbit in action.</strong> This is the moment where the setup pays off.</p>

<ol>
<li>Create a new branch in one of your enabled repositories</li>
<li>Make a code change - even a small one like adding a new function, fixing a bug, or refactoring a method</li>
<li>Push the branch and open a pull request targeting your default branch</li>
<li>Wait 1 to 5 minutes for CodeRabbit to complete its review</li>
</ol>

<p>CodeRabbit will post several things on your PR:</p>

<ul>
<li>
<strong>A walkthrough summary</strong> as the first comment, describing what changed across all files in the PR, organized by file and purpose</li>
<li>
<strong>Inline review comments</strong> on specific lines of code where it identifies potential issues, improvements, or suggestions</li>
<li>
<strong>A review status</strong> indicating whether the review is complete and how many comments were generated</li>
</ul>

<p>Here is what a typical CodeRabbit review looks like in practice. The summary comment appears at the top of the PR conversation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Walkthrough</span>
This PR adds input validation to the user registration endpoint.
The <span class="sb">`registerUser`</span> function now validates email format, password
strength, and username uniqueness before creating the user record.
Error responses use structured error objects with specific error codes.

<span class="gu">## Changes</span>
| File | Change Summary |
|------|---------------|
| src/api/auth.ts | Added validation logic with three new checks |
| src/types/errors.ts | New error code enum and structured error type |
| tests/api/auth.test.ts | Six new test cases covering validation scenarios |
</code></pre>

</div>



<p>Inline comments appear directly on the relevant lines in the diff, just like comments from a human reviewer. Each comment includes a description of the issue, why it matters, and often a suggested fix that you can apply with one click.</p>

<p>If CodeRabbit does not post any comments within 5 minutes, check the troubleshooting section at the end of this guide.</p>

<h2>
  
  
  Step 5: Configure CodeRabbit with .coderabbit.yaml
</h2>

<p><strong>The <code>.coderabbit.yaml</code> file is where you unlock CodeRabbit's full power.</strong> While CodeRabbit works out of the box with sensible defaults, the configuration file lets you tailor review behavior to your team's specific needs, codebase structure, and coding standards.</p>

<p>Create a file named <code>.coderabbit.yaml</code> in the root of your repository:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .coderabbit.yaml</span>
<span class="na">language</span><span class="pi">:</span> <span class="s">en-US</span>

<span class="na">reviews</span><span class="pi">:</span>
  <span class="na">profile</span><span class="pi">:</span> <span class="s">chill</span>
  <span class="na">request_changes_workflow</span><span class="pi">:</span> <span class="kc">false</span>
  <span class="na">high_level_summary</span><span class="pi">:</span> <span class="kc">true</span>
  <span class="na">high_level_summary_placeholder</span><span class="pi">:</span> <span class="s2">"</span><span class="s">@coderabbitai</span><span class="nv"> </span><span class="s">summary"</span>
  <span class="na">auto_title_placeholder</span><span class="pi">:</span> <span class="s2">"</span><span class="s">@coderabbitai"</span>
  <span class="na">poem</span><span class="pi">:</span> <span class="kc">false</span>
  <span class="na">review_status</span><span class="pi">:</span> <span class="kc">true</span>
  <span class="na">collapse_walkthrough</span><span class="pi">:</span> <span class="kc">false</span>
  <span class="na">sequence_diagrams</span><span class="pi">:</span> <span class="kc">true</span>
  <span class="na">changed_files_summary</span><span class="pi">:</span> <span class="kc">true</span>
  <span class="na">path_filters</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">!**/*.lock"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">!**/*.generated.*"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">!**/dist/**"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">!**/node_modules/**"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">!**/*.min.js"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">!**/coverage/**"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">!**/__snapshots__/**"</span>
  <span class="na">path_instructions</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">src/api/**"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">Review all API endpoints for:</span>
        <span class="s">- Input validation on all parameters</span>
        <span class="s">- Proper error handling with appropriate HTTP status codes</span>
        <span class="s">- Authentication and authorization checks</span>
        <span class="s">- Rate limiting considerations</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">src/db/**"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">Review all database code for:</span>
        <span class="s">- SQL injection prevention (parameterized queries only)</span>
        <span class="s">- Proper connection handling and cleanup</span>
        <span class="s">- Transaction usage where multiple writes occur</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">**/*.test.*"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">For test files, focus on:</span>
        <span class="s">- Test coverage of edge cases</span>
        <span class="s">- Proper assertion usage</span>
        <span class="s">- Mock cleanup and isolation between tests</span>

<span class="na">chat</span><span class="pi">:</span>
  <span class="na">auto_reply</span><span class="pi">:</span> <span class="kc">true</span>

<span class="na">knowledge_base</span><span class="pi">:</span>
  <span class="na">opt_out</span><span class="pi">:</span> <span class="kc">false</span>
  <span class="na">learnings</span><span class="pi">:</span>
    <span class="na">scope</span><span class="pi">:</span> <span class="s">auto</span>
</code></pre>

</div>



<p>Commit this file to your default branch (usually <code>main</code>). CodeRabbit reads the configuration on every PR and applies your settings immediately - no restart or reinstallation is required.</p>

<h3>
  
  
  Key configuration options explained
</h3>

<p><strong><code>profile</code></strong> controls how verbose CodeRabbit's reviews are. There are three options:</p>

<ul>
<li>
<strong><code>chill</code></strong> - Fewer comments, focused on significant issues like bugs, security vulnerabilities, and logic errors. Best for teams that want minimal noise.</li>
<li>
<strong><code>assertive</code></strong> - More thorough reviews covering style, naming, documentation, and best practices in addition to bugs. Best for teams that want comprehensive feedback.</li>
<li>
<strong><code>followup</code></strong> - Like assertive, but also checks whether previous review comments were addressed in subsequent commits. Best for teams that want accountability.</li>
</ul>

<p><strong><code>path_filters</code></strong> controls which files CodeRabbit reviews. Use negation patterns (prefixed with <code>!</code>) to exclude files. Common exclusions include lock files, build output, auto-generated code, and test snapshots.</p>

<p><strong><code>path_instructions</code></strong> is the most powerful feature. It lets you provide natural language instructions for specific directories or file patterns. CodeRabbit reads these instructions and adjusts its review focus accordingly. This means your API routes get reviewed for authentication and input validation, your database code gets checked for SQL injection, and your tests get evaluated for edge case coverage - all automatically.</p>

<p><strong><code>knowledge_base</code></strong> enables CodeRabbit to learn from your repository over time. When enabled, CodeRabbit builds context about your codebase patterns, conventions, and architecture, which improves review quality as it learns.</p>

<h2>
  
  
  Step 6: Fine-tune review profiles and instructions
</h2>

<p><strong>Once you have the basic configuration working, refine it based on real review feedback.</strong> After CodeRabbit has reviewed 10 to 20 pull requests, you will have a good sense of which suggestions are helpful and which are noise.</p>

<h3>
  
  
  Adding global review instructions
</h3>

<p>You can add a top-level <code>instructions</code> field to guide CodeRabbit's behavior across all files:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">reviews</span><span class="pi">:</span>
  <span class="na">profile</span><span class="pi">:</span> <span class="s">chill</span>
  <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">This is a Node.js backend service using Express and PostgreSQL.</span>
    <span class="s">We follow these conventions:</span>
    <span class="s">- All async functions must use try/catch with proper error logging</span>
    <span class="s">- Database queries must use parameterized statements</span>
    <span class="s">- API responses must follow our standard envelope format: { data, error, meta }</span>
    <span class="s">- Do not comment on variable naming unless the name is actively misleading</span>
    <span class="s">- Focus on bugs, security issues, and logic errors over style preferences</span>
</code></pre>

</div>



<p>These global instructions apply to every file in every PR. They are particularly useful for setting the tone of reviews and preventing false positives on things your team has already decided are acceptable.</p>

<h3>
  
  
  Creating path-specific review criteria
</h3>

<p>Expand your <code>path_instructions</code> as you identify patterns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code>  <span class="na">path_instructions</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">src/middleware/**"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">Middleware must:</span>
        <span class="s">- Always call next() or send a response</span>
        <span class="s">- Not modify the request object beyond adding typed properties</span>
        <span class="s">- Include error handling that passes errors to the error middleware</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">src/migrations/**"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">Database migrations must:</span>
        <span class="s">- Be reversible (include both up and down functions)</span>
        <span class="s">- Not drop columns or tables that contain production data without a data migration step</span>
        <span class="s">- Use transactions for multi-step changes</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">*.config.*"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">Configuration files should be reviewed for:</span>
        <span class="s">- Hardcoded secrets or credentials (flag immediately)</span>
        <span class="s">- Environment-specific values that should use env variables</span>
        <span class="s">- Reasonable default values</span>
</code></pre>

</div>



<h3>
  
  
  Adjusting the review profile per team feedback
</h3>

<p>If your team finds the <code>chill</code> profile too quiet, switch to <code>assertive</code> and add exclusions for the types of comments that generate the most noise:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">reviews</span><span class="pi">:</span>
  <span class="na">profile</span><span class="pi">:</span> <span class="s">assertive</span>
  <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">Do not comment on:</span>
    <span class="s">- Import ordering (handled by our linter)</span>
    <span class="s">- Line length (handled by Prettier)</span>
    <span class="s">- Missing JSDoc comments on internal functions</span>
    <span class="s">Focus extra attention on:</span>
    <span class="s">- Error handling completeness</span>
    <span class="s">- Security vulnerabilities</span>
    <span class="s">- Performance in database queries and loops</span>
</code></pre>

</div>



<p>This approach gives you the breadth of <code>assertive</code> mode while filtering out the categories of feedback that your other tools already handle.</p>

<h2>
  
  
  Step 7: Interact with CodeRabbit in PR comments
</h2>

<p><strong>CodeRabbit is not a one-way tool - it is conversational.</strong> Every comment it posts is the start of a potential conversation. Understanding how to interact with CodeRabbit effectively makes the difference between treating it as a noisy bot and treating it as a useful team member.</p>

<h3>
  
  
  Replying to review comments
</h3>

<p>When CodeRabbit posts an inline comment on your PR, you can reply directly to it just like you would reply to a human reviewer. CodeRabbit understands natural language and responds conversationally:</p>

<ul>
<li>
<strong>Ask for clarification:</strong> "Why is this a security issue? We validate the input in the middleware layer."</li>
<li>
<strong>Request an alternative:</strong> "This approach would break our caching layer. Can you suggest an alternative that preserves the cache key?"</li>
<li>
<strong>Accept and ask for the fix:</strong> "Good catch. Can you generate the fix for this?"</li>
</ul>

<p>CodeRabbit remembers the context of the conversation within the same PR thread, so follow-up questions work naturally.</p>

<h3>
  
  
  Using command keywords
</h3>

<p>CodeRabbit responds to specific commands posted as PR comments:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>What it does</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>@coderabbitai review</code></td>
<td>Triggers a full re-review of the PR</td>
</tr>
<tr>
<td><code>@coderabbitai resolve</code></td>
<td>Dismisses a specific review comment</td>
</tr>
<tr>
<td><code>@coderabbitai explain</code></td>
<td>Provides a detailed explanation of a finding</td>
</tr>
<tr>
<td><code>@coderabbitai generate docstring</code></td>
<td>Generates documentation for the function in context</td>
</tr>
<tr>
<td><code>@coderabbitai configuration</code></td>
<td>Shows the current configuration for the repository</td>
</tr>
<tr>
<td><code>@coderabbitai help</code></td>
<td>Lists all available commands</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Teaching CodeRabbit your preferences
</h3>

<p>Every interaction you have with CodeRabbit teaches it about your preferences. When you dismiss a comment with an explanation ("We intentionally use any here because the type comes from a third-party library"), CodeRabbit stores that as a learning and avoids making the same suggestion in future reviews.</p>

<p>This feedback loop is critical during the first two weeks of adoption. Teams that actively reply to and correct CodeRabbit's comments typically see a significant reduction in false positives by the end of the first month.</p>

<h2>
  
  
  Step 8: Onboard your team
</h2>

<p><strong>Rolling out CodeRabbit to a team requires communication and a gradual approach.</strong> Developers who discover automated review comments on their PR without context tend to react negatively - it feels like surveillance rather than support. A thoughtful rollout makes the difference between adoption and rejection.</p>

<h3>
  
  
  Week 1: Pilot with early adopters
</h3>

<ol>
<li>
<strong>Select one or two pilot repositories</strong> - choose repos with active PR activity and developers who are open to trying new tools</li>
<li>
<strong>Set the review profile to <code>chill</code></strong> - this minimizes noise during the evaluation period</li>
<li>
<strong>Brief the pilot group</strong> - send a short message explaining what CodeRabbit is, that it will post AI review comments on PRs, and that the feedback is advisory (not blocking merges)</li>
<li>
<strong>Collect feedback</strong> - ask pilot users to note which comments were helpful and which were noise</li>
</ol>

<h3>
  
  
  Week 2: Tune and expand
</h3>

<ol>
<li>
<strong>Review pilot feedback</strong> - identify common false positives and add path filters or instructions to address them</li>
<li>
<strong>Update <code>.coderabbit.yaml</code></strong> based on what you learned during the pilot</li>
<li>
<strong>Expand to 3 to 5 additional repositories</strong> - choose a mix of frontend and backend repos to test across different codebases</li>
<li>
<strong>Share examples of helpful catches</strong> with the broader team to build confidence in the tool</li>
</ol>

<h3>
  
  
  Week 3 and beyond: Full rollout
</h3>

<ol>
<li>
<strong>Enable CodeRabbit on all active repositories</strong> either through the dashboard or by switching the GitHub App to "All repositories" mode</li>
<li>
<strong>Consider switching to the <code>assertive</code> profile</strong> if the team wants more comprehensive feedback</li>
<li>
<strong>Set up branch protection</strong> to require conversation resolution on PRs (this ensures developers read CodeRabbit's comments without making the AI check a blocking requirement)</li>
<li>
<strong>Establish a feedback cadence</strong> - review CodeRabbit's effectiveness monthly and adjust the configuration as needed</li>
</ol>

<h3>
  
  
  Communicating the rollout
</h3>

<p>Here is a template message you can adapt for your team:</p>

<blockquote>
<p>We are setting up CodeRabbit for AI-powered code review on our repositories. Starting today, you will see automated review comments on your pull requests from the CodeRabbit bot. These comments are advisory - they do not block merges - and cover things like potential bugs, security issues, and logic errors. The goal is to get instant feedback so you can fix issues before human reviewers look at the PR, not to add more hoops. If you see a comment that is wrong or unhelpful, reply to it and explain why. CodeRabbit learns from your feedback and gets better over time. Questions? Ask in our team channel.</p>
</blockquote>

<h2>
  
  
  Step 9: Configure advanced settings
</h2>

<p><strong>Once your team is comfortable with the basics, explore these advanced configuration options.</strong></p>

<h3>
  
  
  Enabling auto-fix suggestions
</h3>

<p>On the Pro plan, CodeRabbit can generate one-click fix suggestions for issues it identifies. These appear as GitHub suggested changes that you can commit directly from the PR interface:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">reviews</span><span class="pi">:</span>
  <span class="na">auto_review</span><span class="pi">:</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
  <span class="na">tools</span><span class="pi">:</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
</code></pre>

</div>



<h3>
  
  
  Integrating with Jira or Linear
</h3>

<p>CodeRabbit can read linked Jira or Linear issues to understand the context of a PR. When it knows the ticket description and acceptance criteria, it can validate whether the PR actually addresses the requirements:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">integrations</span><span class="pi">:</span>
  <span class="na">jira</span><span class="pi">:</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
    <span class="na">project_keys</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">BACKEND"</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">FRONTEND"</span>
  <span class="na">linear</span><span class="pi">:</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
</code></pre>

</div>



<h3>
  
  
  Setting up Slack notifications
</h3>

<p>Route review summaries to a Slack channel so the team can see review activity without leaving Slack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">integrations</span><span class="pi">:</span>
  <span class="na">slack</span><span class="pi">:</span>
    <span class="na">enabled</span><span class="pi">:</span> <span class="kc">true</span>
    <span class="na">channel</span><span class="pi">:</span> <span class="s2">"</span><span class="s">#code-reviews"</span>
</code></pre>

</div>



<h3>
  
  
  Configuring for monorepos
</h3>

<p>If your repository is a monorepo with multiple services, use path instructions to apply different review criteria to each service:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">reviews</span><span class="pi">:</span>
  <span class="na">path_instructions</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">services/auth/**"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">This is the authentication service. Focus on:</span>
        <span class="s">- Token validation and expiration handling</span>
        <span class="s">- Password hashing using bcrypt (not MD5 or SHA)</span>
        <span class="s">- OWASP authentication best practices</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">services/payments/**"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">This is the payment processing service. Focus on:</span>
        <span class="s">- PCI DSS compliance patterns</span>
        <span class="s">- Idempotency of payment operations</span>
        <span class="s">- Decimal precision for currency calculations</span>
    <span class="pi">-</span> <span class="na">path</span><span class="pi">:</span> <span class="s2">"</span><span class="s">packages/shared/**"</span>
      <span class="na">instructions</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">This is shared library code used by all services. Focus on:</span>
        <span class="s">- Backward compatibility of exported APIs</span>
        <span class="s">- Type safety and generic usage</span>
        <span class="s">- Performance since this code runs in every service</span>
</code></pre>

</div>



<h2>
  
  
  Step 10: Troubleshoot common issues
</h2>

<p><strong>If CodeRabbit is not working as expected, work through these common issues in order.</strong></p>

<h3>
  
  
  CodeRabbit is not reviewing pull requests
</h3>

<ol>
<li>
<strong>Check the GitHub App installation.</strong> Go to your GitHub organization or account settings, then to Applications &gt; Installed GitHub Apps. Verify that CodeRabbit is listed and has access to the repository in question.</li>
<li>
<strong>Check repository status in the dashboard.</strong> Log in to <a href="https://app.coderabbit.ai" rel="noopener noreferrer">app.coderabbit.ai</a> and verify that the repository is toggled on.</li>
<li>
<strong>Check if the PR is a draft.</strong> CodeRabbit skips draft pull requests by default. Either mark the PR as ready for review or configure CodeRabbit to review drafts.</li>
<li>
<strong>Check path filters.</strong> If your <code>.coderabbit.yaml</code> excludes the paths that changed in the PR, CodeRabbit will have nothing to review and may not post any comments.</li>
<li>
<strong>Check rate limits.</strong> On the free tier, you are limited to 4 PR reviews per hour. If you have exceeded this limit, reviews are queued.</li>
<li>
<strong>Check the webhook delivery.</strong> In your GitHub repository settings, navigate to Webhooks and check the recent deliveries for the CodeRabbit webhook. Failed deliveries indicate a connectivity issue between GitHub and CodeRabbit's servers.</li>
</ol>

<h3>
  
  
  CodeRabbit posts too many comments
</h3>

<ol>
<li>
<strong>Switch the review profile to <code>chill</code></strong> if you are currently on <code>assertive</code> or <code>followup</code>
</li>
<li>
<strong>Add path filters</strong> to exclude auto-generated files, lock files, and build artifacts</li>
<li>
<strong>Add global instructions</strong> telling CodeRabbit what not to comment on (e.g., "Do not comment on import ordering, line length, or missing comments")</li>
<li>
<strong>Exclude test snapshot files</strong> and other files that change frequently but do not need AI review</li>
</ol>

<h3>
  
  
  CodeRabbit comments are not relevant to your codebase
</h3>

<ol>
<li>
<strong>Add path-specific instructions</strong> describing the architecture and conventions of each part of your codebase</li>
<li>
<strong>Reply to irrelevant comments</strong> with explanations of why they do not apply - CodeRabbit learns from these interactions</li>
<li>
<strong>Add framework-specific context</strong> in the global instructions (e.g., "This is a Next.js application using the App Router. Server components do not have access to browser APIs.")</li>
<li>
<strong>Enable the knowledge base</strong> so CodeRabbit builds context about your repository over time:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">knowledge_base</span><span class="pi">:</span>
  <span class="na">opt_out</span><span class="pi">:</span> <span class="kc">false</span>
  <span class="na">learnings</span><span class="pi">:</span>
    <span class="na">scope</span><span class="pi">:</span> <span class="s">auto</span>
</code></pre>

</div>



<h3>
  
  
  YAML configuration is not taking effect
</h3>

<ol>
<li>
<strong>Verify the file name.</strong> It must be exactly <code>.coderabbit.yaml</code> (not <code>.coderabbit.yml</code>, not <code>coderabbit.yaml</code>)</li>
<li>
<strong>Verify the file location.</strong> It must be in the repository root, not in a subdirectory</li>
<li>
<strong>Verify the file is on the default branch.</strong> CodeRabbit reads the configuration from the default branch (usually <code>main</code>), not from the PR branch</li>
<li>
<strong>Validate the YAML syntax.</strong> Use a YAML validator to check for indentation errors, missing colons, or incorrect nesting</li>
<li>
<strong>Trigger a re-review</strong> by commenting <code>@coderabbitai review</code> on a PR to force CodeRabbit to re-read the configuration</li>
</ol>

<h3>
  
  
  Permission errors during installation
</h3>

<ol>
<li>
<strong>For personal accounts:</strong> Ensure you are the account owner. Only the owner can install GitHub Apps.</li>
<li>
<strong>For organizations:</strong> GitHub App installations require organization owner approval. If you see a "Request" button instead of "Install," the request is sent to your org owner. Ask them to approve the CodeRabbit app in the organization settings under Third-party access.</li>
<li>
<strong>For repositories with restricted access:</strong> Some organizations restrict which apps can access specific repositories. Check your organization's GitHub App policies and ensure CodeRabbit is allowed.</li>
</ol>

<h2>
  
  
  Best practices for getting the most out of CodeRabbit
</h2>

<p><strong>Follow these practices to maximize the value CodeRabbit delivers to your team.</strong></p>

<p><strong>Start with the chill profile and increase strictness over time.</strong> The worst outcome is overwhelming your team with comments on day one and having them tune out all automated feedback permanently. Begin with <code>chill</code>, let the team build trust in the tool, and then switch to <code>assertive</code> after two to four weeks.</p>

<p><strong>Write descriptive PR titles and descriptions.</strong> CodeRabbit uses the PR title and description as context for its review. A PR titled "fix bug" gives CodeRabbit almost nothing to work with. A PR titled "Fix race condition in user session cleanup" with a description of the problem helps CodeRabbit provide more targeted and useful feedback.</p>

<p><strong>Keep pull requests small.</strong> CodeRabbit - like human reviewers - provides better feedback on focused PRs. A PR that changes 50 lines across 3 files gets more precise comments than a PR that changes 2,000 lines across 40 files. Aim for PRs under 400 lines of changed code.</p>

<p><strong>Reply to comments instead of silently dismissing them.</strong> Every reply teaches CodeRabbit about your preferences. A team that spends two weeks actively correcting CodeRabbit's false positives ends up with a tool that is far more accurate than a team that silently ignores everything.</p>

<p><strong>Use CodeRabbit alongside static analysis, not instead of it.</strong> CodeRabbit catches logic errors, design issues, and contextual problems that rule-based tools miss. Static analysis tools like <a href="https://dev.to/tool/sonarqube">SonarQube</a> and <a href="https://dev.to/tool/semgrep">Semgrep</a> provide deterministic bug detection, security scanning, and quality gate enforcement that AI review cannot replace. The strongest setup uses both layers together.</p>

<p><strong>Review the CodeRabbit dashboard monthly.</strong> The dashboard shows metrics like review activity, common issue types, and resolution rates. Use these insights to adjust your <code>.coderabbit.yaml</code> configuration and identify areas of your codebase that generate the most review findings.</p>

<h2>
  
  
  What comes after setup
</h2>

<p><strong>Once CodeRabbit is running smoothly, consider these next steps.</strong></p>

<p>Upgrade to the Pro plan if your team consistently hits the free-tier rate limits or wants access to auto-fix suggestions, 40+ built-in linters, and integrations with Jira, Linear, and Slack. The Pro plan costs $24/user/month on annual billing, and CodeRabbit only charges for developers who create pull requests - reviewers, managers, and non-coding team members are not counted.</p>

<p>Add static analysis tools like <a href="https://dev.to/tool/sonarqube">SonarQube</a> or <a href="https://dev.to/tool/semgrep">Semgrep</a> to complement CodeRabbit's AI review with deterministic rule-based scanning. Our guide on <a href="https://dev.to/blog/how-to-automate-code-review">how to automate code reviews</a> covers the full three-layer setup including linting, static analysis, and AI review.</p>

<p>Explore CodeRabbit's advanced features including custom review profiles for different teams within your organization, integration with your issue tracker for context-aware reviews, and the knowledge base for repository-specific learning.</p>

<p>The goal is a review pipeline where CodeRabbit handles the initial pass - catching bugs, flagging security issues, and validating logic - so that your human reviewers can focus on architecture, design decisions, and the high-level questions that actually require human judgment. Teams that reach this state typically report a 30 to 60 percent reduction in review cycle time and significantly fewer defects reaching production.</p>

<h2>
  
  
  Frequently Asked Questions
</h2>

<h3>
  
  
  How do I setup CodeRabbit on GitHub?
</h3>

<p>Go to coderabbit.ai and click Get Started Free. Sign in with your GitHub account and authorize the CodeRabbit GitHub App. Select the repositories you want to enable - either all repositories or specific ones. Once installed, CodeRabbit automatically reviews every new pull request. No GitHub Actions workflow or CI/CD configuration is needed. The entire setup takes under 5 minutes.</p>

<h3>
  
  
  Is CodeRabbit free to use?
</h3>

<p>Yes. CodeRabbit offers a free tier that covers unlimited public and private repositories with AI-powered PR summaries and review comments. The free tier has rate limits of 200 files per hour and 4 PR reviews per hour but imposes no cap on team members or repositories. CodeRabbit Pro is also free forever for open-source projects with public repositories. The Pro plan at $24/user/month removes rate limits and adds advanced features like auto-fix suggestions and 40+ built-in linters.</p>

<h3>
  
  
  How do I configure CodeRabbit with a YAML file?
</h3>

<p>Create a file named .coderabbit.yaml in your repository root. This file controls review behavior including language, review profile (chill, assertive, or followup), path filters, path-specific instructions, and chat settings. Commit and push the file to your default branch. CodeRabbit reads this configuration on every PR and applies your custom settings automatically. No restart or reinstallation is required after changing the config.</p>

<h3>
  
  
  What is the .coderabbit.yaml file?
</h3>

<p>The .coderabbit.yaml file is CodeRabbit's repository-level configuration file. It lives in your repository root and controls how CodeRabbit reviews pull requests for that specific repository. You can configure the review profile, exclude file paths from review, add natural language instructions for specific directories, enable or disable features like sequence diagrams and PR summaries, and set the review language. Each repository can have its own configuration.</p>

<h3>
  
  
  How do I exclude files from CodeRabbit review?
</h3>

<p>Add a path_filters section under reviews in your .coderabbit.yaml file. Use negation patterns with an exclamation mark prefix to exclude paths. For example, adding '!<strong>/*.lock' excludes all lock files, '!</strong>/dist/<strong>' excludes build output, and '!</strong>/node_modules/**' excludes dependencies. CodeRabbit supports glob patterns, so you can be as specific or broad as needed. Changes to path filters take effect on the next PR without any restart.</p>

<h3>
  
  
  How do I interact with CodeRabbit in pull request comments?
</h3>

<p>Reply directly to any CodeRabbit comment on your PR to start a conversation. You can ask it to explain a finding, suggest a different approach, or generate code. Use @coderabbitai review to trigger a full re-review. Use @coderabbitai resolve to dismiss a comment. Use @coderabbitai generate docstring to create documentation for a function. CodeRabbit responds conversationally and remembers context within the same PR thread.</p>

<h3>
  
  
  Does CodeRabbit work with GitLab, Azure DevOps, and Bitbucket?
</h3>

<p>Yes. CodeRabbit supports GitHub, GitLab, Azure DevOps, and Bitbucket. The setup process is similar across all platforms - sign in with your platform account, authorize CodeRabbit, and select repositories. The .coderabbit.yaml configuration file works identically across all supported platforms. GitLab users authenticate through GitLab OAuth, and Azure DevOps and Bitbucket users connect through their respective platform authentication flows.</p>

<h3>
  
  
  How long does CodeRabbit take to review a pull request?
</h3>

<p>CodeRabbit typically completes its review within 1 to 5 minutes after a pull request is opened or updated. Review time depends on the size of the diff and the complexity of the changes. Small PRs with fewer than 100 changed lines are usually reviewed in under 2 minutes. Large PRs with thousands of changed lines may take up to 5 minutes. On the free tier, if you exceed 4 reviews per hour, additional reviews are queued until capacity is available.</p>

<h3>
  
  
  Can I customize what CodeRabbit focuses on during review?
</h3>

<p>Yes. Use the path_instructions section in .coderabbit.yaml to provide natural language instructions for specific directories or file patterns. For example, you can tell CodeRabbit to focus on input validation and authentication for API routes, or to check for SQL injection prevention in database code. You can also set a global review instruction using the instructions field. CodeRabbit also learns from your interactions - when you dismiss or accept suggestions, it adjusts future reviews accordingly.</p>

<h3>
  
  
  What is the difference between chill, assertive, and followup review profiles?
</h3>

<p>The chill profile provides fewer comments and focuses only on significant issues like bugs, security vulnerabilities, and logic errors. The assertive profile is more thorough and comments on style, naming, documentation, and best practices in addition to bugs and security. The followup profile is similar to assertive but also follows up on previous review comments to check whether they were addressed in subsequent commits. Most teams start with the chill profile and switch to assertive once they are comfortable with the tool.</p>

<h3>
  
  
  How do I onboard my team to CodeRabbit?
</h3>

<p>Start by installing CodeRabbit on one or two pilot repositories and setting the review profile to chill. Share a brief message with your team explaining that CodeRabbit will post AI review comments on PRs and that the feedback is advisory, not blocking. Encourage developers to reply to comments rather than silently dismissing them. After one to two weeks, review the team's experience, adjust the .coderabbit.yaml configuration based on feedback, and expand to additional repositories. Avoid enabling all repositories at once.</p>

<h3>
  
  
  Why is CodeRabbit not reviewing my pull requests?
</h3>

<p>Check these common causes: the CodeRabbit GitHub App may not be installed on the repository, the repository may be excluded in your CodeRabbit dashboard settings, the PR may be marked as a draft (CodeRabbit skips drafts by default), or the PR may only contain changes to files excluded by your path_filters configuration. Also verify that the app has the necessary permissions by checking the GitHub Apps section in your repository settings. If none of these apply, visit the CodeRabbit dashboard to check for error messages or rate limit status.</p>

<h3>
  
  
  Can I use CodeRabbit alongside SonarQube or other static analysis tools?
</h3>

<p>Yes, and many teams do. CodeRabbit provides AI-powered semantic review while tools like SonarQube, Semgrep, and DeepSource provide deterministic rule-based analysis. They complement each other because CodeRabbit catches logic errors, missing edge cases, and contextual issues that rules cannot cover, while static analysis tools enforce quality gates and track technical debt over time. There are no conflicts between running CodeRabbit and other tools on the same repository.</p>




<p><em>Originally published at <a href="https://aicodereview.cc/blog/how-to-setup-coderabbit/" rel="noopener noreferrer">aicodereview.cc</a></em></p>

