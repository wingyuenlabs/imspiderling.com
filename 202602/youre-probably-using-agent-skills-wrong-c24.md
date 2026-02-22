---
Title: You're probably using Agent Skills wrong
Description: 
Author: Anson Biggs
Date: 2026-02-22T21:36:13.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1685773253905-915ce8dd470b%3Fcrop%3Dentropy%26cs%3Dtinysrgb%26fit%3Dmax%26fm%3Djpg%26ixid%3DM3wxMTc3M3wwfDF8c2VhcmNofDEzNnx8YmVlaGl2ZXxlbnwwfHx8fDE3NzEzMDEyMjd8MA%26ixlib%3Drb-4.1.0%26q%3D80%26w%3D2000" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fimages.unsplash.com%2Fphoto-1685773253905-915ce8dd470b%3Fcrop%3Dentropy%26cs%3Dtinysrgb%26fit%3Dmax%26fm%3Djpg%26ixid%3DM3wxMTc3M3wwfDF8c2VhcmNofDEzNnx8YmVlaGl2ZXxlbnwwfHx8fDE3NzEzMDEyMjd8MA%26ixlib%3Drb-4.1.0%26q%3D80%26w%3D2000" alt="You're probably using Agent Skills wrong" width="2000" height="1502"></a></p>

<p>The entire ecosystem around Claude Code is pretty confusing, the naming conventions are a mess and the pace of change is beyond any production tool I've seen. However Skills are probably the most misused. I see it at work at ton but a paper just came up on Hacker News:</p>

<p>[SkillsBench: Benchmarking How Well Agent Skills Work Across Diverse Tasks</p>

<p>Agent Skills are structured packages of procedural knowledge that augment LLM agents at inference time. Despite rapid adoption, there is no standard way to measure whether they actually help. We present SkillsBench, a benchmark of 86 tasks across 11 domains paired with curated Skills and deterministic verifiers. Each task is evaluated under three conditions: no Skills, curated Skills, and self-generated Skills. We test 7 agent-model configurations over 7,308 trajectories. Curated Skills raise average pass rate by 16.2 percentage points(pp), but effects vary widely by domain (+4.5pp for Software Engineering to +51.9pp for Healthcare) and 16 of 84 tasks show negative deltas. Self-generated Skills provide no benefit on average, showing that models cannot reliably author the procedural knowledge they benefit from consuming. Focused Skills with 2--3 modules outperform comprehensive documentation, and smaller models with Skills can match larger models without them.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmyx2pxu6r8j44yk8jav3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmyx2pxu6r8j44yk8jav3.png" width="180" height="180"></a>arXiv.orgXiangyi Li</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgab2fqciqzy8h1zvbird.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgab2fqciqzy8h1zvbird.png" width="800" height="466"></a>](<a href="https://arxiv.org/abs/2602.12670" rel="noopener noreferrer">https://arxiv.org/abs/2602.12670</a>)</p>

<p>The HN title is editorialized for some reason <em>"Study: Self-generated Agent Skills are useless"</em>, but it immediately grabbed me since I get massive value from Skills written by Agents, but I also consistently see them misused by my peers. The concept is great, I've been looking at benchmarking specific parts of the Agentic ecosystem myself so this was highly relevant to me. Overall the paper is decent but one bullet invalidates the whole thing:</p>

<blockquote>
<p><strong>Self-Generated Skills</strong>: No Skills provided, but the agent is prompted to generate relevant procedural knowledge before solving the task. This isolates the impact of LLMs’ latent domain knowledge.</p>
</blockquote>

<p>So all they are doing is taking a problem that a model can't solve well on its own, and asking it to write about the task before attempting it. They just reinvented thinking blocks but worse!</p>

<h3>
  
  
  The Skill Anti-Pattern
</h3>

<p>What they did is a very common mistake that I see constantly. My Agent it bad at this thing so I ask the Agent to write a skill on this thing. I'll reiterate this is identical to thinking blocks. In order for your Agent to create something worthwhile you have to make sure they can see the gaps. I see this as the classic CS intro where you ask someone to write out the steps to make a PB&amp;J, you don't really understand what makes the problem hard until you've struggled through solving it.</p>

<p>This directly leads into the largest Faux Pas of the AI era, just asking a LLM someone elses question verbatim, and pasting the LLMs answer as your response. If I ask you how you did something cool with an Agent, and you just on the fly have a fresh Agent build me a SKILL.md on my question, <em>I will kill you.</em></p>

<h2>
  
  
  What are Skills
</h2>

<p>Before getting into proper usage, I just want to cover what skills are. As a primitive they are just markdown files that have some metadata at the top to help Agents/Tools know when to use them, and then the rest of the document is the skill. Each skill has its own folder so it can no only teach your Agent how to do something but also give it better tools.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.claude/skills/
└── monitor-gitlab-ci/
    ├── SKILL.md # The file metioned above
    ├── monitor_ci.sh # Complicated command
    └── references/ # Additional references 
        ├── api_commands.md
        ├── log_analysis.md
        └── troubleshooting.md
</code></pre>

</div>



<p>Above is a Skill I used a ton to let older versions of Claude work on my GitLab CI. It's a folder with a simple markdown Skill that just explained the setup and that the Agent needs to watch the CI until either a job fails or everything passes, a simple CLI to prevent the Agent from writing a script, and additional references for edge cases.</p>

<h2>
  
  
  Skills for Context
</h2>

<p>Agents are completely stateless meaning that every new conversation is like meeting the model for the first time, it has no idea what your project is or what you were working on 10 minutes ago. CLAUDE.md does a lot to fix this, but for a large enough project it can't contain everything. If I open up a monorepo and tell Claude to run a SIL test then it is going to have to run around to figure out how to do that. It has to figure out what language the project is in, then look for common test patterns for that language, its going to see a complicated Docker Compose setup, its going to see that the containers need x86 but we're running on a Mac, then its going to look for CI, etc.</p>

<p>This can all be solved by writing Skills for common, but not universal patterns. A good habit is having Claude explain bespoke parts of your project as a Skill while you implement.</p>

<h2>
  
  
  Skills for Hard Problems
</h2>

<p>Claude can solve some really hard problems, but it might take $500 in tokens and you might have to yell at it for reward hacking a few times. Almost any time I have to intervene on a problem, once the Agent it unstuck I ask it what the gap was that kept it from figuring it out on its own. Sometimes it something silly, but sometimes it is something genuinely insightful and I have Claude make a Skill to fill the gap.</p>

<h2>
  
  
  Conclusion
</h2>

<p>I edited the original benchmark to do Skills my way and the results were incredible, the Agents nailed the test with proper Skills. I don't have the money to spend on fully validating this result but the first pass was good enough for me to be happy. I assume this changes the benchmark enough that it was too hard to do properly so the Authors opted to do it wrong.</p>

<p>Make sure when you create a Skill the Agent knows something that the base model doesn't. This can even happen during the creating of the Skill, but the Agent must have the oppurtunity to understand the issue it is solving before it can solve it better than a base model.</p>

<p>Happy Hacking.</p>

