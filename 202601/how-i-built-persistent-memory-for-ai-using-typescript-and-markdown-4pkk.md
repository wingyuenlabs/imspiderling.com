---
Title: How I Built Persistent Memory for AI Using TypeScript and Markdown
Description: 
Author: Rafhael Marsigli
Date: 2026-01-23T21:35:27.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>Cross-platform. TypeScript strict mode. 30 E2E tests. Multi-OS CI. Independent technical review. Built in <strong>less than 24 hours</strong>, with AI.</p>

<p><strong>This is the story of the build.</strong> What happened in the following 30 days is another story.</p>
</blockquote>

<h2>
  
  
  The pain that woke me up in the morning
</h2>

<p>If you use Claude, Codex, or any AI assistant for programming, you’ve lived this. Every session starts the same way: paste context, repeat rules, reinforce decisions already made, warn about what cannot be done, explain the project’s patterns one more time. No matter how good the AI is, it always starts from zero.</p>

<p><em>“Ah, but just keep a CLAUDE.md, GEMINI.md, etc.”</em> Yes, you can do that. But what can a single file really represent in a living project, with history, accumulated decisions, and current state? In the end, it’s like working with an extremely capable developer, but with a <strong>goldfish memory</strong>.</p>

<p>I had already been trying - and managing, with a lot of manual work — to solve this for months using hand-written Markdown. It worked, but it was rough. Every new project felt like starting from scratch, as if I were carving important decisions into stone every time.</p>

<p>The problem was never the AI. It was the lack of <strong>persistent memory</strong>, <strong>real history</strong>, and decisions that don’t get lost between sessions.</p>

<h2>
  
  
  The idea: external memory for AI
</h2>

<p>After shaping and organizing the ideas in my head for a while, at 8 a.m. I woke up decided — it’s like those days when you wake up possessed by the urge to clean the house — everything was clear enough to turn into code.</p>

<p>That’s how <strong>AIPIM - AI Project Instruction Manager</strong> was born. It used to be AI Project Manager, then AIPM, then… through a random but valuable insight, AIPIM. Not as hype, but as a practical tool. <strong>The proposal is simple:</strong> create an external memory layer between the project and the AI assistant.</p>

<p>This layer solves two core pains: keeping persistent context between sessions and recording the real project history, including decisions, tasks, and current state. All of this is organized in a <code>.project/</code> structure, entirely in Markdown, which the AI can read, respect, and follow.</p>

<ul>
<li>
<code>context.md</code> becomes the source of truth</li>
<li>
<code>current-task.md</code> defines the focus</li>
<li>
<code>decisions/</code> stores immutable architectural decisions</li>
<li>
<code>backlog/</code> with <code>completed/</code> records work honestly</li>
</ul>

<p>I was already using exactly this methodology before AIPIM existed. The difference is that now it became an <strong>automated CLI</strong>.</p>

<h2>
  
  
  How I worked with AI (for real)
</h2>

<p>I didn’t use passive autocomplete. I used <strong>Claude Code</strong> as a pair programmer. I defined the direction, the AI executed, I reviewed everything, fixed a lot of things, and nothing passed without full understanding. There was no “accept suggestion.” There was technical dialogue.</p>

<p><strong>A real example:</strong> I asked for a framework detector. The AI proposed heuristics using <code>package.json</code> and the filesystem. I pointed out edge cases, like React with Vite versus CRA, and demanded hierarchical fallback. The flow was continuous, no context switching, always with human decisions in command.</p>

<h2>
  
  
  By the afternoon, the first <code>aipim install</code> worked
</h2>

<p>Near the end of the afternoon, the first milestone was standing: a functional CLI. It wasn’t magic. There were bugs, tweaks (my goodness, many of them — if Claude had ears, they’d be hurting), and review. AI leaves traces, especially in complex tasks, and someone needs to follow those traces carefully. From the start, I made it clear that the project needed to be cross-platform, with real error handling like <code>EACCES</code> and <code>ENOSPC</code>, working on Windows, macOS, and Linux.</p>

