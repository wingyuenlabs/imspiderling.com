---
Title: Stop Chatting, Start Specifying: Spec-Driven Design with Kiro IDE
Description: 
Author: Pablo Albaladejo
Date: 2025-12-01T21:36:29.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Building Kaiord with Kiro: From Specs to Production
</h2>

<p><strong>How Spec-Driven Development and Test-Driven Development Changed the Way I Code</strong></p>




<h2>
  
  
  The Problem with Chat-First Development
</h2>

<p>We've all been there. You open your AI assistant and start chatting:</p>

<p><em>"Can you help me convert FIT files to JSON?"</em></p>

<p>The AI immediately writes code. But wait, that's not exactly what you wanted. So you clarify:</p>

<p><em>"Actually, I need it to preserve all the workout data..."</em></p>

<p>More code appears. Still not quite right. Another message:</p>

<p><em>"No, I meant it should also handle repetition blocks..."</em></p>

<p>This back-and-forth conversation fills up the context window. By the time you get working code, there's no space left for the AI to help with the details. <strong>The problem: we're asking AI to write code before understanding what we need.</strong></p>




<h2>
  
  
  Enter Spec-Driven Development
</h2>

<p>When I discovered Kiro and its spec-driven approach, everything changed. Instead of jumping straight to code, I learned to <strong>think first, code later</strong>.</p>

<p>Here's how it works:</p>

<h3>
  
  
  Step 1: Create a Specification with Kiro
</h3>

<p>Before writing any code, Kiro helps me create three documents by asking me questions:</p>

<ol>
<li>
<strong>requirements.md</strong> - What the feature needs to do</li>
<li>
<strong>design.md</strong> - How it will work</li>
<li>
<strong>tasks.md</strong> - Step-by-step implementation plan</li>
</ol>

<p>I tell Kiro "I need to convert FIT files to KRD format," and Kiro starts asking questions:</p>

<ul>
<li><em>"What data needs to be preserved?"</em></li>
<li><em>"What are the acceptance criteria?"</em></li>
<li><em>"What tolerances are acceptable for round-trip conversion?"</em></li>
</ul>

