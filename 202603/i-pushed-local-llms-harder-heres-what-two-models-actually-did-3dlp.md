---
Title: I Pushed Local LLMs Harder. Here's What Two Models Actually Did.
Description: 
Author: Donald Cruver
Date: 2026-03-02T21:51:26.000Z
Robots: noindex,nofollow
Template: index
---
<p>In Part 1 of this series, I set up Claude Code against local LLMs on dual MI60 GPUs and watched it scaffold a Flask application from scratch. Small tasks worked. Complex ones did not. I ended with three ideas I wanted to test: running a dense model, trying Claude Code's agent teams feature, and building a persistent memory layer for coding sessions.</p>

<p>I started the experiment that mattered most: giving a local LLM a project with real scope and seeing what happened. I ran the same project against two different models. The results were instructive, and not in the direction I expected.</p>

<h2>
  
  
  The Project
</h2>

<p>The model's goal was to build a Python CLI tool called health-correlate. It would connect to my InfluxDB health database, retrieve time-series data for metrics like glucose readings, blood ketone levels, blood pressure, body weight, and subjective wellbeing scores, resample everything to daily aggregates, and run Pearson correlation analysis with configurable time-lag support. The output: a ranked table of correlations with p-values. A weather bucket fed from a Weatherflow sensor extended the scope further. The tool could also correlate outdoor conditions against health metrics.</p>

<p>That scope requires several interdependent modules: a data layer for InfluxDB Flux queries, a statistics layer for the correlation math, a CLI layer with multiple subcommands, and a visualization layer. Enough moving parts that the model would need to maintain architectural coherence across files and across iterations.</p>

<p>I structured each session using the subagent pattern. Three Claude Code subagent prompt files handled the phases in sequence: data layer first, then the correlation engine, then the CLI and visualization. Each subagent prompt included a mandatory test-fix loop: write, run, fail, fix, repeat until tests pass before declaring the phase done.</p>

<h2>
  
  
  The Prompt
</h2>

<p>The top-level prompt I gave Claude Code was short. Its job was to delegate work to the subagents, not to do it directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Build health-correlate using Subagents</span>

Build a Python CLI tool that finds correlations between health metrics in InfluxDB.

<span class="gu">## Strategy</span>
Use subagents to keep each phase in its own context window:
<span class="p">
1.</span> <span class="gs">**Phase 1**</span>: Delegate to @phase1-influx to build the InfluxDB data layer
<span class="p">2.</span> <span class="gs">**Phase 2**</span>: Delegate to @phase2-correlation to add statistical analysis
<span class="p">3.</span> <span class="gs">**Phase 3**</span>: Delegate to @phase3-cli to create CLI and visualization

<span class="gu">## Critical Requirement: Test-Fix Loop</span>

Each subagent MUST:
<span class="p">1.</span> Write tests for their code
<span class="p">2.</span> Run tests after each function/module
<span class="p">3.</span> If tests fail → debug and fix → re-test
<span class="p">4.</span> Only report "Done" when ALL tests pass

Do NOT accept a phase as complete if tests are failing.

<span class="gu">## Coordination</span>
<span class="p">-</span> Each subagent writes/updates PROGRESS.md when finished
<span class="p">-</span> Wait for each phase to complete before starting the next
<span class="p">-</span> Verify tests pass before proceeding: <span class="sb">`python -m pytest tests/ -v`</span>
<span class="p">-</span> Do not implement phases yourself - delegate to the subagents

<span class="gu">## Environment</span>
<span class="p">-</span> INFLUXDB_TOKEN is set in the environment
<span class="p">-</span> InfluxDB at influxdb.cruver.network:30086
<span class="p">-</span> Virtual environment at ./venv (activate with <span class="sb">`source venv/bin/activate`</span>)

<span class="gu">## Begin</span>
Start by delegating Phase 1 to @phase1-influx.
After it completes, verify tests pass, then delegate Phase 2.
After Phase 2 completes, verify tests pass, then delegate Phase 3.
</code></pre>

</div>



<p>Each subagent lived in <code>.claude/agents/</code> as a markdown file. The files defined the tools available to the subagent, the specific deliverables, and critically, the test-fix loop requirement. The Phase 1 subagent prompt looked like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="nn">---</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">phase1-influx</span>
<span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Build</span><span class="nv"> </span><span class="s">InfluxDB</span><span class="nv"> </span><span class="s">data</span><span class="nv"> </span><span class="s">layer</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">health-correlate."</span>
<span class="na">tools</span><span class="pi">:</span> <span class="s">Read, Write, Edit, Bash, Glob, Grep</span>
<span class="na">model</span><span class="pi">:</span> <span class="s">inherit</span>
<span class="nn">---</span>

<span class="gh"># Phase 1: InfluxDB Data Layer</span>

...

<span class="gu">## Test-Fix Loop (MANDATORY)</span>

