---
Title: How I Turned a Bespoke Code Reviewer Into a Skill Any Project Can Use
Description: 
Author: Shane Wilkey
Date: 2026-03-13T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>The review layer in Code Genie works. The problem is it's hardwired — the criteria, the persona, the output format are all written specifically for that project. Every new project that wants the same quality gate has to rebuild it from scratch. That's not a system, that's a copy-paste habit. Here's how I extracted it into a reusable Skill.</p>




<h2>
  
  
  The Problem With Bespoke Logic
</h2>

<p>Here's what the <a href="https://github.com/southwestmogrown/code_genie" rel="noopener noreferrer">Code Genie</a> reviewer looks like in its original form:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">reviewer</span> <span class="o">=</span> <span class="nc">Agent</span><span class="p">(</span>
    <span class="n">role</span><span class="o">=</span><span class="sh">"</span><span class="s">Senior Code Reviewer</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">goal</span><span class="o">=</span><span class="sh">"""</span><span class="s">Review code critically and objectively. Identify bugs, edge cases,
    convention violations, and anything that would fail in production.
    Do not give the benefit of the doubt.</span><span class="sh">"""</span><span class="p">,</span>
    <span class="n">backstory</span><span class="o">=</span><span class="sh">"""</span><span class="s">You are a senior engineer with high standards and low tolerance
    for sloppy code. You</span><span class="sh">'</span><span class="s">ve seen too many production incidents caused by code
    that looked fine in review. You are thorough, specific, and direct.</span><span class="sh">"""</span><span class="p">,</span>
    <span class="n">llm</span><span class="o">=</span><span class="n">llm</span><span class="p">,</span>
    <span class="n">verbose</span><span class="o">=</span><span class="bp">True</span>
<span class="p">)</span>

<span class="n">review_task</span> <span class="o">=</span> <span class="nc">Task</span><span class="p">(</span>
    <span class="n">description</span><span class="o">=</span><span class="sh">"""</span><span class="s">Review the following Python code for a crewAI agent workflow.
    Check for: correct use of crewAI Agent and Task APIs, proper error handling
    for LLM failures, retry logic that won</span><span class="sh">'</span><span class="s">t loop infinitely, and clear separation
    of agent responsibilities. Return PASS with brief justification, or FAIL with
    specific actionable feedback.</span><span class="sh">"""</span><span class="p">,</span>
    <span class="n">expected_output</span><span class="o">=</span><span class="sh">"</span><span class="s">PASS or FAIL with specific feedback</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">agent</span><span class="o">=</span><span class="n">reviewer</span>
<span class="p">)</span>
</code></pre>

</div>



<p>Look at the task description. It's reviewing Python code for a crewAI agent workflow. It's checking for correct use of crewAI APIs. It knows the retry logic is there and checks it specifically. Every line of that task is written for Code Genie and Code Genie only.</p>

<p>Drop this into a Next.js project and it'll review TypeScript like it's looking for crewAI API misuse. Drop it into a data pipeline and it'll flag missing retry logic that was never supposed to be there. The reviewer works — inside the project it was written for.</p>

<p>That's the cost of bespoke logic. It's not wrong. It's just not portable.</p>




<h2>
  
  
  What "Reusable" Actually Means
</h2>

<p>Before extracting anything, it helps to define what you're extracting toward. A reusable Skill needs to do three things:</p>

<p><strong>Work without project-specific assumptions baked in.</strong> The Skill can't know in advance what language you're using, what framework you're on, or what your conventions look like. That information has to come from outside — from CLAUDE.md, from the task prompt, from context passed in at runtime.</p>

<p><strong>Accept context from whatever project it's dropped into.</strong> The Skill should have explicit slots for project-specific information. Not hardcoded defaults — actual inputs it expects to receive and knows how to use.</p>

<p><strong>Produce output in a consistent format.</strong> Whatever calls the Skill — a crewAI agent, a GitHub Actions workflow, a human reading the result — should be able to rely on the output looking the same every time. PASS or FAIL, specific feedback, actionable items. No surprises.</p>

<p>That's the design spec. If the extracted Skill satisfies all three, it's genuinely reusable. If it satisfies two out of three, it's still bespoke with better formatting.</p>




<h2>
  
  
  Identifying What's Generic vs What's Project-Specific
</h2>

<p>The extraction exercise is simple in theory: go through the original reviewer and put everything into one of two buckets.</p>

<p><strong>Generic</strong> — applies to any code review, regardless of project:</p>

<ul>
<li>The reviewer persona (skeptical, specific, direct)</li>
<li>The output format (PASS or FAIL with actionable feedback)</li>
<li>The core review criteria (correctness, edge cases, readability)</li>
<li>The instruction not to give the benefit of the doubt</li>
</ul>

<p><strong>Project-specific</strong> — belongs in CLAUDE.md or the task prompt, not the Skill:</p>

<ul>
<li>The language and framework being reviewed</li>
<li>The specific APIs being checked</li>
<li>The architectural patterns to enforce</li>
<li>The conventions that count as violations</li>
</ul>

