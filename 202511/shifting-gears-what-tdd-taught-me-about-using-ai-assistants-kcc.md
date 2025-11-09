---
Title: Shifting Gears: What TDD Taught Me About Using AI Assistants
Description: 
Author: Treveshan Naidoo
Date: 2025-11-09T21:52:34.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’ve ever studied the TDD Gears model from <a href="https://www.tddbuddy.com/references/tdd-gears.html" rel="noopener noreferrer">TDD Buddy</a> , you’ll know it’s one of the clearest ways to explain how our thinking evolves when practicing Test-Driven Development. Each gear represents a different scale of thought and a level of strictness in applying practices, while the underlying principles remain constant.</p>

<p>In recent months, I’ve found myself drawing a strong parallel between the <strong>TDD Gears model</strong> and how we use <strong>AI assistants</strong> such as ChatGPT, GitHub Copilot, or Claude. The way we “shift gears” when coding, designing, or debugging feels remarkably similar to how we move through the gears of TDD. In both, the key is to maintain discipline, reflection, and professionalism regardless of speed.</p>

<p>This article maps the TDD Gears to AI Assistant Gears showing how we can approach AI as a thinking partner, not an autopilot.</p>

<p><strong>Understanding TDD Gears</strong><br>
In TDD, each gear represents both a degree of adherence to core practices and a scale of thought. As we shift gears, we adjust how rigidly we apply certain practices, but the principles of professionalism, maintainability, and quality never change.</p>

<ul>
<li>
<strong>Low Gear</strong>: Used when we’re unsure of the domain or algorithm. We move slowly, deliberately, and follow every core practice strictly. The goal is to build context and understanding.</li>
<li>
<strong>Medium Gear</strong>: Applied when we’ve gained some familiarity. We begin to refine our design and can short-circuit some rules, mixing core and advanced practices.</li>
<li>
<strong>High Gear</strong>: Reserved for when we’re confident. We work on larger problems or follow existing patterns. We take more risks because we’ve earned that confidence through lower gears.</li>
<li>
<strong>Reverse Gear</strong>: Used when stuck. We back out of the problem, return to a green state, and restart in low gear.</li>
</ul>

<p>This model teaches us that <strong>mastery isn’t about speed, it’s about control.</strong> The same applies to using AI.</p>

<p><strong>The AI Assistant Gears Model</strong><br>
AI assistants can dramatically accelerate our work, but only if used thoughtfully. Without structure, it’s easy to fall into the trap of asking AI to “do everything”, skipping understanding entirely. To use AI responsibly and effectively, we can think of our interaction with it as shifting through four gears.</p>

<p><strong>Low Gear: Exploring with AI</strong><br>
When entering a new domain, framework, or problem space, we start in low gear. In this mode, AI acts as a <strong>teaching assistant</strong> rather than a code generator.</p>

<p>We focus on small, incremental learning:</p>

<ul>
<li>Ask focused, narrow questions (“What does this error mean?” or “Show a minimal example of configuring HttpClientFactory”).</li>
<li>Validate all answers against official documentation or trusted sources.</li>
<li>Avoid copy-pasting code into production without fully understanding it.</li>
<li>Write tests or small experiments to verify correctness.</li>
</ul>

<p>In this gear, the goal is to <strong>build context and understanding</strong>, not to ship code. Just as low gear in TDD helps us form accurate mental models through micro-tests, low gear with AI helps us build accurate mental models through micro-prompts.</p>

<p><strong>Medium Gear: Designing with AI</strong><br>
Once you have a basic understanding of the domain, you can shift into medium gear. This is where AI becomes a design partner rather than just an explainer.</p>

<p>Here, we begin using AI for reasoning about structure and elegance:</p>

<ul>
<li>Request design alternatives or architecture suggestions.</li>
<li>Ask it to propose test cases, error handling, or interface designs.</li>
<li>Use it to review or refactor code for clarity and maintainability.</li>
<li>Compare trade-offs between technologies or design patterns.</li>
</ul>

<p>You might allow AI to generate slightly larger pieces of code, but every suggestion still passes through your professional judgment. You review, rewrite, and reason through it as though it came from a junior developer.</p>

