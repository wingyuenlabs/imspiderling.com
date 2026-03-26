---
Title: Scaling your productivity with spec docs in your IDE - Anti Gravity.
Description: 
Author: Matthew Christiansen
Date: 2026-03-26T21:49:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>Google’s Anti Gravity is built on a simple premise: remove friction so developers can stay in "the flow." But the real unlock isn’t just the IDE itself, it’s how you architect your interactions with the AI inside it. While this article is relevant for any IDE you are using with AI, if you aren't using it yet, I would recommend you take a look at Anti Gravity. </p>

<p>What I am doing with my IDE and AI now:<br>
I’ve started moving away from "chatting" with AI. Instead, I’ve been creating small .md files that act as Instruction Layers for Gemini. This simple shift turns an LLM from a conversational assistant into a predictable, modular part of my technical stack.</p>

<p>The Core Concept: </p>

<p>Prompts as Configuration<br>
Instead of re-explaining my requirements every time I open a terminal, I treat prompts like Angular Components. I create a simple Markdown file that tells Gemini exactly how to behave for a specific task. There are also versions of doing this in certain tools branded as workflows, skills, rules, etc but the premise really is just writing something down easily referencable by your ide to regularly bring into your AI prompts without re-writing that will point your LLM in the right direction when helping with code.</p>

<p>When you find yourself prompting the same thing multiple times, whether in Anti Gravity or any integrated IDE, you shouldn't be typing; you should be referencing a spec and updating it.</p>

<p>An "Angular-Style" Instruction File:</p>

<p>Encapsulated: A clearly defined purpose.<br>
Reusable: Use it across any feature branch.<br>
Consistent: Standardized output every single time.</p>

<p>Example: pr-assistant.md</p>

<h1>
  
  
  PR Comment Assistant
</h1>

<p>Goal: <br>
Draft and post high-quality GitHub PR comments via the git CLI.</p>

<p>Rules:<br>
Keep feedback actionable, professional, and concise.<br>
Call out uncertainty explicitly; no speculation.</p>

<p>Structure: <br>
Context → Suggestion → Reasoning.</p>

<p>Execution:<br>
Output via git CLI; do not use inline IDE comments.<br>
Treat comments as notes for reviewers and future contributors.</p>

<p>Why This Works (Anti Gravity in Practice):<br>
Without structure, you suffer from Prompt Drift. Your instructions get lazy, the output becomes inconsistent, and you repeat yourself constantly.</p>

<p>By using .md instruction files, you achieve Cognitive Offloading:</p>

<p>Predictability: <br>
Gemini follows the spec, not the "vibe" of your last message.<br>
Efficiency: <br>
You stay focused in your IDE, invoking capabilities rather than brainstorming prompts.<br>
Scalability: <br>
These files live in your repo. They evolve with your codebase, they’re version-controlled, and they can be shared across your entire team.<br>
Angular as the Language of Focus:<br>
This approach mirrors core Angular principles almost perfectly:<br>
Separation of Concerns: <br>
Each .md file has one specific job. Reusability: The same instruction file works across different PRs and projects.<br>
Consistency: <br>
You get standardized outputs, every time. Instead of just "talking" to an AI, you are composing it into your development environment.</p>

<p>Closing Thought<br>
The real power of Anti Gravity within an Angular codebase isn’t just writing code faster, it’s reducing the mental overhead of the development process.</p>

<p>When you pair a high-performance IDE with small, well-designed instruction files, you get something bigger than an assistant. You get a custom-built, automated collaborator.</p>

