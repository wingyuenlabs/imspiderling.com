---
Title: Junior Survival Guide for Simplicity
Description: 
Author: Sara A.
Date: 2026-01-26T21:44:07.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Junior Survival Guide for Simplicity
</h1>

<h2>
  
  
  Use Patterns Thoughtfully
</h2>

<ul>
<li>Apply design patterns (e.g., Strategy, Factory, Builder) to solve recurring problems, but avoid over-engineering.
</li>
<li>Avoid "pattern obsession" — only use a pattern when it adds clarity and solves a real problem.</li>
</ul>




<h2>
  
  
  Introduce abstractions (interfaces/protocols/traits) only when you need
</h2>

<ul>
<li>multiple implementations now, or</li>
<li>a clear seam for testing/integration boundaries, or</li>
<li>a genuine contract used by multiple consumers.</li>
</ul>




<h2>
  
  
  Leverage Existing Tools and Libraries
</h2>

<ul>
<li>Prefer well-maintained, widely-used libraries for common problems (validation, HTTP, auth, persistence, scheduling).
</li>
<li>Only build your own when requirements truly don’t fit existing tools.</li>
</ul>




<h2>
  
  
  Favour Convention Over Configuration
</h2>

<ul>
<li>Prefer established conventions and defaults in your stack.
</li>
<li>Avoid excessive custom configuration unless you can explain the concrete benefit and cost.
</li>
<li>Keep configuration minimal and predictable.</li>
</ul>




<h2>
  
  
  Solve Problems, Not Hypotheticals
</h2>

<ul>
<li>Avoid implementing solutions for problems that do not exist yet.
</li>
<li>Design for current requirements, and extend when needed.
</li>
<li>Start with a single, clean implementation and refactor if multiple implementations are required.
</li>
<li>However, do not forget the SOLID principles → objects or entities should be open for extension but closed for modification.</li>
</ul>




<h2>
  
  
  Design for Readability Over Cleverness
</h2>

<ul>
<li>Write code that is easy to read and understand for others (and your future self).
</li>
<li>Avoid overly clever tricks or shortcuts that obscure the intent of the code.</li>
</ul>




<h2>
  
  
  Follow DRY (Don’t Repeat Yourself)
</h2>

<ul>
<li>Extract common logic into reusable methods or classes when multiple areas of the codebase share functionality.
</li>
<li>Ensure the logic is truly universal and not overly specific to a particular use case.</li>
</ul>




<h2>
  
  
  Avoid Over-Generalisation
</h2>

<ul>
<li>Resist the temptation to generalise different business cases into a single method or class just to reduce duplication.
</li>
<li>Over-generalisation can lead to bloated or overly complex code that is harder to understand, maintain, or adapt.</li>
</ul>




<h2>
  
  
  DTO Specialisation Per Module/Domain
</h2>

<ul>
<li>Keep transfer models local to the boundary/context that uses them (API layer, messaging, persistence, internal domain).
</li>
<li>Don’t be afraid to duplicate boundary models if it keeps each context clear and reduces coupling.</li>
</ul>




<h2>
  
  
  Make State and Side Effects Explicit
</h2>

<ul>
<li>Hidden state is the root of most bugs.
</li>
<li>Prefer immutable objects where possible.
</li>
<li>Avoid methods that secretly mutate shared state.
</li>
<li>If a method has side effects (DB writes, HTTP calls, file I/O), make it obvious in the name or structure.</li>
</ul>




<h2>
  
  
  Validate at the Boundaries
</h2>

<ul>
<li>Validate inputs at system edges (controllers, message consumers).
</li>
<li>Fail fast with clear errors.
</li>
<li>Inside the core logic, assume data is already valid.</li>
</ul>




<h2>
  
  
  Prefer Composition Over Inheritance
</h2>

<ul>
<li>Use inheritance only for true “is-a” relationships.
</li>
<li>Prefer composing behaviour via small collaborators.
</li>
<li>Deep inheritance trees are almost always a smell.</li>
</ul>




<h2>
  
  
  Keep Methods Small and Single-Purpose
</h2>

<ul>
<li>One method should do one thing.
</li>
<li>If a method has “and” in its description → split it.
</li>
<li>Long methods hide bugs and make testing painful.</li>
</ul>




<h2>
  
  
  Name Things Like You’re Teaching a Human
</h2>

<ul>
<li>Naming is design.
</li>
<li>Avoid abbreviations unless they are universally known.
</li>
<li>Good names remove the need for comments.</li>
</ul>




<h2>
  
  
  Write Tests That Explain Behaviour
</h2>

<ul>
<li>Tests are executable documentation.
</li>
<li>Test names should read like sentences describing behaviour.
</li>
<li>Test outcomes, not internal implementation details.</li>
</ul>




<h2>
  
  
  Log for Humans, Not for Debuggers
</h2>

<ul>
<li>Logs should explain what happened and why.
</li>
<li>Avoid noise logs like “entered method X”.
</li>
<li>Log business events, not internal plumbing.</li>
</ul>




<h2>
  
  
  Errors Are Part of the API
</h2>

<p>Treat failures as part of the contract:</p>

<ul>
<li>make error cases explicit and meaningful,</li>
<li>avoid silent failures (null/empty/default),</li>
<li>choose the idiom of your language (exceptions, result types, error values) consistently.</li>
</ul>




<h2>
  
  
  Make the Happy Path Obvious
</h2>

<ul>
<li>Structure code so the main flow is immediately visible.
</li>
<li>Handle validation and edge cases early.
</li>
<li>Avoid deeply nested logic.</li>
</ul>




<h2>
  
  
  Delete Code Aggressively
</h2>

<ul>
<li>Dead code is technical debt.
</li>
<li>If it’s unused, remove it.
</li>
<li>Version control already remembers it.
</li>
<li>Commented-out code is a red flag.</li>
</ul>




<h2>
  
  
  The Golden Meta Rule
</h2>

<blockquote>
<p>If something feels complicated, it probably is.<br><br>
Stop and simplify before adding more code.</p>
</blockquote>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwlel4bw83fq04cqov7sr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwlel4bw83fq04cqov7sr.png" alt=" " width="800" height="644"></a></p>