<h2>
  
  
  Updates without overwriting customization
</h2>

<p>This was a point where AI alone wouldn’t solve it. The problem was updating base templates without destroying user customizations. The obvious suggestion was to use <code>git diff</code>. My answer was simple: what if there’s no git? What if the repository is dirty?</p>

<p>The solution was architectural. Each generated file receives a cryptographic SHA-256 signature embedded as a comment. If the hash matches, the file is pristine and can be updated. If it diverges, it was modified and must be preserved. Claude implemented it perfectly, but the decision was human. This point, by the way, was one of the most praised in the technical review, compared to strategies used in Kubernetes manifests.</p>

<h2>
  
  
  Tests: where AI really shined
</h2>

<p>I don’t like writing tests <em>(although I do write them)</em>, and here the AI was absurdly good. I asked for unit tests for the detector and, in minutes, there were 17 cases covering all frameworks. Then I asked for more: 30 cross-platform E2E scenarios. Tests came for installation, update, validation, and task lifecycle, running on Linux, Windows, and macOS in CI. That would have been days of manual work.</p>

<p>It took just a few hours.</p>

<h2>
  
  
  <code>TypeScript Strict</code>, no compromises
</h2>

<p>Here I was picky, but I’d be with myself anyway. Strict mode, zero <code>any</code>. The rule was explicit and the AI respected it. In the technical review, the type safety score was 95/100.</p>

<h2>
  
  
  The Brutal and Honest Feedback
</h2>

<p>The strengths were clear: clean architecture, the SHA-256 signature system, 30 cross-platform E2E scenarios, and strict TypeScript. The red flags were fair too: a <code>diff</code> command announced but not implemented, a dependency installed and not used, and too few tests in the updater, a critical module.</p>

<p>The final score was 88/100.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzxtyui6n9oyeww6e58nm.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzxtyui6n9oyeww6e58nm.webp" alt="Basic flow of using AIPIM" width="800" height="446"></a></p>

<h2>
  
  
  What I Learned in These First 24 Hours
</h2>

<p>AI doesn’t replace judgment; it executes. Tests stopped being an excuse; in 2026, not having tests is a choice. And speed doesn’t have to kill quality when there’s process.</p>

<h2>
  
  
  Numbers, 24 Hours Later
</h2>

<p>There were about 2,000 lines of core code and 2,000 lines of tests, 30 E2E scenarios, strict TypeScript without <code>any</code>. Without AI, this would have taken one to two weeks. With AI, it took about 24 real hours — including a trip to the mall with my family in the middle of the day. The project was functional, tested, and published to npm. Exactly as I wanted: an evolved version of my old Jurassic Markdown manager.</p>

<h2>
  
  
  What Came Next
</h2>

<p>In the following days, I used AIPIM to manage AIPIM itself:</p>

<ul>
<li>I documented nine sessions</li>
<li>Completed 31 tasks</li>
<li>Wrote eight ADRs</li>
<li>Fixed all 16 red flags</li>
<li>Raised the score from 88 to 90</li>
</ul>

<p>But that’s for a next post, because this one has already crossed the healthy longform limit (even though I enjoy it).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbswjb427zo238jrb68y8.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbswjb427zo238jrb68y8.webp" alt="Installing AIPIM in your project" width="800" height="446"></a></p>

<h2>
  
  
  For Those Who Want to Go Further
</h2>

<p>The complete technical report is available in the <a href="https://github.com/rmarsigli/aipim/blob/main/.project/reports/first-code-quality-report-2026-01-08.md" rel="noopener noreferrer">repository</a>, generated with Claude Code (Sonnet 4.5), and can be read in full. AIPIM is already on <a href="https://npmjs.com/aipim" rel="noopener noreferrer">npm</a> and on <a href="https://github.com/rmarsigli/aipim" rel="noopener noreferrer">github</a>.</p>

<p>Usage is simple and explained in the <code>README</code>.</p>

