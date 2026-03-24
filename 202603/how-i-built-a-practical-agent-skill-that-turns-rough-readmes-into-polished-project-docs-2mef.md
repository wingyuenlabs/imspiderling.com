---
Title: How I built a practical agent skill that turns rough READMEs into polished project docs
Description: 
Author: Debbie O'Brien
Date: 2026-03-24T21:44:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you're new to agent skills, start with my beginner guide first:</p>

<p><a href="https://dev.to/debs_obrien/what-are-agent-skills-beginners-guide-e2n">What Are Agent Skills? Beginners Guide</a></p>

<p>That post covers what skills are, how they get loaded, and how to build a tiny one from scratch.</p>

<p>This post picks up where that one stops.</p>

<p>Instead of another tiny example, I want to show you what a practical skill looks like when it solves a real problem.</p>

<p>We are going to take the idea of a skill and use it to turn rough project READMEs into polished docs that are consistent, accurate, and reusable across repos. I picked README generation because the output is easy to judge, it comes up again and again, and once you get it right for one project you want the same quality bar everywhere.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3vdwqr7y2i1m5t5vki8r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3vdwqr7y2i1m5t5vki8r.png" alt="Before vs After"></a></p>

<h2>
  
  
  The problem with one-off README prompts
</h2>

<p>You can absolutely ask an agent to improve your README and get something decent back.</p>

<p>Sometimes it will even be very good.</p>

<p>But if you do that across multiple projects, the cracks show up quickly:</p>

<ul>
<li>badge styles are inconsistent</li>
<li>section order changes from repo to repo</li>
<li>install commands drift away from the actual package manager</li>
<li>social links get guessed</li>
<li>simple projects end up with bloated READMEs</li>
<li>the agent repeats the same repo-scanning work every time</li>
</ul>

<p>That is exactly the kind of problem skills are good at solving.</p>

<p>Not because they magically make the model smarter, but because they turn a vague prompt into a reusable workflow.</p>

<h2>
  
  
  The first version was just one file
</h2>

<p>I did not start with a big architecture.</p>

<p>The first version of <code>readme-wizard</code> was just a single <code>SKILL.md</code> with instructions telling the agent to:</p>

<ul>
<li>detect the project name, description, license, git remote, package manager, and CI setup</li>
<li>add a better structure to the README</li>
<li>use shields.io badges</li>
<li>include a Quick Start section with real commands</li>
<li>show a project structure tree</li>
<li>add contributor avatars, documentation links, and optional social badges</li>
</ul>

<p>That first version worked.</p>

<p>And that matters.</p>

<p>One of the easiest mistakes to make with agent workflows is over-engineering too early. A single file is often enough to prove whether the workflow is useful before you invest more time into it.</p>

<p>Here is the important part: start with the smallest thing that can produce a useful result on a real project.</p>

<h2>
  
  
  What broke in practice
</h2>

<p>Once I started testing the skill on real repos, the limitations showed up quickly.</p>

<p>The main issue was not that the agent could not write a README. It could.</p>

<p>The issue was consistency.</p>

<p>The single-file version was asking the <code>SKILL.md</code> to do too many jobs at once:</p>

<ul>
<li>writing guidance</li>
<li>badge formats</li>
<li>project-type adaptation rules</li>
<li>README structure templates</li>
<li>Mermaid diagram templates</li>
<li>instructions for how to detect project metadata</li>
</ul>

<p>That creates a few problems.</p>

<p>First, the file gets bloated fast. By the time I had all those rules and templates inline, it was over 150 lines and hard to maintain.</p>

<p>Second, the agent had to figure out how to inspect the repo on every single run. There was no scanning script yet — just instructions saying "detect the package manager, find the license, parse the git remote." The agent would improvise that detection work each time. Sometimes it got it right. Sometimes it missed a CI workflow file, guessed at the wrong package manager, or invented social links that did not exist.</p>

<p>Third, all of that detection reasoning burned tokens and produced inconsistent results. The kind of work that should be boring and repeatable was instead fuzzy and error-prone.</p>

<h2>
  
  
  The turning point: treat the skill like a workflow, not a prompt
</h2>

<p>That was the point where the skill stopped being just a better prompt and started becoming a real workflow.</p>

<p>The structure ended up looking like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.agents/skills/readme-wizard/
├── SKILL.md
├── scripts/
│   └── scan_project.sh
├── references/
│   └── readme-best-practices.md
├── assets/
│   ├── badges.json
│   ├── diagrams.md
│   └── readme-template.md
└── evals/
    └── evals.json
</code></pre>

</div>



<p>Every part has a different job. And that is the point.</p>

<h2>
  
  
  <code>SKILL.md</code> became the orchestrator
</h2>

<p>Instead of being one giant wall of instructions, <code>SKILL.md</code> became the thin coordinator.</p>

<p>Its job is to define the workflow:</p>

<ol>
<li>run the scan script</li>
<li>read the README best-practices guide</li>
<li>build from the template</li>
<li>pull badge formats from the badge catalog</li>
<li>validate against the eval assertions</li>
<li>only load diagram templates if the project actually needs them</li>
</ol>

<p>That is a much better use of the main skill file.</p>

<p>It keeps the top-level instructions focused on sequence and judgment instead of burying everything in one place.</p>

<p>Here is what the workflow section of the final <code>SKILL.md</code> looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Workflow</span>

<span class="gu">### 1. Scan the project</span>
Run <span class="sb">`scripts/scan_project.sh &lt;project-directory&gt;`</span> to collect structured JSON metadata.

<span class="gu">### 2. Read the best practices guide</span>
Read <span class="sb">`references/readme-best-practices.md`</span> before writing.