<p>Based on my answers, Kiro generates the specification documents. Let me show you a real example from Kaiord. When I needed to convert FIT files (Garmin's binary format) to KRD (my JSON format), Kiro created this spec:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># fit-to-krd-conversion/requirements.md</span>

<span class="gu">## User Story</span>
As a cyclist or runner, I want to convert FIT workout files to KRD format, 
so that I can edit workout structures in a human-readable format and validate 
them against a standard schema.

<span class="gu">## Acceptance Criteria</span>
<span class="p">-</span> WHEN the converter receives a valid FIT file
<span class="p">-</span> THEN it SHALL produce a valid KRD document
<span class="p">-</span> AND it SHALL preserve all workout steps
<span class="p">-</span> AND it SHALL validate against the KRD schema
</code></pre>

</div>



<p>You can see the full spec in <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/specs/core/fit-to-krd-conversion/requirements.md" rel="noopener noreferrer"><code>.kiro/specs/core/fit-to-krd-conversion/requirements.md</code></a>.</p>

<p><strong>Why this matters:</strong> The spec becomes Kiro's guide. Instead of guessing what I want, Kiro has a clear document that captures my requirements through our conversation.</p>

<h3>
  
  
  Step 2: Kiro Defines the Architecture
</h3>

<p>After understanding my requirements, Kiro proposes the architecture in the design document. It asks me questions like:</p>

<ul>
<li><em>"Should we use hexagonal architecture with ports and adapters?"</em></li>
<li><em>"What external libraries will we use?"</em></li>
<li><em>"What are the key design goals?"</em></li>
</ul>

<p>Based on my answers, Kiro generates the design document describing how the conversion will work:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># design.md</span>

<span class="gu">## Architecture</span>

We'll create:
<span class="p">-</span> Port interface: FitReader (domain/ports)
<span class="p">-</span> Adapter implementation: garmin-fitsdk (adapters/fit)
<span class="p">-</span> Use case: convertFitToKrd (application/use-cases)

<span class="gu">### Key Design Goals</span>
<span class="p">
1.</span> <span class="gs">**Round-trip safety**</span>: FIT → KRD → FIT preserves data within tolerances
<span class="p">2.</span> <span class="gs">**Hexagonal architecture**</span>: Domain logic isolated from external dependencies
<span class="p">3.</span> <span class="gs">**Dependency injection**</span>: Swappable FIT SDK implementations via ports
<span class="p">4.</span> <span class="gs">**Schema validation**</span>: All KRD output validated against JSON schema
<span class="p">5.</span> <span class="gs">**Type safety**</span>: Strict TypeScript with no implicit <span class="sb">`any`</span>
</code></pre>

</div>



<p>This planning phase saved me hours of refactoring later. The code structure was clear before Kiro wrote a single line.</p>

<p>See <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/steering/architecture.md" rel="noopener noreferrer"><code>.kiro/steering/architecture.md</code></a> for the full architecture rules.</p>

<h3>
  
  
  Step 3: Kiro Breaks Down Tasks
</h3>

<p>Finally, Kiro breaks the work into small, testable pieces in the tasks document:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># tasks.md</span>
<span class="p">
-</span> [ ] Define FitReader port interface
<span class="p">-</span> [ ] Implement Garmin FIT SDK adapter
<span class="p">-</span> [ ] Create convertFitToKrd use case
<span class="p">-</span> [ ] Write unit tests (AAA pattern)
<span class="p">-</span> [ ] Add round-trip validation
<span class="p">-</span> [ ] Document API examples
</code></pre>

</div>



<p>Each task is small enough for Kiro to implement in one sitting, making progress visible and momentum sustainable. I review each task, and Kiro implements it following TDD.</p>




<h2>
  
  
  Steering Documents: Teaching Kiro My Rules
</h2>

<p>Before Kiro writes any code, I need to teach it how I want things done. This is where <strong>steering documents</strong> come in.</p>

<p>Steering documents are markdown files in <code>.kiro/steering/</code> that define the rules Kiro must follow. Think of them as a style guide, architecture manual, and best practices document all in one—but for AI.</p>

<p>Here's what I defined for Kaiord:</p>

<h3>
  
  
  My Steering Documents
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.kiro/steering/
├── architecture.md       # Hexagonal architecture rules
├── tdd.md               # Test-driven development workflow
├── code-style.md        # Code quality standards
├── zod-patterns.md      # Schema-first patterns
├── testing.md           # Testing strategies
├── error-patterns.md    # Error handling patterns
└── ... 18 documents total
</code></pre>

</div>



<p>Each document tells Kiro:</p>

<ul>
<li>
<strong>What patterns to follow</strong> (e.g., "Use hexagonal architecture")</li>
<li>
<strong>What to avoid</strong> (e.g., "Never use <code>any</code> types")</li>
<li>
<strong>How to structure code</strong> (e.g., "Files ≤ 100 lines, functions &lt; 40 LOC")</li>
<li>
<strong>When to apply rules</strong> (e.g., "Write tests first, then implementation")</li>
</ul>

<p><strong>The key insight:</strong> Kiro reads these documents automatically while working. I don't need to repeat myself in every conversation. The rules are always there, guiding every decision Kiro makes.</p>

<h3>
  
  
  Example: Architecture Steering Document
</h3>

<p>Here's a snippet from my <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/steering/architecture.md" rel="noopener noreferrer"><code>.kiro/steering/architecture.md</code></a>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Architecture — Hexagonal &amp; DI</span>

Layers:
<span class="p">
-</span> <span class="gs">**domain/**</span> — pure KRD types &amp; rules
<span class="p">-</span> <span class="gs">**application/**</span> — use-cases; depends on <span class="sb">`ports/`</span> only
<span class="p">-</span> <span class="gs">**ports/**</span> — I/O contracts (Fit/Tcx/Zwift Reader/Writer)
<span class="p">-</span> <span class="gs">**adapters/**</span> — concrete implementations (e.g., @garmin/fitsdk)
<span class="p">-</span> <span class="gs">**cli/**</span> — end-user commands; depends on <span class="sb">`application`</span>

Rules:
<span class="p">
-</span> <span class="sb">`domain`</span> depends on no one
<span class="p">-</span> <span class="sb">`application`</span> MUST NOT import external libs nor <span class="sb">`adapters/`</span>
<span class="p">-</span> <span class="sb">`adapters`</span> implement <span class="sb">`ports`</span> and may use external libs
</code></pre>

</div>



<p>When Kiro generates code, it automatically follows these architecture rules. If I accidentally try to import an adapter in a use case, Kiro warns me about the violation. I never have to remind Kiro about architectural boundaries—it's all in the steering document.</p>




<h2>
  
  
  Growing Together: Managing Codebase Complexity
</h2>

<p>As a codebase grows, it's easy for things to get messy—especially when an AI can generate hundreds of lines in seconds. To keep our collaboration smooth and productive, I found that Kiro works best when we have clear processes and healthy boundaries.</p>

<p>Think of it as setting up guardrails so we can run fast without falling off a cliff.</p>

<h3>
  
  
  1. The Safety Net: Test-Driven Development (TDD)
</h3>

<p>For me, TDD isn't just a methodology; it's how I communicate my intent to Kiro. It acts as our safety net, allowing us to build features and refactor code with confidence.</p>

<p>With TDD rules defined in my steering documents, Kiro follows this workflow automatically:</p>

<ol>
<li>
<strong>Kiro writes the test first</strong> (it fails - red)</li>
<li>
<strong>Kiro writes code to pass the test</strong> (it passes - green)</li>
<li>
<strong>We refactor together</strong> (keep tests green - refactor)</li>
</ol>

<p>Here's a real example from <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/application/use-cases/convert-fit-to-krd.ts" rel="noopener noreferrer"><code>packages/core/src/application/use-cases/convert-fit-to-krd.ts</code></a>:</p>

<h3>
  
  
  Step 1: Kiro Writes the Test (Red)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// packages/core/src/application/use-cases/convert-fit-to-krd.test.ts</span>
<span class="nf">describe</span><span class="p">(</span><span class="dl">"</span><span class="s2">convertFitToKrd</span><span class="dl">"</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">it</span><span class="p">(</span><span class="dl">"</span><span class="s2">should convert FIT workout to KRD format</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Arrange</span>
    <span class="kd">const</span> <span class="nx">fitBuffer</span> <span class="o">=</span> <span class="nf">loadFitFixture</span><span class="p">(</span><span class="dl">"</span><span class="s2">WorkoutIndividualSteps.fit</span><span class="dl">"</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">fitReader</span> <span class="o">=</span> <span class="nf">createMockFitReader</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">validator</span> <span class="o">=</span> <span class="nf">createSchemaValidator</span><span class="p">();</span>

    <span class="c1">// Act</span>
    <span class="kd">const</span> <span class="nx">krd</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">convertFitToKrd</span><span class="p">(</span><span class="nx">fitReader</span><span class="p">,</span> <span class="nx">validator</span><span class="p">)({</span>
      <span class="nx">fitBuffer</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="c1">// Assert</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">krd</span><span class="p">.</span><span class="nx">version</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">"</span><span class="s2">1.0</span><span class="dl">"</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">krd</span><span class="p">.</span><span class="kd">type</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">"</span><span class="s2">workout</span><span class="dl">"</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">krd</span><span class="p">.</span><span class="nx">extensions</span><span class="p">.</span><span class="nx">workout</span><span class="p">.</span><span class="nx">steps</span><span class="p">).</span><span class="nf">toHaveLength</span><span class="p">(</span><span class="mi">4</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p>This test fails at first because <code>convertFitToKrd</code> doesn't exist yet.</p>

<h3>
  
  
  Step 2: Kiro Writes Minimal Code (Green)
</h3>

<p>Only after writing the test does Kiro implement the actual conversion function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// packages/core/src/application/use-cases/convert-fit-to-krd.ts</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">convertFitToKrd</span> <span class="o">=</span>
  <span class="p">(</span><span class="nx">fitReader</span><span class="p">:</span> <span class="nx">FitReader</span><span class="p">,</span> <span class="nx">validator</span><span class="p">:</span> <span class="nx">SchemaValidator</span><span class="p">)</span> <span class="o">=&gt;</span>
  <span class="k">async </span><span class="p">(</span><span class="nx">params</span><span class="p">:</span> <span class="nx">ConvertFitToKrdParams</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">KRD</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// Read FIT file</span>
    <span class="kd">const</span> <span class="nx">krd</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fitReader</span><span class="p">(</span><span class="nx">params</span><span class="p">.</span><span class="nx">fitBuffer</span><span class="p">);</span>

    <span class="c1">// Validate result</span>
    <span class="kd">const</span> <span class="nx">errors</span> <span class="o">=</span> <span class="nx">validator</span><span class="p">.</span><span class="nf">validate</span><span class="p">(</span><span class="nx">krd</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">errors</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">throw</span> <span class="k">new</span> <span class="nc">KrdValidationError</span><span class="p">(</span><span class="dl">"</span><span class="s2">Validation failed</span><span class="dl">"</span><span class="p">,</span> <span class="nx">errors</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c1">// Return validated KRD</span>
    <span class="k">return</span> <span class="nx">krd</span><span class="p">;</span>
  <span class="p">};</span>
</code></pre>

</div>



<p>The test passes! But we're not done yet.</p>

<h3>
  
  
  Step 3: We Refactor Together
</h3>

<p>Now Kiro and I improve the code while keeping tests green. Kiro might suggest:</p>

<ul>
<li>Extracting helper functions</li>
<li>Improving error messages</li>
<li>Adding edge case handling</li>
</ul>

<p>I review the suggestions, and Kiro applies the changes—all while the tests continue to pass.</p>

<h3>
  
  
  The AAA Pattern
</h3>

<p>All our tests follow the <strong>AAA pattern</strong> (Arrange-Act-Assert):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">it</span><span class="p">(</span><span class="dl">"</span><span class="s2">should validate KRD against schema</span><span class="dl">"</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Arrange</span>
  <span class="kd">const</span> <span class="nx">validator</span> <span class="o">=</span> <span class="nf">createSchemaValidator</span><span class="p">(</span><span class="nx">mockLogger</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">invalidKrd</span> <span class="o">=</span> <span class="p">{</span> <span class="na">version</span><span class="p">:</span> <span class="dl">"</span><span class="s2">1.0</span><span class="dl">"</span> <span class="p">};</span> <span class="c1">// missing required fields</span>

  <span class="c1">// Act</span>
  <span class="kd">const</span> <span class="nx">errors</span> <span class="o">=</span> <span class="nx">validator</span><span class="p">.</span><span class="nf">validate</span><span class="p">(</span><span class="nx">invalidKrd</span><span class="p">);</span>

  <span class="c1">// Assert</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">errors</span><span class="p">).</span><span class="nf">toHaveLength</span><span class="p">(</span><span class="mi">2</span><span class="p">);</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">errors</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">field</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">);</span>
  <span class="nf">expect</span><span class="p">(</span><span class="nx">errors</span><span class="p">[</span><span class="mi">1</span><span class="p">].</span><span class="nx">field</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">"</span><span class="s2">metadata</span><span class="dl">"</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p>This pattern makes tests easy to read and understand. The Arrange section sets up test data, the Act section calls the function, and the Assert section checks the result.</p>

<p><strong>The result:</strong> The code works the first time. No debugging sessions trying to figure out why something breaks.</p>

<p>You can see the TDD guidelines I defined in <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/steering/tdd.md" rel="noopener noreferrer"><code>.kiro/steering/tdd.md</code></a>. Kiro follows these rules for every function in Kaiord.</p>

<h3>
  
  
  2. Healthy Boundaries: Small Files, Happy Code
</h3>

<p>Just like us, Kiro does better work when tasks are bite-sized. I've set up some helpful constraints that encourage us to keep things simple:</p>

<ul>
<li>
<strong>Keep files small</strong> (≤ 100 lines)</li>
<li>
<strong>Keep functions focused</strong> (&lt; 40 LOC)</li>
<li>
<strong>Be specific</strong> (NO <code>any</code> types)</li>
</ul>

<p>These aren't just rigid rules; they actually help Kiro write better, more maintainable code. When a file starts getting too long, Kiro notices and suggests, <em>"Hey, maybe we should split this up?"</em></p>

<p><strong>Example:</strong> When we were building the FIT converter, Kiro naturally organized the code into small, focused modules instead of one giant file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>adapters/fit/
├── garmin-fitsdk.ts          # Main entry (&lt; 100 lines)
├── duration.mapper.ts         # Duration mapping (&lt; 100 lines)
├── target.converter.ts        # Target conversion (&lt; 100 lines)
└── workout.mapper.ts          # Workout mapping (&lt; 100 lines)
</code></pre>

</div>



<p>Each file stays under 100 lines, making the code easy to understand for both of us.</p>

<p>These automated checks run in the background, gently nudging us back to the right path if we stray. <strong>The result:</strong> A codebase that feels clean and manageable, no matter how much feature work we do.</p>




<h3>
  
  
  Architecture Enforcement
</h3>

<p>I defined hexagonal architecture rules in steering docs that Kiro enforces. The key rule:</p>

<blockquote>
<p><strong>Domain depends on nothing. Application depends only on domain and ports. Adapters implement ports.</strong></p>
</blockquote>

<p>When Kiro generates code, it automatically:</p>

<ul>
<li>Creates port interfaces (contracts)</li>
<li>Implements adapters (external libraries)</li>
<li>Writes use cases (business logic)</li>
<li>Wires everything in providers (dependency injection)</li>
</ul>

<p>If I accidentally try to import an adapter in a use case, Kiro warns me:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>⚠️ Architecture violation detected
   Application layer should not import adapters
   Import the port interface instead
</code></pre>

</div>



<p>This keeps the codebase maintainable and testable. See <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/steering/architecture.md" rel="noopener noreferrer"><code>.kiro/steering/architecture.md</code></a> for the full rules I defined.</p>




<h2>
  
  
  Real Examples from Kaiord
</h2>

<p>Let me show you three real examples where spec-driven development with Kiro made a huge difference:</p>

<h3>
  
  
  Example 1: FIT to KRD Conversion
</h3>

<p><strong>The challenge:</strong> Convert Garmin's binary FIT format to my JSON format without losing data.</p>

<p><strong>The spec:</strong> <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/specs/core/fit-to-krd-conversion/requirements.md" rel="noopener noreferrer"><code>.kiro/specs/core/fit-to-krd-conversion/requirements.md</code></a></p>

<p><strong>What Kiro generated:</strong></p>

<ul>
<li>Port interface in <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/ports/fit-reader.ts" rel="noopener noreferrer"><code>packages/core/src/ports/fit-reader.ts</code></a>
</li>
<li>Adapter implementation in <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/adapters/fit/garmin-fitsdk.ts" rel="noopener noreferrer"><code>packages/core/src/adapters/fit/garmin-fitsdk.ts</code></a>
</li>
<li>Use case in <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/application/use-cases/convert-fit-to-krd.ts" rel="noopener noreferrer"><code>packages/core/src/application/use-cases/convert-fit-to-krd.ts</code></a>
</li>
<li>Complete test suite with 90%+ coverage</li>
<li>Round-trip validation to ensure no data loss</li>
</ul>

<p><strong>Time saved:</strong> What would have taken me 2 weeks of trial-and-error took 3 days with clear specifications.</p>

<h3>
  
  
  Example 2: Duration Types with Discriminated Unions
</h3>

<p><strong>The challenge:</strong> Support 14 different duration types (time, distance, calories, heart rate conditions, power conditions, etc.)</p>

<p><strong>The spec:</strong> <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/specs/core/zod-schema-migration/design.md" rel="noopener noreferrer"><code>.kiro/specs/core/zod-schema-migration/design.md</code></a></p>

<p><strong>What Kiro generated:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// From https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/domain/schemas/duration.ts</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">durationSchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">discriminatedUnion</span><span class="p">(</span><span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">,</span> <span class="p">[</span>
  <span class="c1">// Simple durations</span>
  <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">literal</span><span class="p">(</span><span class="dl">"</span><span class="s2">time</span><span class="dl">"</span><span class="p">),</span> <span class="na">seconds</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">()</span> <span class="p">}),</span>
  <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">literal</span><span class="p">(</span><span class="dl">"</span><span class="s2">distance</span><span class="dl">"</span><span class="p">),</span> <span class="na">meters</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">()</span> <span class="p">}),</span>
  <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">literal</span><span class="p">(</span><span class="dl">"</span><span class="s2">calories</span><span class="dl">"</span><span class="p">),</span> <span class="na">calories</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">()</span> <span class="p">}),</span>

  <span class="c1">// Conditional durations</span>
  <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
    <span class="na">type</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">literal</span><span class="p">(</span><span class="dl">"</span><span class="s2">heart_rate_less_than</span><span class="dl">"</span><span class="p">),</span>
    <span class="na">heartRate</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">(),</span>
    <span class="na">seconds</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">().</span><span class="nf">optional</span><span class="p">()</span>
  <span class="p">}),</span>
  <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
    <span class="na">type</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">literal</span><span class="p">(</span><span class="dl">"</span><span class="s2">power_greater_than</span><span class="dl">"</span><span class="p">),</span>
    <span class="na">power</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">(),</span>
    <span class="na">seconds</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">().</span><span class="nf">optional</span><span class="p">()</span>
  <span class="p">}),</span>

  <span class="c1">// ... 14 types total</span>
<span class="p">]);</span>
</code></pre>

</div>



<p><strong>The benefit:</strong> TypeScript knows exactly which fields exist for each duration type. No more runtime errors from accessing wrong fields!</p>

<h3>
  
  
  Example 3: Round-Trip Validation
</h3>

<p><strong>The challenge:</strong> Ensure conversions preserve data (FIT → KRD → FIT should match original).</p>

<p><strong>The implementation:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// packages/core/src/application/use-cases/validate-round-trip.ts</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">validateRoundTrip</span> <span class="o">=</span> <span class="p">(</span>
  <span class="nx">fitReader</span><span class="p">:</span> <span class="nx">FitReader</span><span class="p">,</span>
  <span class="nx">fitWriter</span><span class="p">:</span> <span class="nx">FitWriter</span><span class="p">,</span>
  <span class="nx">checker</span><span class="p">:</span> <span class="nx">ToleranceChecker</span><span class="p">,</span>
  <span class="nx">logger</span><span class="p">:</span> <span class="nx">Logger</span>
<span class="p">)</span> <span class="o">=&gt;</span> <span class="k">async </span><span class="p">(</span><span class="nx">params</span><span class="p">:</span> <span class="nx">ValidateRoundTripParams</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Convert FIT → KRD → FIT</span>
  <span class="kd">const</span> <span class="nx">krd</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fitReader</span><span class="p">(</span><span class="nx">params</span><span class="p">.</span><span class="nx">fitBuffer</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">convertedFit</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fitWriter</span><span class="p">(</span><span class="nx">krd</span><span class="p">);</span>

  <span class="c1">// Check tolerances</span>
  <span class="kd">const</span> <span class="nx">deviations</span> <span class="o">=</span> <span class="nx">checker</span><span class="p">.</span><span class="nf">checkTolerances</span><span class="p">(</span>
    <span class="nx">params</span><span class="p">.</span><span class="nx">fitBuffer</span><span class="p">,</span>
    <span class="nx">convertedFit</span>
  <span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">deviations</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">ToleranceExceededError</span><span class="p">(</span><span class="dl">"</span><span class="s2">Tolerances exceeded</span><span class="dl">"</span><span class="p">,</span> <span class="nx">deviations</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>Tolerances:</strong></p>

<ul>
<li>Time: ±1 second</li>
<li>Power: ±1 watt or ±1% FTP</li>
<li>Heart Rate: ±1 bpm</li>
<li>Cadence: ±1 rpm</li>
</ul>

<p>This ensures data integrity across all conversions.</p>




<h2>
  
  
  Automation with Agent Hooks
</h2>

<p>One of Kiro's most powerful features is agent hooks - automated workflows that trigger based on events.</p>

<p>I created a hook that automatically generates GitHub pull requests from my specs. Here's how it works:</p>

<p><strong>Hook configuration:</strong> <a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/hooks/create-github-pr.kiro.hook" rel="noopener noreferrer"><code>.kiro/hooks/create-github-pr.kiro.hook</code></a></p>

<p>When I finish a feature, the hook:</p>

<ol>
<li>Reads my spec from <code>.kiro/specs/[feature-name]/</code>
</li>
<li>Gets the requirements, design, and tasks</li>
<li>Generates a PR title from the main goal</li>
<li>Fills the PR template with spec content</li>
<li>Creates the PR using GitHub MCP (Model Context Protocol)</li>
<li>Adds appropriate labels</li>
</ol>

<p><strong>Time saved:</strong> Creating a PR went from 10 minutes to 30 seconds.</p>

<p><strong>Bonus:</strong> Every PR has complete documentation because it's generated from the spec!</p>




<h2>
  
  
  The Numbers Tell the Story
</h2>

<p>After building Kaiord with Kiro's spec-driven approach:</p>

<h3>
  
  
  Code Quality
</h3>

<ul>
<li>
<strong>86.5% test coverage</strong> - More than most production apps</li>
<li>
<strong>300+ unit tests</strong> - TDD from day one</li>
<li>
<strong>55+ E2E tests</strong> - Playwright for integration testing</li>
<li>
<strong>0 <code>any</code> types</strong> - Complete type safety</li>
<li>
<strong>550+ TypeScript files</strong> - All following architecture rules</li>
</ul>

<h3>
  
  
  Test Coverage by Package
</h3>

<ul>
<li>
<strong>Core Package:</strong> 88% (target: ≥80%)</li>
<li>
<strong>Converters/Mappers:</strong> 94% (target: ≥90%)</li>
<li>
<strong>CLI Package:</strong> 82% (target: ≥70%)</li>
<li>
<strong>Web SPA:</strong> 86.5% (target: ≥70%)</li>
</ul>

<h3>
  
  
  Kiro Integration
</h3>

<ul>
<li>
<strong>30+ specifications</strong> in <a href="https://github.com/pablo-albaladejo/kaiord/tree/main/.kiro/specs" rel="noopener noreferrer"><code>.kiro/specs/</code></a>
</li>
<li>
<strong>18 steering documents</strong> in <a href="https://github.com/pablo-albaladejo/kaiord/tree/main/.kiro/steering" rel="noopener noreferrer"><code>.kiro/steering/</code></a>
</li>
<li>
<strong>8 agent hooks</strong> in <a href="https://github.com/pablo-albaladejo/kaiord/tree/main/.kiro/hooks" rel="noopener noreferrer"><code>.kiro/hooks/</code></a>
</li>
<li>
<strong>100% of code</strong> generated with Kiro's help</li>
</ul>

<h3>
  
  
  Production Ready
</h3>

<ul>
<li>
<strong>3 npm packages published:</strong>

<ul>
<li>
<a href="https://www.npmjs.com/package/@kaiord/core" rel="noopener noreferrer">@kaiord/core</a> - Core library</li>
<li>
<a href="https://www.npmjs.com/package/@kaiord/cli" rel="noopener noreferrer">@kaiord/cli</a> - Command-line tool</li>
</ul>


</li>

<li>

<strong>1 web app deployed:</strong> <a href="https://pablo-albaladejo.github.io/kaiord/" rel="noopener noreferrer">Live demo</a>
</li>

<li>

<strong>5 CI/CD workflows</strong> - Automated testing, linting, deployment</li>

</ul>




<h2>
  
  
  What I Learned
</h2>

<h3>
  
  
  1. Planning Saves Time
</h3>

<p>Writing specs feels slow at first. But it's much faster than:</p>

<ul>
<li>Rewriting code because requirements changed</li>
<li>Debugging code you don't understand</li>
<li>Explaining to the AI what you already explained 5 times</li>
</ul>

<p><strong>Example:</strong> The FIT conversion spec took 2 hours to write. It saved me 2 weeks of confusion.</p>

<h3>
  
  
  2. Tests Give Confidence
</h3>

<p>Following TDD means I know my code works. When I add features, the tests tell me immediately if I broke something.</p>

<p><strong>Example:</strong> When I added support for swimming workouts, my existing tests caught 3 bugs before I even ran the app.</p>

<h3>
  
  
  3. Architecture Matters
</h3>

<p>Using hexagonal architecture (ports and adapters) made my code:</p>

<ul>
<li>Easy to test (mock the ports)</li>
<li>Easy to extend (add new adapters)</li>
<li>Easy to understand (clear boundaries)</li>
</ul>

<p><strong>Example:</strong> Adding TCX format support required zero changes to existing code - just a new adapter.</p>

<h3>
  
  
  4. AI Works Better with Structure
</h3>

<p>Kiro's spec-driven approach gives AI the context it needs to help effectively. Instead of guessing, it implements what's documented.</p>

<h3>
  
  
  5. Specs Save Time
</h3>

<p>Clear specs mean:</p>

<ul>
<li>Less confusion during implementation</li>
<li>Fewer bugs from misunderstood requirements</li>
<li>Better code reviews</li>
<li>Easier onboarding for new developers</li>
</ul>

<h3>
  
  
  6. TDD Prevents Bugs
</h3>

<p>Writing tests first catches bugs early. When we write a test, we think about:</p>

<ul>
<li>What should happen in normal cases?</li>
<li>What should happen in edge cases?</li>
<li>What should happen when errors occur?</li>
</ul>

<p>This thinking prevents many bugs before they're written.</p>




<h2>
  
  
  Try It Yourself
</h2>

<p>Want to see spec-driven development in action? Check out Kaiord:</p>

<ul>
<li>
<strong>Source code:</strong> <a href="https://github.com/pablo-albaladejo/kaiord" rel="noopener noreferrer">github.com/pablo-albaladejo/kaiord</a>
</li>
<li>
<strong>Live demo:</strong> <a href="https://pablo-albaladejo.github.io/kaiord/" rel="noopener noreferrer">pablo-albaladejo.github.io/kaiord</a>
</li>
<li>
<strong>Specifications:</strong> Browse <a href="https://github.com/pablo-albaladejo/kaiord/tree/main/.kiro/specs" rel="noopener noreferrer"><code>.kiro/specs/</code></a> to see real examples</li>
<li>
<strong>Steering docs:</strong> Read <a href="https://github.com/pablo-albaladejo/kaiord/tree/main/.kiro/steering" rel="noopener noreferrer"><code>.kiro/steering/</code></a> for architecture rules</li>
</ul>

<h3>
  
  
  Key Files to Explore
</h3>

<p><strong>Specifications (see how Kiro plans):</strong></p>

<ul>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/specs/core/fit-to-krd-conversion/requirements.md" rel="noopener noreferrer"><code>.kiro/specs/core/fit-to-krd-conversion/requirements.md</code></a> - How Kiro planned FIT conversion</li>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/specs/core/zod-schema-migration/design.md" rel="noopener noreferrer"><code>.kiro/specs/core/zod-schema-migration/design.md</code></a> - Schema-first approach</li>
</ul>

<p><strong>Steering Docs (see the rules Kiro follows):</strong></p>

<ul>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/steering/architecture.md" rel="noopener noreferrer"><code>.kiro/steering/architecture.md</code></a> - Hexagonal architecture rules</li>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/steering/tdd.md" rel="noopener noreferrer"><code>.kiro/steering/tdd.md</code></a> - Test-driven development workflow</li>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/.kiro/steering/zod-patterns.md" rel="noopener noreferrer"><code>.kiro/steering/zod-patterns.md</code></a> - Schema-first patterns</li>
</ul>

<p><strong>Implementation (see the code Kiro generated):</strong></p>

<ul>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/domain/schemas/duration.ts" rel="noopener noreferrer"><code>packages/core/src/domain/schemas/duration.ts</code></a> - Discriminated unions with Zod</li>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/application/use-cases/convert-fit-to-krd.ts" rel="noopener noreferrer"><code>packages/core/src/application/use-cases/convert-fit-to-krd.ts</code></a> - Clean use case</li>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/ports/fit-reader.ts" rel="noopener noreferrer"><code>packages/core/src/ports/fit-reader.ts</code></a> - Port interface</li>
<li>
<a href="https://github.com/pablo-albaladejo/kaiord/blob/main/packages/core/src/adapters/fit/garmin-fitsdk.ts" rel="noopener noreferrer"><code>packages/core/src/adapters/fit/garmin-fitsdk.ts</code></a> - Adapter implementation</li>
</ul>




<h2>
  
  
  Final Thoughts
</h2>

<p>Building Kaiord taught me that <strong>AI doesn't replace thinking - it amplifies it</strong>.</p>

<p>With Kiro's spec-driven approach:</p>

<ul>
<li>I think more clearly about what I need</li>
<li>I document decisions before coding</li>
<li>I test continuously (TDD)</li>
<li>I maintain clean architecture</li>
<li>AI implements what I specify</li>
</ul>

<p>The result? Production-ready code that:</p>

<ul>
<li>Works reliably (86.5% test coverage)</li>
<li>Makes sense (clean architecture)</li>
<li>Can grow (extensible design)</li>
<li>Helps others (open source)</li>
</ul>

<p><strong>Spec-driven development isn't slower - it's smarter.</strong> The time you spend planning pays back 10x in code that works the first time.</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<strong>Kaiord Repository:</strong> <a href="https://github.com/pablo-albaladejo/kaiord" rel="noopener noreferrer">github.com/pablo-albaladejo/kaiord</a>
</li>
<li>
<strong>Live Web App:</strong> <a href="https://pablo-albaladejo.github.io/kaiord/" rel="noopener noreferrer">pablo-albaladejo.github.io/kaiord</a>
</li>
<li>
<strong>npm Packages:</strong>

<ul>
<li><a href="https://www.npmjs.com/package/@kaiord/core" rel="noopener noreferrer">@kaiord/core</a></li>
<li><a href="https://www.npmjs.com/package/@kaiord/cli" rel="noopener noreferrer">@kaiord/cli</a></li>
</ul>


</li>

<li>

<strong>Kiro IDE:</strong> <a href="https://kiro.dev" rel="noopener noreferrer">kiro.dev</a>
</li>

<li>

<strong>Kiroween Hackathon:</strong> <a href="https://kiroween.devpost.com/" rel="noopener noreferrer">kiroween.devpost.com</a>
</li>

</ul>




