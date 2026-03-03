---
Title: Closing the Loop: From Audit Violation to AI Fix in One Command
Description: 
Author: Dariusz Newecki
Date: 2026-03-03T21:35:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every developer who uses static analysis tools knows the feeling. You run your linter or audit tool, get a wall of violations, and then spend 10 minutes manually figuring out what to show your AI assistant to fix them.</p>

<p>Copy the file path. Find the relevant class. Construct a prompt. Hope the AI has enough context.</p>

<p>We just eliminated that entire step in CORE.</p>

<h2>
  
  
  The Problem
</h2>

<p>CORE has a constitutional governance system — 85 rules that audit the codebase for violations ranging from architectural boundaries to AI safety patterns. When the audit fails, you get output like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>❌ AUDIT FAILED
  ai.prompt.model_required    │ 38 errors
  architecture.max_file_size  │ 13 warnings
  modularity.single_responsibility │ 11 warnings
</code></pre>

</div>



<p>Useful. But then what? You still have to manually translate "ai.prompt.model_required in src/will/agents/coder_agent.py line 158" into a context package that an AI can actually work with.</p>

<p>That translation is pure mechanical mapping. It shouldn't be human work.</p>

<h2>
  
  
  The Insight
</h2>

<p>Audit tools know <strong>what</strong> is broken and <strong>where</strong>.</p>

<p>Context tools know <strong>what the AI needs to see</strong> to fix it.</p>

<p>The gap between them is just a function call.</p>

<h2>
  
  
  What We Built
</h2>

<h3>
  
  
  Part 1: <code>core-admin context build</code>
</h3>

<p>First, we built a command that simulates exactly what the autonomous CoderAgent sees before generating code. Not a semantic search, not a guess — the exact same pipeline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>core-admin context build <span class="se">\</span>
  <span class="nt">--file</span> src/will/agents/coder_agent.py <span class="se">\</span>
  <span class="nt">--symbol</span> CoderAgent <span class="se">\</span>
  <span class="nt">--task</span> code_modification <span class="se">\</span>
  <span class="nt">--output</span> var/context_for_claude.md
</code></pre>

</div>



<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>================================================================================
CORE CONTEXT PACKAGE  [Agent Simulation Mode]
================================================================================
Target file  : src/will/agents/coder_agent.py
Target symbol: CoderAgent
Task type    : code_modification
Items        : 8
Tokens (est) : ~3644
Build time   : 944ms

### 1. src/shared/context.py::CoreContext
Source : 🔍 vector search / Qdrant (semantic: 0.74)
...

### 6. src/will/agents/coder_agent.py::CoderAgent.build_context_package
Source : 🗄️  DB lookup (direct / graph)
...
</code></pre>

</div>



<p>Every item shows <em>why</em> it was included — AST force-add, DB graph traversal, or vector search with score. You can verify the AI's "view" before it touches anything.</p>

<h3>
  
  
  Part 2: Audit hints
</h3>

<p>Then we added one function to the audit formatter. When the audit fails, it now prints the exact command to run for each actionable finding:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>╭─────────────── AI Workflow — Next Steps ───────────────╮
│ 65 actionable location(s). Run the command below for   │
│ each, then paste the output to Claude.                 │
╰────────────────────────────────────────────────────────╯

  ERROR ai.prompt.model_required
  Line 158: direct call to 'make_request_async()' detected.

  core-admin context build \
      --file src/will/agents/coder_agent.py \
      --task code_modification \
      --output var/context_for_claude.md

  ERROR ai.prompt.output_validation_required
  Line 199: missing mandatory call(s): ['_validate_output']

  core-admin context build \
      --file src/shared/ai/prompt_model.py \
      --symbol _validate_output \
      --task code_modification \
      --output var/context_for_claude.md
</code></pre>

</div>



<p>The audit knows the file. It knows the rule. The mapping to task type is mechanical (<code>ai.*</code> → <code>code_modification</code>, <code>test.*</code> → <code>test_generation</code>). So it just... does it.</p>

<h2>
  
  
  The Complete Workflow
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>core-admin code audit
  → ❌ FAILED
  → 💡 65 actionable locations
  → copy one context build command
  → run it → var/context_for_claude.md
  → paste to Claude
  → fix
  → repeat
</code></pre>

</div>



<p>Zero manual translation. Zero "what should I show the AI?" The audit tells you exactly what to run next.</p>

<h2>
  
  
  The Meta Moment
</h2>

<p>After deploying, we ran the audit again. CORE found modularity debt in the files we had just written:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>WARN  modularity.single_responsibility
src/cli/commands/check/formatters.py  ← file we just edited

WARN  modularity.single_responsibility
src/cli/resources/context/build.py    ← file we just built
</code></pre>

</div>



<p>And immediately printed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>core-admin context build \
    --file src/cli/resources/context/build.py \
    --task code_modification \
    --output var/context_for_claude.md
</code></pre>

</div>



<p>The system audited our work and told us how to fix it. That's the governance model working as designed.</p>

<h2>
  
  
  Why This Matters
</h2>

<p>Most AI coding workflows are: <em>human decides what context to provide → AI generates → human reviews</em>.</p>

<p>The bottleneck is the first step. What context? Which file? Which class? How much?</p>

<p>CORE's constitutional audit already has that information. It found the violation, it knows where it lives. The only missing piece was surfacing it in a form that feeds directly into the next action.</p>

<p>This pattern is general. Any tool that produces findings with file paths and rule IDs can do this. Your linter, your security scanner, your test coverage report — they all know what's broken. The question is whether they tell you what to do next.</p>

<h2>
  
  
  Code
</h2>

<p>CORE is MIT licensed and available at <a href="https://github.com/DariuszNewecki/CORE" rel="noopener noreferrer">github.com/DariuszNewecki/CORE</a>.</p>

<p>The relevant pieces:</p>

<ul>
<li>
<code>src/cli/resources/context/build.py</code> — agent simulation command</li>
<li>
<code>src/cli/commands/check/formatters.py</code> — <code>print_context_build_hints()</code>
</li>
</ul>

<p>The core idea fits in about 60 lines. The <code>_extract_symbol()</code> function tries three approaches in priority order: structured context dict, then message parsing. <code>_infer_task_type()</code> maps rule prefixes to task types. The rest is just formatting.</p>




<p><em>CORE is a constitutional AI governance system — an experiment in building autonomous development tools that remain safely bounded by human-defined constraints. Previous post: <a href="https://dev.toyour-previous-post-link">Building an AI That Follows a Constitution</a></em></p>

<p><em>CORE's PromptModel pattern was inspired by prompt engineering work by <a href="https://substack.com/@ruben" rel="noopener noreferrer">Ruben Hassid</a> — worth following if structured AI invocation interests you.</em></p>