After writing each function:
<span class="p">1.</span> Create a test in tests/test_influx.py
<span class="p">2.</span> Run: <span class="sb">`python -m pytest tests/test_influx.py -v`</span>
<span class="p">3.</span> If test fails:
<span class="p">   -</span> Read the error message carefully
<span class="p">   -</span> Fix the code
<span class="p">   -</span> Re-run the test
<span class="p">   -</span> Repeat until ALL tests pass
<span class="p">4.</span> Do NOT proceed to the next function until current tests pass
</code></pre>

</div>



<p>The pattern repeats across all three phases. The subagent does not consider its work done until tests pass. This is the mechanism that produced the automated bug-fixing: the model was not doing anything clever; it was following the loop.</p>

<h2>
  
  
  Model One: Qwen3-Coder-Next 80B
</h2>

<p>The first model was Qwen3-Coder-Next: 80 billion total parameters, 3 billion active per forward pass, 512 experts with 10 activated per token. I ran an AWQ-quantized version under vLLM across both MI60s in tensor-parallel mode.</p>

<p>The vLLM container command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>--model cyankiwi/Qwen3-Coder-Next-AWQ-4bit
--tensor-parallel-size 2
--max-model-len 65536
--gpu-memory-utilization 0.95
--enable-auto-tool-choice
--tool-call-parser qwen3_coder
</code></pre>

</div>



<p>LiteLLM translated Claude API calls to OpenAI format. Pointing Claude Code at the local stack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">ANTHROPIC_BASE_URL</span><span class="o">=</span><span class="s2">"http://feynman.cruver.network:4000"</span>
<span class="nb">export </span><span class="nv">ANTHROPIC_API_KEY</span><span class="o">=</span><span class="s2">"sk-litellm-local-dev"</span>
</code></pre>

</div>



<p>The first attempt hit a hard wall at 32,768 tokens. Agentic sessions accumulate context fast: every file read, every tool call result, every test output appends to the history. By the time the model was writing the CLI module, the session had consumed 33,839 tokens and vLLM returned a context overflow error.</p>

<p>I bumped the window to 65,536 and ran again. The data layer completed. The correlation engine completed. The test-fix loop worked as designed: the model generated code, ran the tests, read the failures, and corrected them. Five bugs were caught and fixed automatically before either module was signed off.</p>

<p>The CLI phase did not finish. Not from another hard limit, but from what I started calling the context snowball problem. At 65,000 tokens, processing the incoming prompt takes several minutes per turn. The GPUs peg at 100%, but no generation is happening. Local inference does not have attention caching the way cloud providers do. Every turn reprocesses the entire conversation history from scratch. The model was not thinking; it was rereading.</p>

<p>The results from Qwen3-Coder-Next: the architecture was sound, the correlation math was correct, the Flux queries for InfluxDB were valid. The project just ran out of runway before finishing.</p>

<h3>
  
  
  The Tool-Calling Flag That Is Not Optional
</h3>

<p>The <code>--tool-call-parser qwen3_coder</code> flag in vLLM matters. Without it, Qwen3-Coder-Next generates tool calls in a format Claude Code cannot parse. The model produces output that looks like a valid tool call, Claude Code processes it, and nothing happens. No error, no message; the tool call evaporates. Adding the flag fixed it immediately.</p>

<h2>
  
  
  The Model Switch
</h2>

<p>Qwen3.5-35B-A3B was released while the Qwen3-Coder-Next experiment was still in progress. The naming is confusing, but it is a 35 billion parameter MoE with 3 billion active per token. Smaller headline count than Coder-Next, but one difference mattered more for this use case: it runs in GGUF format under llama.cpp, which made 131,072 tokens of context per instance practical.</p>

<p>The bigger change was the GPU architecture. Instead of tensor-parallel across both GPUs for one model instance, the new configuration runs two independent llama-server instances, one per GPU:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">llama-server-0</span><span class="pi">:</span>           <span class="c1"># GPU0: Sonnet tier, no thinking</span>
  <span class="na">environment</span><span class="pi">:</span>
    <span class="s">HIP_VISIBLE_DEVICES=0</span>
  <span class="na">command</span><span class="pi">:</span>
    <span class="s">--ctx-size </span><span class="m">131072</span>
    <span class="s">--flash-attn on</span>
    <span class="s">--jinja</span>
    <span class="s">--reasoning-budget </span><span class="m">0</span>

<span class="na">llama-server-1</span><span class="pi">:</span>           <span class="c1"># GPU1: Opus tier, extended thinking</span>
  <span class="na">environment</span><span class="pi">:</span>
    <span class="s">HIP_VISIBLE_DEVICES=1</span>
  <span class="na">command</span><span class="pi">:</span>
    <span class="s">--ctx-size </span><span class="m">131072</span>
    <span class="s">--flash-attn on</span>
    <span class="s">--jinja</span>
    <span class="s">--reasoning-budget -1</span>
</code></pre>

</div>



