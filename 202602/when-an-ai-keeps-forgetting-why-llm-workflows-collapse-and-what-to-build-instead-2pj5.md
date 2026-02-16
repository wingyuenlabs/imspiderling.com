---
Title: When an AI Keeps Forgetting: Why LLM Workflows Collapse and What to Build Instead
Description: 
Author: John Wade
Date: 2026-02-16T21:40:57.000Z
Robots: noindex,nofollow
Template: index
---


<p>I was six months into building a career intelligence project across ChatGPT and Claude when I noticed the rot. Terms I'd defined precisely were drifting — "Career Intelligence Framework" became "Career Intel System" in one session, "CI Framework" in another. Decisions I'd made weeks earlier resurfaced as open questions. I'd explain a concept, get good work from it, then three sessions later have to explain it again because the model had no memory of the conversation where we'd settled it. And references were getting vague — "update the ontology" could mean three different files depending on which session you were in.</p>

<p>I thought something was misconfigured. It wasn't. ChatGPT loses track of terminology due to token windows, memory constraints, and the architecture of LLMs. The drift isn't a bug you can report. It's the default outcome of how these systems work — they regenerate language from patterns rather than retrieving it from stable storage. Every response is a fresh reconstruction, not a recall. Without scaffolding to hold the context steady, long-term projects erode.</p>

<p>That erosion had a shape. I started cataloging it.</p>




<h2>
  
  
  Why It Breaks
</h2>

<p>The failure modes weren't random. They fell into seven categories that kept recurring across every project thread, every model, every session length. I named them C1 through C7 — a collapse risk taxonomy — because naming them precisely was the first step toward designing countermeasures.</p>

<p><strong>C1: Context saturation.</strong> Too much material floods the token window. The model stops tracking earlier context as the window fills. You notice it when the model asks about something you covered twenty messages ago, or when responses get generic because the model can't focus.</p>

<p><strong>C2: Instruction dilution.</strong> Overlapping or conflicting instructions pile up over a conversation. The model tries to satisfy all of them and satisfies none well. You notice it when output quality degrades despite the model having good context — the instructions are competing.</p>

<p><strong>C3: Vocabulary drift.</strong> The model regenerates terminology from patterns instead of retrieving canonical terms. "Career Intelligence Framework" becomes "Career Intelligence System" becomes "your CI project." Each variation is close enough to not flag as wrong, far enough to create ambiguity over time. You notice it when you're not sure which name is the official one.</p>

<p><strong>C4: Reference ambiguity.</strong> "That config file," "the framework we discussed," "update the ontology" — references that were clear in context become vague across sessions. The model doesn't maintain a stable reference table. You notice it when you have to clarify which file, which concept, which version.</p>

<p><strong>C5: Goal creep.</strong> Project scope shifts silently. The model follows wherever the conversation leads, and conversations naturally expand. You started designing a context pack template; three exchanges later you're discussing full automation pipelines. No checkpoint catches the drift because the model doesn't maintain a stable goal state.</p>

<p><strong>C6: Evidence entropy.</strong> Provenance erodes. A decision was made, but the rationale lives in a conversation that's now compressed or closed. The evidence that supported a design choice degrades because there's no frozen state — no snapshot of what was known and decided at a given moment.</p>

<p><strong>C7: Thread fragmentation.</strong> Conversations split across sessions, models, and tools. The discussion about vocabulary lives in one thread; the architecture decision lives in another; the implementation happened in a third. No single view stitches them together, and the model in any one thread can't see the others.</p>

<p>These aren't independent failures. They compound. Vocabulary drift (C3) creates reference ambiguity (C4). Context saturation (C1) accelerates instruction dilution (C2). Thread fragmentation (C7) makes all of them worse because no single session has the full picture.</p>




<h2>
  
  
  What I Tried
</h2>

<p>The first instinct was comprehensive: design a full scaffolding system — what I called the Harness — that mapped every collapse risk to a specific countermeasure.</p>

<p><strong>Ontology.yml</strong> for vocabulary stability. One canonical term per concept, approved synonyms, unique IDs. When the model drifts from "Career Intelligence Framework" to "Career Intel System," the ontology is the authority. In theory, a linter could catch drift in pull requests before it enters the codebase.</p>