<p>In TDD, medium gear is where we build elegance through deliberate design. With AI, medium gear is where we amplify that design through guided collaboration, without surrendering ownership.</p>

<p><strong>High Gear: Scaling with AI</strong><br>
High gear is for those moments when you are fully comfortable with both the domain and your assistant. You understand its strengths, its blind spots, and its limitations. You also have solid guardrails comprehensive tests, static analysis, and peer review.</p>

<ul>
<li>In high gear, you can use AI to scale your efforts responsibly:</li>
<li>Draft large feature scaffolds or API endpoints.</li>
<li>Generate documentation or consistent code patterns across services.</li>
<li>Automate repetitive refactors using verified patterns.</li>
<li>Combine human judgment with AI speed to tackle broader changes.</li>
</ul>

<p>You may allow AI to take on more of the work, but your principles remain unchanged. You still review every significant output, validate assumptions, and maintain accountability for quality. As with TDD, the higher gears don’t remove the need for discipline they only expand the scale of what can be safely achieved.</p>

<p><strong>Reverse Gear: Getting Unstuck</strong><br>
Every developer reaches a point where things stop making sense. Prompts yield worse results, refactors become tangled, or code starts breaking in unexpected ways. This is when you shift into reverse.</p>

<p>Reverse gear means backing out safely and regaining clarity.</p>

<ul>
<li>
<strong>Use source control</strong>. Every AI-driven change should exist in a branch or commit you can easily revert. Source control is your safety net when experimentation fails.</li>
<li>Return to a** known-good green build** and start again from a stable point.</li>
<li>Reduce scope. Create a minimal example that reproduces the problem and focus your AI prompts on that smaller context.</li>
<li>Change your angle of attack ask “What’s the smallest change to fix this test?” instead of “Fix the whole class.”</li>
<li>Use “learning prompts” to explore misunderstood areas (e.g., “Show a minimal working example of this API”).</li>
<li>If the conversation context is polluted, start a fresh session.</li>
</ul>

<p>TDD has a rule of thumb called <strong>red/red/reverse</strong> if you’ve made two failed attempts to fix a red test, back out and try again from green. The same rule applies here: after two failed attempts or confusing AI outputs, reverse, clean up, and restart smaller.</p>

<p><strong>The Principles That Never Change</strong><br>
In both TDD and AI-assisted development, the principles are what ensure professionalism:</p>

<ul>
<li>You remain fully responsible for the code you ship.</li>
<li>Everything important must be tested and verified.</li>
<li>Clarity and maintainability outweigh cleverness.</li>
<li>Accuracy and truth matter more than convenience.</li>
<li>Continuous learning is the real measure of progress.</li>
</ul>

<p>The practices may evolve with experience, but these core values never change.</p>

<p><strong>Bringing It Together</strong><br>
Here’s how the two models align:</p>

<p><strong>Low Gear</strong> TDD Focus: Build context and understanding. AI Assistant Focus: Learn through small, verified prompts.</p>

<p><strong>Medium Gear</strong> TDD Focus: Build elegance through design. AI Assistant Focus: Co-design and refactor with critical review.</p>

<p><strong>High Gear</strong> TDD Focus: Solve large or integrated problems. AI Assistant Focus: Scale safely with guardrails and validation.</p>

<p><strong>Reverse Gear</strong> TDD Focus: Back out and restart smaller. AI Assistant Focus: Use source control, revert, and reset scope.</p>

<p>Both models are about deliberate thinking, not speed. They remind us that professionalism lies not in how fast we produce code, but in how consciously we evolve it.</p>

<p><strong>Final Thought</strong><br>
AI assistants, like TDD, are tools for <em>thinking better</em>, not thinking less. The magic happens when we learn when to slow down, when to shift up, and when to reverse.</p>

<p>The next time you’re working with AI, ask yourself:</p>

<ul>
<li>What gear am I in right now?</li>
<li>Have I earned the right to be in this gear?</li>
<li>Do I need to downshift or reverse to regain control?</li>
</ul>

<p>Shifting gears is not about doing more it’s about doing better.</p>