<span class="gu">### 3. Build the README</span>
Use <span class="sb">`assets/readme-template.md`</span> as the base structure.
Replace {{PLACEHOLDER}} markers with actual project data from the scan.

<span class="gu">### 4. Add badges</span>
Read <span class="sb">`assets/badges.json`</span> for the full badge catalog.
Only include badges for things that actually exist.

<span class="gu">### 5. Validate the output</span>
Review the generated README against the assertions in <span class="sb">`evals/evals.json`</span>.

<span class="gu">### 6. Optionally add a diagram</span>
Only read <span class="sb">`assets/diagrams.md`</span> if the project has multiple components.
</code></pre>

</div>



<p>Short, focused, and easy to follow. Each step points to another file instead of trying to carry everything inline.</p>

<h2>
  
  
  The script handled the mechanical work
</h2>

<p>The biggest improvement was moving repo scanning into a script.</p>

<p>The skill now runs <code>scripts/scan_project.sh &lt;project-directory&gt;</code> and gets structured JSON back with things like:</p>

<ul>
<li>project name</li>
<li>description</li>
<li>license</li>
<li>owner and repo</li>
<li>package manager</li>
<li>CI provider and workflows</li>
<li>social links</li>
<li>directory structure</li>
</ul>

<p>Instead of the agent improvising that detection work every time, it runs one script and gets clean, structured data back. Boring and repeatable. Exactly what you want for metadata gathering.</p>

<p>The current reference version also goes a bit further. It checks local files first, then uses the GitHub API to look up the repo homepage and crawls it for additional social links. That is a good example of how a skill can evolve — start with the reliable local-file path, then add enrichment once the core workflow is stable.</p>

<h2>
  
  
  References and assets gave everything a home
</h2>

<p>The remaining pieces fell into two folders.</p>

<p><code>references/readme-best-practices.md</code> holds the writing guidance: section order, tone, project-type adaptation, badge rules, and common pitfalls. The agent only reads it when it is about to write, not every time the skill loads.</p>

<p><code>assets/</code> holds reusable inputs: <code>badges.json</code> for badge formats, <code>readme-template.md</code> for the base README structure, and <code>diagrams.md</code> for Mermaid templates when a project is complex enough to justify one.</p>

<p>This is where the skill becomes easy to customize. Want to change badge styles? Edit the badge catalog. Want a different README structure? Edit the template. Want to skip diagrams for simpler repos? The skill just avoids loading that asset entirely.</p>

<p>Keeping domain knowledge and data out of the main instructions makes the whole thing much easier to maintain.</p>

<h2>
  
  
  Evals made the quality bar explicit
</h2>

<p>Once the skill was doing real work, I wanted a way to define what good actually meant.</p>

<p>That is what the evals are for.</p>

<p>The <code>evals/evals.json</code> file includes prompts for different cases:</p>

<ul>
<li>a straightforward README improvement request</li>
<li>a casual "make this look professional" request</li>
<li>a minimal project that should not get bloated</li>
<li>a badge-focused request that should only generate real badges</li>
</ul>

<p>I like this part because it forces the standards out into the open.</p>

<p>Instead of vaguely feeling that the README is better, you can check for specific things:</p>

<ul>
<li>no placeholder text</li>
<li>badges only for real metadata</li>
<li>Quick Start commands that match the detected package manager</li>
<li>section depth proportional to the project</li>
<li>no fabricated social links</li>
</ul>

<p>That makes the skill easier to improve without drifting.</p>

<h2>
  
  
  The larger lesson
</h2>

<p>The interesting thing about this project is not really README generation.</p>

<p>The larger lesson is that a useful skill usually stops looking like a prompt pretty quickly.</p>

<p>It becomes a small system.</p>

<p>Some parts should stay flexible and language-driven.</p>

<p>Some parts should be deterministic.</p>

<p>Some parts should be reusable data.</p>

<p>Some parts should act as tests.</p>

<p>Once you see that pattern, it applies to a lot more than READMEs:</p>

<ul>
<li>commit message workflows</li>
<li>code review checklists</li>
<li>release note generation</li>
<li>internal documentation standards</li>
<li>repo audits</li>
<li>team-specific engineering conventions</li>
</ul>

<p>That is the shift I find most useful when working with agents.</p>

<p>You stop asking the model to improvise the whole workflow every time.</p>

<p>Instead, you give it a structure that makes good behavior easier.</p>

<h2>
  
  
  If you want to build your own skill
</h2>

<p>If you want to build your own skill, this is the path I would recommend:</p>

<ol>
<li>Start with one <code>SKILL.md</code>.</li>
<li>Test it on a real project as early as possible.</li>
<li>Watch for repeated logic and consistency failures.</li>
<li>Move mechanical work into scripts.</li>
<li>Move domain knowledge into references.</li>
<li>Move templates and data into assets.</li>
<li>Add evals once the skill matters enough to maintain.</li>
</ol>

<p>That sequence keeps the architecture earned.</p>

<p>You are not building a folder structure for its own sake. You are extracting parts only when they prove they deserve to exist.</p>

<h2>
  
  
  Try it yourself
</h2>

<p>If you want to explore the full tutorial series or inspect the finished reference implementation, the repo is here:</p>

<p><a href="https://github.com/debs-obrien/learn-agent-skills" rel="noopener noreferrer">debs-obrien/learn-agent-skills</a></p>

<p>And if you just want to try the skill without building it yourself:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx skills add debs-obrien/readme-wizard
</code></pre>

</div>



<p>Then open any project and tell your agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Improve the README for this project using the readme-wizard skill.
</code></pre>

</div>



<p>The point is not just that a skill can write a better README.</p>

<p>The point is how you get from a useful first draft to something reusable.</p>

