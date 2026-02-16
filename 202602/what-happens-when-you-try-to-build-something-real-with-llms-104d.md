---
Title: What Happens When You Try to Build Something Real with LLMs
Description: 
Author: John Wade
Date: 2026-02-16T21:35:35.000Z
Robots: noindex,nofollow
Template: index
---


<p>I use ChatGPT, Claude, and Cursor every day. Most developers I know do too. Generating functions, debugging, asking questions, getting unstuck. That part works. The tools are good at short tasks with clear scope.</p>

<p>But at some point I wanted to try something different. Not a one-off script or a quick answer, but a sustained project built across months, where the LLMs weren't just answering questions but helping architect, review, and develop something that accumulated over time.</p>

<p>The project was a personal knowledge management system. I wanted a way to track how my professional knowledge connects across domains: research threads I'm following, skills I'm developing, decisions I've made and why, lessons from projects that I'd otherwise lose to scattered notes and forgotten conversations. Something I could query, build on, and evolve. I designed the architecture across ChatGPT sessions, built the implementation in Cursor, used Claude for adversarial review when something looked too clean to trust. I called it the Career Intelligence Space.</p>

<p>I should be clear about what "built" means here. I wrote some Python scripts. I designed YAML schemas and ontologies. I structured a git repository with governance documents and context packs. But a lot of the building happened in conversations. Long sessions where I'd describe what I needed, the model would propose an architecture, I'd push back, it would revise, and we'd iterate until something held up. The LLMs weren't autocomplete. They were development environments. And that distinction matters, because the failures I hit aren't the failures that come from a tool giving a bad code suggestion. They're the failures that come from a tool being a core part of a development process that runs for months.</p>




<h2>
  
  
  What Breaks
</h2>

<p>Six months in, I had a catalog. Not of features or bugs, but of structural failure modes. Patterns that kept recurring regardless of which model I used, which environment I was in, or how carefully I set things up.</p>

<p>Terms I'd defined precisely would drift. "Career Intelligence Framework" became "Career Intel System" in one session, "CI Framework" in another. Decisions I'd made weeks earlier resurfaced as open questions. The model would build an elaborate solution based on assumptions nobody had tested. One model would produce confident analysis that a second model dismantled in three questions. And the workspace itself would leak: paths, naming conventions, and configuration from one environment contaminating another because no enforcement mechanism existed beyond documentation.</p>

<p>None of this was random. The failures had structure. They stemmed from how LLMs actually work: stateless sessions, finite context windows, pattern regeneration instead of memory retrieval, optimization pressures that reward confidence over accuracy. Once I understood the mechanisms, I could design against them. Not perfectly, but specifically.</p>




<h2>
  
  
  What This Series Covers
</h2>

<p>The series is called <strong>Building at the Edges of LLM Tooling</strong>. I wrote up the failures and the design responses. Each post follows the same arc: something broke, here's the structural reason why, here's what I tried, and here's what the fix revealed about how these tools actually work. Every post ends with a reusable principle, something that generalizes beyond my project to LLM workflows in general.</p>

<p><strong>When an AI Keeps Forgetting:</strong> LLM workflows collapse by default. I cataloged seven failure modes and designed scaffolding to hold them. The lesson: scaffolding exists to prevent collapse so you can do actual work. The moment it becomes the work, you've built the wrong thing.</p>

<p><strong>When the Sandbox Leaks:</strong> Context contamination across workspaces. Two environments that were supposed to be independent kept coupling because documentation isn't a boundary. Enforcement is.</p>

<p><strong>When a Model Goes Wide Instead of Deep:</strong> The model produced analysis that looked thorough but never went deep. Quality gates that block progression until depth criteria are met, not just coverage checks, are what actually hold.</p>

<p><strong>When a Session Forgets What It Knew:</strong> Sessions are compression machines. Every session compresses your project into what fits the context window, and the compression is lossy. The failure isn't the model forgetting. It's building as if it remembers.</p>

<p><strong>When One Model Reviews Its Own Work:</strong> Confident, comprehensive, wrong. A second model caught what the first couldn't see about its own output. Single-model review is structurally limited, and the fix isn't a better prompt. It's a different model with an adversarial brief.</p>




<h2>
  
  
  Who This Is For
</h2>

<p>I used LLMs for small tasks for a long time. Generating a function, explaining an error, drafting a commit message. I never hit these walls. The tools handle short-scope work well.</p>

<p>When I started using them for something bigger, a side project that spans weeks, a documentation system, a workflow that runs across multiple tools, I hit them. The failures weren't obvious when they started. Terms drifted slowly. Context degraded gradually. Confidence looked like quality until someone checked the evidence. By the time the pattern was visible, I'd been building on unstable ground for a while.</p>

<p>This series is what I found when I checked. Start with <a href="https://dev.to/john_wade_dev/when-an-ai-keeps-forgetting-why-llm-workflows-collapse-and-what-to-build-instead-2pj5">When an AI Keeps Forgetting</a>. It lays the foundation everything else builds on.</p>