<p>Here's the same reviewer split into those two buckets:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># GENERIC — goes into the Skill
</span><span class="n">role</span><span class="o">=</span><span class="sh">"</span><span class="s">Senior Code Reviewer</span><span class="sh">"</span>
<span class="n">goal</span><span class="o">=</span><span class="sh">"</span><span class="s">Review code critically. Identify bugs, edge cases, and anything that would fail in production.</span><span class="sh">"</span>
<span class="n">backstory</span><span class="o">=</span><span class="sh">"</span><span class="s">You are a senior engineer with high standards. You are thorough, specific, and direct.</span><span class="sh">"</span>
<span class="n">output_format</span><span class="o">=</span><span class="sh">"</span><span class="s">PASS with brief justification, or FAIL with specific actionable feedback</span><span class="sh">"</span>

<span class="c1"># PROJECT-SPECIFIC — goes into the task prompt, sourced from CLAUDE.md
</span><span class="n">language</span><span class="o">=</span><span class="sh">"</span><span class="s">Python</span><span class="sh">"</span>
<span class="n">framework</span><span class="o">=</span><span class="sh">"</span><span class="s">crewAI</span><span class="sh">"</span>
<span class="n">specific_checks</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">crewAI Agent and Task API usage</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">retry logic</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">agent separation</span><span class="sh">"</span><span class="p">]</span>
<span class="n">conventions</span><span class="o">=</span><span class="sh">"</span><span class="s">[from CLAUDE.md]</span><span class="sh">"</span>
</code></pre>

</div>



<p>Once it's split this cleanly, the Skill writes itself.</p>




<h2>
  
  
  Writing the Skill
</h2>

<p>Here's the full <code>SKILL_code_review.md</code>, built from the generic bucket:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="nn">---</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">code-review</span>
<span class="na">description</span><span class="pi">:</span> <span class="s">Review code critically and return a structured PASS or FAIL with</span>
<span class="s">specific, actionable feedback. Use this skill whenever an agent or workflow needs</span>
<span class="s">a quality gate on generated or submitted code. Trigger when a task involves</span>
<span class="s">reviewing, evaluating, or checking code before it proceeds to the next step.</span>
<span class="nn">---</span>

<span class="gh"># Code Review Skill</span>

A reusable code review layer for agent workflows and manual review tasks.

<span class="gu">## Reviewer Persona</span>

You are a senior software engineer with high standards and low tolerance for
code that merely looks correct. You have seen too many production incidents
caused by plausible-looking code that was never properly scrutinized. You are
thorough, specific, and direct. You do not give the benefit of the doubt.

<span class="gu">## What You Always Check</span>

Regardless of language or framework, every review covers:
<span class="p">
1.</span> <span class="gs">**Correctness**</span> — Does the code do what it's supposed to do?
   Does it handle the happy path? Does it handle failure?
<span class="p">2.</span> <span class="gs">**Edge cases**</span> — What happens at the boundaries? Empty input,
   null values, unexpected types, off-by-one conditions.
<span class="p">3.</span> <span class="gs">**Readability**</span> — Can another developer understand this code
   without asking questions?
<span class="p">4.</span> <span class="gs">**Side effects**</span> — Does this code do anything it shouldn't?
   Modify state it doesn't own? Make calls it wasn't asked to make?

<span class="gu">## What You Check From Project Context</span>

The following criteria come from the project's CLAUDE.md and task prompt.
Apply them in addition to the universal criteria above:
<span class="p">
-</span> Language and framework conventions
<span class="p">-</span> Architectural patterns to enforce or avoid
<span class="p">-</span> Specific APIs or libraries in use
<span class="p">-</span> Anti-patterns explicitly called out in CLAUDE.md

<span class="gu">## Output Format</span>

Return exactly one of the following:

<span class="gs">**PASS**</span>
Brief justification (1-2 sentences). What makes this code acceptable.

<span class="gs">**FAIL**</span>
Specific feedback only. For each issue:
<span class="p">-</span> What the problem is
<span class="p">-</span> Where it is (function name, line reference, or code snippet)
<span class="p">-</span> What the fix should be

Do not return vague feedback. "This could be improved" is not actionable.
"The retry loop on line 24 has no exit condition — add a max_retries check
before the recursive call" is actionable.
</code></pre>

</div>



<p>That's the complete Skill. Generic enough to work anywhere, structured enough to produce consistent output, with explicit slots for project context to plug into.</p>




<h2>
  
  
  Plugging It Back Into Code Genie
</h2>