<p><strong>Context Packs</strong> for session re-injection. Instead of dumping the full project into the context window and hoping for the best, a compact bundle — purpose, glossary slice, current milestone, active constraints, open questions — gives the model exactly what it needs for this session and nothing else. Anti-C1 and anti-C2 by design: lean input, focused output.</p>

<p><strong>Decision Log</strong> for rationale tracking. When a design choice is made, the log captures what was decided, why, what alternatives were considered, and whether it's reversible. This fights C4 and C5 — references stay unambiguous and scope changes are explicit rather than silent.</p>

<p><strong>Checkpoints</strong> for frozen state. Git tags and releases that capture the repository at a known-good moment. Evidence entropy (C6) happens when there's no stable reference point — checkpoints create them.</p>

<p><strong>Chronicle</strong> for continuity. A narrative timeline that stitches fragmented conversations into a coherent history. When work splits across sessions and models (C7), the chronicle is the single thread that connects them.</p>

<p>The mapping was clean: every component targeted specific collapse risks, and every collapse risk had at least one countermeasure. But then the design tension hit. I was comparing this full Harness approach against a simpler one — Stage B, a GitHub-based architecture with clear repository structure, frontmatter conventions, and manual discipline but no automation layer. ChatGPT's assessment was direct: the Harness had "complexity overhead; requires upkeep; risks being 'meta-work' that slows real tasks." Stage B had "less explicit about entropy risks; relies on manual verification."</p>

<p>The risk was building scaffolding that becomes the project. Spending more time maintaining the immune system than doing the work the immune system was supposed to protect.</p>




<h2>
  
  
  What It Revealed
</h2>

<p>The Harness design exposed a principle I kept coming back to: the scaffolding is a servo, not the engine. A servo keeps the system upright. The engine does the actual work. When the servo becomes the work — when you're spending sessions on ontology maintenance and context pack generation instead of the research and writing the project exists to produce — you've built the wrong thing.</p>

<p>The deeper insight was that collapse isn't a failure state. It's the default state. LLMs don't have memory — they have pattern regeneration. They don't retrieve your terms; they reconstruct approximations. They don't recall your decisions; they infer from whatever's in the current window. They don't maintain project state; they work in a perpetual present. Everything outside the current context window is effectively gone unless you re-inject it. And re-injection is always lossy — summaries strip nuance, context packs compress rationale, session handoffs lose texture.</p>

<p>Once I understood collapse as the baseline rather than the exception, the design question changed. It wasn't "how do I prevent collapse?" — you can't, not fully, not with current architecture. It was "where do I need the scaffolding to hold, and where can I let it flex?" Vocabulary stability matters for a long-running project — terms that drift create compounding ambiguity. Decision provenance matters when you'll revisit choices months later. Session continuity matters less for one-off analysis and more for iterative development across weeks.</p>

<p>The scaffolding spectrum that emerged: start with repository structure and naming conventions (the things that cost nothing and prevent the most common failures). Add an ontology when vocabulary drift becomes noticeable. Add context packs when session re-entry starts taking more time than the work itself. Add automation when manual discipline fails under load. Don't build the full Harness on day one. Build the piece that addresses the collapse risk you're actually hitting.</p>




<h2>
  
  
  The Reusable Rule
</h2>

<p>If you're running a long-term project through LLMs — anything that spans more than a few sessions — the model will lose track. Not because it's broken, but because it doesn't have memory. It has pattern regeneration operating inside a finite context window, and everything outside that window is gone.</p>

<p>The diagnostic: when you're re-explaining concepts the model already worked with, that's context saturation or vocabulary drift. When the model suggests something you already rejected, that's evidence entropy. When you're not sure which version of a term is canonical, that's vocabulary drift compounding into reference ambiguity. When the conversation has quietly shifted from your original goal to something adjacent, that's goal creep. Name the failure. The C1–C7 taxonomy isn't sacred — rename them, extend them, collapse them — but catalog what's actually breaking so you can design against it specifically rather than generically.</p>

<p>The scaffolding principle: add components when specific collapse risks appear, not before. Map every piece of infrastructure to the failure it prevents. If it doesn't prevent a named failure, it's probably meta-work.</p>

<p>And remember the servo principle: scaffolding exists to prevent collapse so you can do actual work. The moment it becomes the work, you've built the wrong thing.</p>