<p>LiteLLM routes <code>claude-opus-*</code> requests to GPU1 and everything else to GPU0. Both instances serve the same GGUF file from a shared volume. For agentic coding sessions, which are mostly sequential, one instance handles the full session while the other sits idle for chat.</p>

<p>Two flags turned out to be essential discoveries.</p>

<p>The first is <code>--jinja</code>. Without it, Qwen3.5 tool calls fail silently, the same failure mode as Qwen3-Coder-Next without <code>--tool-call-parser</code>. The Jinja2-based chat template handles the tool call formatting that Claude Code expects. This is the llama.cpp equivalent of vLLM's <code>--tool-call-parser</code> flag. It is not documented prominently.</p>

<p>The second is <code>--dangerously-skip-permissions</code>. For unattended agentic sessions through a remote connection, Claude Code's permission prompts appear in a TUI that is not visible over SSH. The first Qwen3.5 session stalled for sixteen minutes while Claude Code waited for bash approval input. The code was being created, the tests were queued, and the process was sleeping. The flag eliminates the prompts entirely.</p>

<p>One other finding from monitoring the running sessions: Claude Code makes outbound HTTPS connections to Anthropic infrastructure even when <code>ANTHROPIC_BASE_URL</code> points to a local endpoint. Model API calls route correctly to the local LiteLLM proxy. Process authentication and telemetry go to Anthropic's servers directly. A valid Anthropic account is still required; fully offline deployments are not possible without network-level blocking.</p>

<h2>
  
  
  Model Two: Qwen3.5-35B-A3B Results
</h2>

<p>Same project, same subagent prompt files, same InfluxDB target.</p>

<p>Phase 1 (data layer): complete. The model wrote valid Flux queries for the InfluxDB client library on the first attempt. Flux syntax is unusual enough that I expected at least one hallucinated API call. There were none.</p>

<p>Phase 2 (correlation engine): complete. 25 tests passing. Four bugs were caught and fixed by the test-fix loop: a numpy bool versus Python bool type mismatch, a NaN handling order issue before correlation, a sorting column reference error, and a list length mismatch in the test suite. The model found them, diagnosed them, and fixed them without any intervention.</p>

<p>Phase 3 (CLI and visualization): incomplete. The test suite had a structural mock issue: the patch decorators targeted <code>fetch_weather</code> and <code>fetch_metric</code>, but <code>get_influx_client</code> was called first in the execution path, so the mocks never fired. The production code itself was correct. Running the CLI against the live InfluxDB database: <code>list-metrics</code> returned real metric types, <code>correlate glucose ketone</code> computed a real Pearson coefficient, <code>correlate-all</code> ranked glucose against all available weather metrics. The visualization module was not written before the session ended.</p>

<p>One hallucination appeared in Phase 3: <code>df.to_json(default=str)</code>. The <code>default</code> parameter belongs to <code>json.dumps</code>, not <code>DataFrame.to_json</code>. The model conflated two similar APIs. It surfaced immediately under test and would not have survived a type checker either.</p>

<h2>
  
  
  What the Configuration Changes Actually Mean
</h2>

<p>Context window size was the single most significant variable between the two runs. 65,536 tokens is not enough runway for a multi-module Python project built in a single agentic session. 131,072 is enough to mostly finish one, with the subagent pattern distributing the load across isolated context windows.</p>

<p>The GPU split architecture change is also meaningful beyond raw numbers. Tensor-parallel across two GPUs keeps both GPUs active for every request, but introduces inter-GPU communication overhead for each forward pass. Two independent instances run each GPU at full throughput for its own requests, with LiteLLM load-balancing between them. For sequential agentic work, this is a better fit than tensor-parallel.</p>

<p>The context snowball problem does not disappear with a wider window or a different model. It is structural: local inference without attention caching reprocesses the entire history on every turn. The mitigations are a wide enough context ceiling that the snowball does not hit it, and Claude Code's <code>/compact</code> command to compress history mid-session when it does.</p>

<h2>
  
  
  Where Things Stand
</h2>

<p>Qwen3-Coder-Next showed that the agentic loop produces correct code and the subagent architecture handles multi-phase projects. It ran out of context before finishing.</p>

<p>Qwen3.5 showed that with more headroom and the right flags, the same approach produces a mostly functional tool from a clean cold start: valid database queries, correct statistical analysis, a working CLI with real data, and only one meaningful hallucination across three phases.</p>

<p>"Mostly functional" is an honest description. Phase 3 was incomplete. The test suite had fixable issues I had to understand to fix. The visualization module was not written. This is not a one-shot codebase generator. It is a capable assistant that gets significant work done and then needs a human to close the gap.</p>

<p>That is better than where Part 1 left things. For a tool built mostly autonomously from a cold start, it is a more useful outcome than I expected.</p>

<p>The same Qwen3.5 stack turned out to be useful beyond coding sessions. I have been running it as the backend for a fully local AI assistant, handling everything from daily digests to email triage to camera alerts. That is a different kind of test, and it deserves its own post.</p>