<p>With the Skill written, the original Code Genie reviewer slims down considerably. The agent definition drops to its essentials and delegates to the Skill. The project-specific criteria move into the task prompt where they belong:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">reviewer</span> <span class="o">=</span> <span class="nc">Agent</span><span class="p">(</span>
    <span class="n">role</span><span class="o">=</span><span class="sh">"</span><span class="s">Senior Code Reviewer</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">goal</span><span class="o">=</span><span class="sh">"</span><span class="s">Apply the code-review Skill to evaluate the submitted code.</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">backstory</span><span class="o">=</span><span class="sh">"</span><span class="s">You follow the code-review Skill process precisely.</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">llm</span><span class="o">=</span><span class="n">llm</span><span class="p">,</span>
    <span class="n">verbose</span><span class="o">=</span><span class="bp">True</span>
<span class="p">)</span>

<span class="n">review_task</span> <span class="o">=</span> <span class="nc">Task</span><span class="p">(</span>
    <span class="n">description</span><span class="o">=</span><span class="sh">"""</span><span class="s">Using the code-review Skill, review the submitted Python code.

    Project context (from CLAUDE.md):
    - Language: Python 3.11
    - Framework: crewAI
    - Conventions: agents have single responsibilities, tasks return structured output
    - Anti-patterns: no raw exception swallowing, no infinite retry loops

    Apply both the universal criteria from the Skill and the project context above.
    Return PASS or FAIL with specific feedback.</span><span class="sh">"""</span><span class="p">,</span>
    <span class="n">expected_output</span><span class="o">=</span><span class="sh">"</span><span class="s">PASS or FAIL with specific feedback per the code-review Skill format</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">agent</span><span class="o">=</span><span class="n">reviewer</span>
<span class="p">)</span>
</code></pre>

</div>



<p>The loop still works exactly as before. The output format is identical. But the review layer is now a dependency rather than a hardcoded implementation — and that distinction matters the next time you need a quality gate somewhere else.</p>




<h2>
  
  
  Plugging It Into a Different Project
</h2>

<p>Here's the same Skill wired into a completely different context — a Next.js TypeScript project with no connection to crewAI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">review_task</span> <span class="o">=</span> <span class="nc">Task</span><span class="p">(</span>
    <span class="n">description</span><span class="o">=</span><span class="sh">"""</span><span class="s">Using the code-review Skill, review the submitted TypeScript code.

    Project context (from CLAUDE.md):
    - Language: TypeScript strict
    - Framework: Next.js 16, React 19
    - Conventions: named exports, Result pattern for error handling,
      no raw throws in business logic
    - Anti-patterns: no default exports, no useEffect for data fetching,
      do not mutate props

    Apply both the universal criteria from the Skill and the project context above.
    Return PASS or FAIL with specific feedback.</span><span class="sh">"""</span><span class="p">,</span>
    <span class="n">expected_output</span><span class="o">=</span><span class="sh">"</span><span class="s">PASS or FAIL with specific feedback per the code-review Skill format</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">agent</span><span class="o">=</span><span class="n">reviewer</span>
<span class="p">)</span>
</code></pre>

</div>



<p>Same Skill. Different language, different framework, different conventions. The Skill adapts because the project-specific context is coming from the task prompt — sourced from CLAUDE.md — not baked into the Skill itself. The reviewer persona, the universal criteria, the output format are all unchanged.</p>

<p>That's portability in practice.</p>




<h2>
  
  
  When to Extract vs When to Leave It Bespoke
</h2>

<p>Not everything should be a Skill. The extraction is worth the effort when all three of the following are true:</p>

<p><strong>The logic is genuinely reusable.</strong> If you can't imagine using it in at least two different projects, it probably belongs where it is. Extraction for its own sake adds maintenance overhead without payoff.</p>

<p><strong>The project-specific parts can be cleanly separated.</strong> If you spend more time trying to untangle what's generic from what's specific than you would just rewriting it for the next project, leave it bespoke. The extraction exercise should feel like sorting, not surgery.</p>

<p><strong>You'll actually use it again.</strong> This one sounds obvious but it's easy to miss. Building a reusable Skill you never reach for is just premature abstraction with extra steps.</p>

<p>If all three are true, extract it. If one or two are true, extract the concept but not necessarily the implementation — document what worked and use it as a reference when the next similar problem comes up.</p>

<p>The code review Skill cleared all three bars easily. The reviewer persona and output format are the same everywhere. The project-specific criteria separated cleanly into the task prompt. And a quality gate on generated code is useful in every project that generates code.</p>




<h2>
  
  
  What's Next
</h2>

<p>The code review Skill is now in the <code>[ai-workflow-toolkit](https://github.com/southwestmogrown/ai-workflow-toolkit)</code> repo alongside the CLAUDE.md generator and the README writer. Three skills, one repo, all portable.</p>

<p>The pattern behind all three is the same: build it bespoke first, extract what's generic, make it portable. Resist the urge to abstract before you've built something real — you can't know what's generic until you've seen the specific version work.</p>

<p>Next up: how I'm using these building blocks to wire up a full agent workflow on GitHub issues — from writing the issue to merging the PR, with Claude handling every step in between.</p>




<p><em>Tags: python, ai, crewai, webdev</em><br>
<em>Series: Building With AI Agents — Article 3 of 12</em></p>

