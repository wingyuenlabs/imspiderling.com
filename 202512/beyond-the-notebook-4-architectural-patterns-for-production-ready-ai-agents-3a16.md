---
Title: Beyond the Notebook: 4 Architectural Patterns for Production-Ready AI Agents
Description: 
Author: Fabricio Quagliariello
Date: 2025-12-10T21:57:39.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/google-kaggle-ai-agents-2025-11-10">Google AI Agents Writing Challenge</a>: Learning Reflections</em></p>




<h2>
  
  
  Introduction
</h2>

<p>The gap between a "Hello World" agent running in a Jupyter Notebook and a reliable, production-grade system is not a step—it's a chasm (and it is not an easy one to cross).</p>

<p>I recently had the privilege to participate in the <strong>5-Day AI Agents Intensive Course with Google and Kaggle.</strong> After completing the coursework and finalizing the capstone project, I realized that beyond the many valuable things we enjoyed in this course (very valuable white papers, carefully designed notebooks, and exceptional expert panels in the live sessions), the real treasure wasn't just learning the ADK syntax—it was the <strong>architectural patterns</strong> subtly embedded within the lessons. </p>

<p>As an architect building production systems for over 20 years, including multi-agent workflows and enterprise integrations, I've seen firsthand where theoretical agents break under real-world constraints.</p>

<p>We are moving from an era of "prompt engineering" to "agent architecture" where "context engineering" is key. As with any other emerging architectural paradigm, this shift demands blueprints that ensure reliability, efficiency, and ethical safety. Without them, we risk agents that silently degrade, violate user privacy, or execute irreversible actions without oversight.</p>

<p>Drawing from the course and my own experience as an AI Architect, I have distilled the curriculum into four essential patterns that transform fragile prototypes into robust production systems:</p>

<h3>
  
  
  The 4 Core Patterns
</h3>

<ol>
<li>
<strong>Outside-In Evaluation Hierarchy:</strong> Shifting focus from the final answer to the decision-making trajectory.</li>
<li>
<strong>Dual-Layer Memory Architecture:</strong> Balancing ephemeral session context with persistent, consolidated knowledge.</li>
<li>
<strong>Protocol-First Interoperability:</strong> Decoupling agents from tools using standardized protocols like MCP and A2A.</li>
<li>
<strong>Long-Running Operations &amp; Resumability:</strong> Managing state for asynchronous tasks and human-in-the-loop workflows.</li>
</ol>

<p>Throughout this analysis, I'll apply a <strong>6-point framework</strong> grounded in the principles of <strong>Principia Agentica</strong>—ensuring these patterns respect human sovereignty, fiduciary responsibility, and meaningful human control.</p>

<h3>
  
  
  The Analysis Framework
</h3>

<ol>
<li>
<strong>The Production Problem:</strong> Why naive approaches fail at scale.</li>
<li>
<strong>The Architectural Solution:</strong> The specific design pattern taught in the course.</li>
<li>
<strong>Key Implementation Details:</strong> Concrete code-level insights from the ADK notebooks.</li>
<li>
<strong>Production Considerations:</strong> Real-world deployment implications (latency, cost, scale).</li>
<li>
<strong>Connection to Ethical Design:</strong> How the pattern supports human sovereignty, fiduciary responsibility, or ethical agent architecture. I will include a "failure scenario" where I'll try to illustrate what could happen without the ethical safeguard.</li>
<li>
<strong>Key Takeaways:</strong> A distilled summary of each pattern's production principle, implementation guidance, and ethical anchor—designed to serve as a quick reference for architects moving from prototype to production.</li>
</ol>

<p>Let's do this!</p>




<h2>
  
  
  Pattern 1: Outside-In Evaluation Hierarchy (Trajectory as Truth)
</h2>

<p>In traditional software, if the output is correct, the test passes. In agentic AI, a correct answer derived from a hallucination or a dangerous logic path is a ticking time bomb.</p>

<h3>
  
  
  1. The Production Problem
</h3>

<p>Naive evaluation strategies often fail in production due to the non-deterministic nature of LLMs. We face two specific traps:</p>

<ul>
<li>
<strong>The "Lucky Guess" Trap:</strong> Imagine an agent asked to "Get the weather in Tokyo." A bad agent might hallucinate "It is sunny in Tokyo" without calling the weather tool. If it happens to be sunny, a traditional <code>assert result == expected</code> test passes. This hides a critical failure in logic that will break as soon as the weather changes.</li>
<li>
<strong>The "Silent Failure" of Efficiency:</strong> An agent might solve a user request but take 25 steps to do what should have taken 3. This bloats token costs and latency—a failure mode that boolean output checks completely miss.</li>
</ul>

<h3>
  
  
  2. The Architectural Solution
</h3>

<p>Day 4 of the course introduced the concept of <strong>Glass Box Evaluation</strong>. We move away from simple output verification to a hierarchical approach:</p>

<ol>
<li>
<strong>Level 1: Black Box (End-to-End):</strong> Did the user get the right result?</li>
<li>
<strong>Level 2: Glass Box (Trajectory):</strong> Did the agent use the correct tools in the correct order?</li>
<li>
<strong>Level 3: Component (Unit):</strong> Did the individual tools perform as expected?</li>
</ol>

<p>This shift treats the <strong>trajectory</strong> (Thought → Action → Observation) as the unit of truth. By evaluating the trajectory, we ensure the agent isn't just "getting lucky," but is actually <em>reasoning</em> correctly.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkltnk27rcr2v6gsp5xd9.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkltnk27rcr2v6gsp5xd9.webp" alt="pattern1_1" width="800" height="784"></a></p>

<h3>
  
  
  3. Implementation Details: Field Notes from the ADK
</h3>

<p>The ADK provides specific primitives to capture and score these trajectories without writing custom parsers for every test.</p>

<p><strong>From <code>adk web</code> to <code>evalset.json</code></strong><br>
Instead of manually writing test cases, the ADK encourages a "Capture and Replay" workflow. During development (using <code>adk web</code>), when you spot a successful interaction, you can persist that session state. This generates an <code>evalset.json</code> that captures not just the input/output, but the <em>expected tool calls</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">Conceptual</span><span class="w"> </span><span class="err">structure</span><span class="w"> </span><span class="err">of</span><span class="w"> </span><span class="err">an</span><span class="w"> </span><span class="err">ADK</span><span class="w"> </span><span class="err">evalset</span><span class="w"> </span><span class="err">entry</span><span class="w">
</span><span class="err">//</span><span class="w"> </span><span class="err">Traditional</span><span class="w"> </span><span class="err">test:</span><span class="w"> </span><span class="err">just</span><span class="w"> </span><span class="err">input/output</span><span class="w">
</span><span class="err">//</span><span class="w"> </span><span class="err">ADK</span><span class="w"> </span><span class="err">evalset</span><span class="w"> </span><span class="err">contains</span><span class="w"> </span><span class="err">evalcases</span><span class="w"> </span><span class="err">with</span><span class="w"> </span><span class="err">invocations:</span><span class="w"> </span><span class="err">input</span><span class="w"> </span><span class="err">(queries)</span><span class="w"> </span><span class="err">+</span><span class="w"> </span><span class="err">expected_tool_use</span><span class="w"> </span><span class="err">+</span><span class="w"> </span><span class="err">reference</span><span class="w"> </span><span class="err">(output)</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ask_GOOGLE_price"</span><span class="p">,</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">a</span><span class="w"> </span><span class="err">given</span><span class="w"> </span><span class="err">name</span><span class="w"> </span><span class="err">of</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">evaluation</span><span class="w"> </span><span class="err">set</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">evaluation</span><span class="w"> </span><span class="err">cases</span><span class="w"> </span><span class="err">are</span><span class="w"> </span><span class="err">included</span><span class="w"> </span><span class="err">here</span><span class="w">
      </span><span class="s2">"query"</span><span class="err">:</span><span class="w"> </span><span class="s2">"What is the stock price of GOOG?"</span><span class="p">,</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">user</span><span class="w"> </span><span class="err">input</span><span class="w">
      </span><span class="s2">"reference"</span><span class="err">:</span><span class="w"> </span><span class="s2">"The price is $175..."</span><span class="p">,</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">expected</span><span class="w"> </span><span class="err">semantic</span><span class="w"> </span><span class="err">output</span><span class="w">
      </span><span class="s2">"expected_tool_use"</span><span class="err">:</span><span class="w"> </span><span class="p">[</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">expected</span><span class="w"> </span><span class="err">trajectory</span><span class="w">
        </span><span class="p">{</span><span class="w"> 
            </span><span class="nl">"tool_name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"get_stock_price"</span><span class="p">,</span><span class="w"> 
            </span><span class="nl">"tool_input"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">arguments</span><span class="w"> </span><span class="err">passed</span><span class="w"> </span><span class="err">to</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">tool</span><span class="w">
                </span><span class="nl">"symbol"</span><span class="p">:</span><span class="w"> </span><span class="s2">"GOOG"</span><span class="w"> 
            </span><span class="p">}</span><span class="w"> 
        </span><span class="p">}</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="err">//</span><span class="w"> </span><span class="err">other</span><span class="w"> </span><span class="err">evaluation</span><span class="w"> </span><span class="err">cases</span><span class="w"> </span><span class="err">...</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"initial_session"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"state"</span><span class="p">:</span><span class="w"> </span><span class="p">{},</span><span class="w">
    </span><span class="nl">"app_name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hello_world"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"user_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user_..."</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">specific</span><span class="w"> </span><span class="err">id</span><span class="w"> </span><span class="err">of</span><span class="w"> </span><span class="err">the</span><span class="w"> </span><span class="err">user</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">

</span></code></pre>

</div>



<p>This JSON represents an <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/eval_set.py#L22" rel="noopener noreferrer"><code>EvalSet</code></a> containing one <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/eval_case.py#L132" rel="noopener noreferrer"><code>EvalCase</code></a>. Each <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/eval_case.py#L132" rel="noopener noreferrer"><code>EvalCase</code></a> has a <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/models/gemma_llm.py#L157" rel="noopener noreferrer"><code>name</code></a>, <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/cli/browser/polyfills-B6TNHZQ6.js#L1" rel="noopener noreferrer"><code>data</code></a> (which is a list of invocations), and an optional <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/agent_evaluator.py#L233" rel="noopener noreferrer"><code>initial_session</code></a>. Each invocation within the <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/cli/browser/polyfills-B6TNHZQ6.js#L1" rel="noopener noreferrer"><code>data</code></a> list includes a <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/contributing/samples/crewai_tool_kwargs/agent.py#L33" rel="noopener noreferrer"><code>query</code></a>, <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/evaluation_constants.py#L20" rel="noopener noreferrer"><code>expected_tool_use</code></a>, <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/local_eval_sets_manager.py#L55" rel="noopener noreferrer"><code>expected_intermediate_agent_responses</code></a>, and a <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/evaluation_constants.py#L22" rel="noopener noreferrer"><code>reference</code></a> response. </p>

<p>The <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/eval_set.py#L22" rel="noopener noreferrer"><code>EvalSet</code></a> object itself also includes <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/tests/unittests/evaluation/test_in_memory_eval_sets_manager.py#L34" rel="noopener noreferrer"><code>eval_set_id</code></a>, <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/models/gemma_llm.py#L157" rel="noopener noreferrer"><code>name</code></a>, <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/agents/base_agent.py#L117" rel="noopener noreferrer"><code>description</code></a>, <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/cli/adk_web_server.py#L933" rel="noopener noreferrer"><code>eval_cases</code></a>, and <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/evaluation/eval_set.py#L38" rel="noopener noreferrer"><code>creation_timestamp</code></a> fields. </p>

<p><strong>Configuring the Judge</strong></p>

<p>In the <code>test_config.json</code>, we can move beyond simple string matching. The course demonstrated configuring <strong>LLM-as-a-Judge</strong> evaluators.</p>

<ul>
<li>
<strong>Naive Approach:</strong> Uses an exact match evaluator (brittle, fails on phrasing differences).</li>
<li>
<strong>Architectural Approach:</strong> Uses <code>TrajectoryEvaluator</code> alongside <code>SemanticSimilarity</code>. The ADK allows us to define "Golden Sets" where the <em>reasoning path</em> is the standard, allowing the LLM judge to penalize agents that skip steps or hallucinate data, even if the final text looks plausible.</li>
</ul>

<p><strong>Core Configuration Components</strong></p>

<p>To configure an LLM-as-a-Judge effectively, you must construct a specific input payload with four components:</p>

<ol>
<li>
<strong>The Agent's Output:</strong> The actual response generated by the agent you are testing.</li>
<li>
<strong>The Original Prompt:</strong> The specific instruction or query the user provided.</li>
<li>
<strong>The "Golden" Answer:</strong> A reference answer or ground truth to serve as a benchmark.</li>
<li>
<strong>A Detailed Evaluation Rubric:</strong> Specific criteria (e.g., "Rate helpfulness on a scale of 1-5") and requirements for the judge to explain its reasoning.</li>
</ol>

<p><strong>ADK Default Evaluators</strong></p>

<p>The ADK Evaluation Framework includes several default evaluators, accessible via the <code>MetricEvaluatorRegistry</code>:</p>

<ul>
<li>
<strong><code>RougeEvaluator</code></strong>: Uses the ROUGE-1 metric to score similarity between an agent's final response and a golden response.</li>
<li>
<strong><code>FinalResponseMatchV2Evaluator</code></strong>: Uses an LLM-as-a-judge approach to determine if an agent's response is valid.</li>
<li>
<strong><code>TrajectoryEvaluator</code></strong>: Assesses the accuracy of an agent's tool use trajectories by comparing the sequence of tool calls against expected calls. It supports various match types (<code>EXACT</code>, <code>IN_ORDER</code>, <code>ANY_ORDER</code>).</li>
<li>
<strong><code>SafetyEvaluatorV1</code></strong>: Assesses the safety (harmlessness) of an agent's response, delegating to Vertex Gen AI Eval SDK.</li>
<li>
<strong><code>HallucinationsV1Evaluator</code></strong>: Checks if a model response contains any false, contradictory, or unsupported claims by segmenting the response into sentences and validating each against the provided context.</li>
<li>
<strong><code>RubricBasedFinalResponseQualityV1Evaluator</code></strong>: Assesses the quality of an agent's final response against user-defined rubrics, using an LLM as a judge.</li>
<li>
<strong><code>RubricBasedToolUseV1Evaluator</code></strong>: Assesses the quality of an agent's tool usage against user-defined rubrics, employing an LLM as a judge.</li>
</ul>

<p>These evaluators can be configured using <code>EvalConfig</code> objects, which specify the criteria and thresholds for assessment.</p>

<p><strong>Bias Mitigation Strategies</strong></p>

<p>A major challenge is handling bias, such as the tendency for models to give average scores or prefer the first option presented:</p>

<ul>
<li>
<strong>Pairwise Comparison (A/B Testing):</strong> Instead of asking for an absolute score, configure the judge to compare two different responses (Answer A vs. Answer B) and force a choice. This yields a "win rate," which is often a more reliable signal.</li>
<li>
<strong>Swapping Operation:</strong> To counter <strong>position bias</strong>, invoke the judge twice, swapping the order of the candidates. If the results are inconsistent, the result can be labeled as a "tie".</li>
<li>
<strong>Rule Augmentation:</strong> Embed specific evaluation principles, references, and rubrics directly into the judge's system prompt.</li>
</ul>

<p><strong>Advanced Configuration: Agent-as-a-Judge</strong></p>

<p>There's a distinction between standard <strong>LLM-as-a-Judge</strong> (which evaluates final text outputs) and <strong>Agent-as-a-Judge</strong>:</p>

<ul>
<li>
<strong>Standard LLM-as-a-Judge:</strong> Best for evaluating the final response (e.g., "Is this summary accurate?").</li>
<li>
<strong>Agent-as-a-Judge:</strong> Necessary when you need to evaluate the <strong>process</strong>, not just the result. You configure the judge to ingest the agent's full <strong>execution trace</strong> (including internal thoughts, tool calls, and tool arguments). This allows the judge to assess intermediate steps, such as whether the correct tool was chosen or if the plan was logically structured.</li>
</ul>

<p><strong>Evaluation Architectures</strong></p>

<p>You can use several architectural approaches when configuring your judge:</p>

<ul>
<li>
<strong>Point-wise:</strong> The judge evaluates a single candidate in isolation.</li>
<li>
<strong>Pair-wise / List-wise:</strong> The judge compares two or more candidates simultaneously to produce a ranking.</li>
<li>
<strong>Multi-Agent Collaboration:</strong> For high-stakes evaluation, you can configure multiple LLM judges to debate or vote (e.g., "Peer Rank" algorithms) to produce a final consensus, rather than relying on a single model.</li>
</ul>

<p><strong>Example Configuration</strong></p>

<p>For a pairwise comparison judge, structure the prompt to output in a structured JSON format:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"winner"</span><span class="p">:</span><span class="w"> </span><span class="s2">"A"</span><span class="p">,</span><span class="w"> </span><span class="err">//</span><span class="w"> </span><span class="err">or</span><span class="w"> </span><span class="s2">"B"</span><span class="w"> </span><span class="err">or</span><span class="w"> </span><span class="s2">"tie"</span><span class="w">
  </span><span class="nl">"rationale"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Answer A provided more specific delivery details..."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This structured output allows you to programmatically parse the judge's decision and calculate metrics like win/loss rates at scale.</p>

<p><strong>Analogy</strong></p>

<p>You can think of configuring an <strong>LLM-as-a-Judge</strong> like setting up a <strong>blind taste test</strong>. If you just hand a judge a cake and ask "Is this good?", they might be polite and say "Yes." But if you provide them with a <strong>Golden Answer</strong> (a cake baked by a master chef) and use <strong>Pairwise Comparison</strong> (ask "Which of these two is better?"), you force them to make a critical distinction, resulting in far more accurate and actionable feedback.</p>

<h3>
  
  
  4. Production Considerations
</h3>

<p>Moving this pattern from a notebook to a live system requires handling scale and cost.</p>

<ul>
<li>
<strong>Dynamic Sampling:</strong> You cannot trace and judge every single production interaction with an LLM—it’s too expensive. A robust pattern is <strong>100/10 sampling</strong>: capture 100% of traces that result in user errors or negative feedback, but only sample 10% of successful sessions to monitor for latency drift (P99) and token bloat.</li>
<li>
<strong>The Evaluation Flywheel:</strong> Evaluation isn't a one-time gate before launch. Production traces (captured via OpenTelemetry) must be fed back into the development cycle. Every time an agent fails in production, that specific trajectory should be anonymized and added to the <code>evalset.json</code> as a regression test.</li>
<li>
<strong>Latency Impact:</strong> Trajectory logging must be asynchronous. The user should receive their response immediately, while the trace data is pushed to the observability store (like LangSmith or a custom SQL db) in a background thread to avoid degrading the user experience.</li>
</ul>

<h3>
  
  
  5. Ethical Connection
</h3>

<p><strong>"The Trajectory is the Truth"</strong> is the technical implementation of <strong>Fiduciary Responsibility</strong>. We cannot claim an agent is acting in the user's best interest if we only validate the <em>result</em> (the "what") while ignoring the <em>process</em> (the "how"). We must ensure the agent isn't achieving the right ends through manipulative, inefficient, or unethical means.</p>

<p><strong>Concrete Failure Scenario:</strong></p>

<p>Consider a hiring agent that filters job candidates. Without trajectory validation, it could discriminate based on protected characteristics (age, gender, ethnicity) during the filtering process, yet pass all output tests by producing a "diverse" final shortlist through cherry-picking. The bias hides in the <em>how</em>—which resumes were read, which criteria were weighted, which candidates were never considered. Output validation alone cannot detect this algorithmic discrimination. Only trajectory evaluation exposes the unethical reasoning path.</p>

<h3>
  
  
  Key Takeaways
</h3>

<ul>
<li>
<strong>Production Principle:</strong> Trust the reasoning process, not just the output. Trajectory validation is the difference between lucky guesses and reliable intelligence.</li>
<li>
<strong>Implementation:</strong> Use ADK's <code>TrajectoryEvaluator</code> with <code>EvalSet</code> objects to capture expected tool calls alongside expected outputs. Configure LLM-as-a-Judge with Golden Sets and pairwise comparison to avoid evaluation bias.</li>
<li>
<strong>Ethical Anchor:</strong> This pattern operationalizes <strong>Fiduciary Responsibility</strong>—we validate that the agent serves the user's interests through sound reasoning, not through shortcuts, hallucinations, or hidden bias.</li>
</ul>




<p>Validating the <em>how</em> is critical, but what happens when the reasoning path spans not just one conversation turn, but weeks or months? An agent that reasons correctly in the moment can still fail catastrophically if it forgets what it learned yesterday. This brings us to our second pattern: managing the agent's memory architecture.</p>

<h2>
  
  
  Pattern 2: Dual-Layer Memory Architecture (Session vs. Memory)
</h2>

<h3>
  
  
  1. The Production Problem
</h3>

<p>Although models like Gemini 1.5 have introduced massive context windows, treating context as infinite is an architectural anti-pattern.</p>

<p>Consider a <strong>Travel Agent Bot</strong>: In Session 1, the user mentions a "shellfish allergy." By Session 10, months later, that critical fact is buried under thousands of tokens of hotel searches and flight comparisons</p>

<p>This might lead to two very concrete failures:</p>

<ul>
<li>
<strong>Context Rot:</strong> As the context window fills with noise, the model's ability to attend to specific, older instructions (like the allergy) degrades.</li>
<li>
<strong>Cost Spiral:</strong> Re-sending the entire history of every past interaction for every new query creates a linear cost increase that makes the system economically unviable at scale.</li>
</ul>

<h3>
  
  
  2. The Architectural Solution
</h3>

<p>We must distinguish between the <strong>Workbench</strong> and the <strong>Filing Cabinet</strong>.</p>

<ul>
<li>
<strong>The Session (Workbench):</strong> An ephemeral, mutable space for the current task. It holds the immediate "Hot Path" context. To keep it performant, we apply <strong>Context Compaction</strong>—automatically summarizing or truncating older turns while keeping the most recent ones raw.</li>
<li>
<strong>The Memory (Filing Cabinet):</strong> A persistent layer for consolidated facts. This requires an ETL (Extract, Transform, Load) pipeline where the agent <em>Extracts</em> facts from the session, <em>Consolidates</em> them (deduplicating against existing knowledge), and <em>Stores</em> them for semantic retrieval later.</li>
</ul>

<h3>
  
  
  3. Implementation Details: Code Insights
</h3>

<p>The ADK moves memory management from manual implementation to configuration.</p>

<p><strong>Session Hygiene via Compaction</strong><br>
In the ADK, we don't manually trim strings. We configure the agent to handle its own hygiene using <code>EventsCompactionConfig</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google.adk.agents.base_agent</span> <span class="kn">import</span> <span class="n">BaseAgent</span>
<span class="kn">from</span> <span class="n">google.adk.apps.app</span> <span class="kn">import</span> <span class="n">App</span><span class="p">,</span> <span class="n">EventsCompactionConfig</span>
<span class="kn">from</span> <span class="n">google.adk.apps.llm_event_summarizer</span> <span class="kn">import</span> <span class="n">LlmEventSummarizer</span> <span class="c1"># Assuming this is your summarizer
</span>
<span class="c1"># Define a simple BaseAgent for the example
</span><span class="k">class</span> <span class="nc">MyAgent</span><span class="p">(</span><span class="n">BaseAgent</span><span class="p">):</span>
    <span class="n">name</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="sh">"</span><span class="s">my_agent</span><span class="sh">"</span>
    <span class="n">description</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="sh">"</span><span class="s">A simple agent.</span><span class="sh">"</span>

    <span class="k">def</span> <span class="nf">call</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">context</span><span class="p">,</span> <span class="n">content</span><span class="p">):</span>
        <span class="k">pass</span>

<span class="c1"># Create an instance of LlmEventSummarizer or your custom summarizer
</span><span class="n">my_summarizer</span> <span class="o">=</span> <span class="nc">LlmEventSummarizer</span><span class="p">()</span>

<span class="c1"># Create an EventsCompactionConfig
</span><span class="n">events_compaction_config_instance</span> <span class="o">=</span> <span class="nc">EventsCompactionConfig</span><span class="p">(</span>
    <span class="n">summarizer</span><span class="o">=</span><span class="n">my_summarizer</span><span class="p">,</span>
    <span class="n">compaction_interval</span><span class="o">=</span><span class="mi">5</span><span class="p">,</span>
    <span class="n">overlap_size</span><span class="o">=</span><span class="mi">2</span>
<span class="p">)</span>

<span class="c1"># Create an App instance with the EventsCompactionConfig
</span><span class="n">my_app</span> <span class="o">=</span> <span class="nc">App</span><span class="p">(</span>
    <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">my_application</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">root_agent</span><span class="o">=</span><span class="nc">MyAgent</span><span class="p">(),</span>
    <span class="n">events_compaction_config</span><span class="o">=</span><span class="n">events_compaction_config_instance</span>
<span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="n">my_app</span><span class="p">.</span><span class="nf">model_dump_json</span><span class="p">(</span><span class="n">indent</span><span class="o">=</span><span class="mi">2</span><span class="p">))</span>

</code></pre>

</div>



<p><strong>Persistence: From RAM to DB</strong><br>
In notebooks, we often use <code>InMemorySessionService</code>. This is dangerous for production because a container restart wipes the conversation. The architectural shift is moving to <code>DatabaseSessionService</code> (backed by SQL or Firestore) which persists the <code>Session</code> object state, allowing users to resume conversations across devices.</p>

<p><strong>The Memory Consolidation Pipeline</strong><br>
Day 3b introduced the framework for moving from raw storage to intelligent consolidation. This is where the "Filing Cabinet" becomes smart. The workflow is an <strong>LLM-driven ETL pipeline</strong> with four stages:</p>

<ol>
<li>
<strong>Ingestion:</strong> The system receives raw session history.</li>
<li>
<p><strong>Extraction &amp; Filtering:</strong> An LLM analyzes the conversation to extract meaningful facts, guided by developer-defined <strong>Memory Topics</strong>:<br>
The LLM extracts <em>only</em> facts matching these topics.<br>
</p>

<pre class="highlight python"><code><span class="c1"># Conceptual configuration (Vertex AI Memory Bank, Day 5)
</span><span class="n">memory_topics</span> <span class="o">=</span> <span class="p">[</span>
  <span class="sh">"</span><span class="s">user_preferences</span><span class="sh">"</span><span class="p">,</span>      <span class="c1"># "Prefers window seats"
</span>  <span class="sh">"</span><span class="s">dietary_restrictions</span><span class="sh">"</span><span class="p">,</span>  <span class="c1"># "Allergic to shellfish"
</span>  <span class="sh">"</span><span class="s">project_context</span><span class="sh">"</span>        <span class="c1"># "Leading Q4 marketing campaign"
</span><span class="p">]</span>

</code></pre>




</li>

<li>

<p><strong>Consolidation (The "Transform" Phase):</strong> The LLM retrieves existing memories and decides:</p>

<ul>
<li>
<strong>CREATE:</strong> Novel information → new memory entry.</li>
<li>
<strong>UPDATE:</strong> New info refines existing memory → merge (e.g., "Likes marketing" becomes "Leading Q4 marketing project").</li>
<li>
<strong>DELETE:</strong> New info contradicts old → invalidate (e.g., Dietary restrictions change).</li>
</ul>


</li>

<li><p><strong>Storage:</strong> Consolidated memories persist to a vector database for semantic retrieval.</p></li>

</ol>

<p><em>Note: While Day 3b uses <code>InMemoryMemoryService</code> to teach the API, it stores raw events without consolidation. For production-grade consolidation, we look to the Vertex AI Memory Bank integration introduced in Day 5.</em></p>

<p><strong>Retrieval Strategies: Proactive vs. Reactive</strong><br>
The course highlighted two distinct patterns for getting data <em>out</em> of the Filing Cabinet:</p>

<ol>
<li>
<strong>Proactive (<code>preload_memory</code>):</strong> Injects relevant user facts into the system prompt <em>before</em> the model generates a response. Best for high-frequency preferences (e.g., "User always prefers aisle seats").</li>
<li>
<strong>Reactive (<code>load_memory</code>):</strong> Gives the agent a tool to search the database. The agent decides <em>if</em> it needs to look something up. Best for obscure facts to save tokens.</li>
</ol>
<h3>
  
  
  4. Production Considerations
</h3>

<ul>
<li>
<strong>Asynchronous Consolidation:</strong> Moving data from the Workbench to the Filing Cabinet is expensive. In production, this ETL process should happen <strong>asynchronously</strong>. Do not make the user wait for the agent to "file its paperwork." Trigger the memory extraction logic in a background job after the session concludes.</li>
<li>
<strong>Semantic Search:</strong> Keyword search is insufficient for the Filing Cabinet. Production memory requires vector embeddings. If a user asks for "romantic dining," the system must be able to retrieve a past note about "candlelight dinners," even if the word "romantic" wasn't used.</li>
<li>
<strong>The "Context Stuffing" Trade-off:</strong> While <code>preload_memory</code> reduces latency (no extra tool roundtrip), it increases input token costs on every turn. <code>load_memory</code> is cheaper on average but adds latency when retrieval is needed.</li>
</ul>
<h3>
  
  
  5. Ethical Design Note
</h3>

<p>This architecture embodies <strong>Privacy by Design</strong>. By distinguishing between the transient session and persistent memory, we can implement rigorous "forgetting" protocols.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rpwqy62zpbieauqys90.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0rpwqy62zpbieauqys90.webp" alt="Pattern 2" width="800" height="191"></a></p>

<p>We scrub Personally Identifiable Information (PII) from the session log <em>before</em> it undergoes consolidation into long-term memory, ensuring we act as fiduciaries of user data rather than creating an unmanageable surveillance log.</p>

<p><strong>Concrete Failure Scenario:</strong></p>

<p>Imagine a healthcare agent that remembers a patient mentioned their HIV status in Session 1. Without a dual-layer architecture, this fact sits in plain text in the session log forever, accessible to any system with database read permissions. If the system is breached, or if a support engineer needs to debug a session, the patient's private health information is exposed. Worse, without consolidation logic, the system doesn't know to <em>delete</em> this information if the patient later says "I was misdiagnosed—I don't have HIV." The agent treats every utterance as equally permanent, creating a privacy nightmare where sensitive data proliferates uncontrollably across logs and backups.</p>
<h3>
  
  
  Key Takeaways
</h3>

<ul>
<li>
<strong>Production Principle:</strong> Context is expensive, but privacy is priceless. Design memory systems that distinguish between what an agent needs now (hot session) and what it needs forever (consolidated memory).</li>
<li>
<strong>Implementation:</strong> Use <code>EventsCompactionConfig</code> for session hygiene and implement a PII scrubber in your ETL pipeline before consolidation. Leverage Vertex AI Memory Bank for production-grade semantic memory with built-in privacy controls.</li>
<li>
<strong>Ethical Anchor:</strong> This pattern operationalizes <strong>Privacy by Design</strong>—we build forgetfulness and data minimization into the architecture, treating user data as a liability to protect, not an asset to hoard.</li>
</ul>



<p>With robust evaluation validating our agent's reasoning and a dual-layer memory preserving context over time, we might assume our system is production-ready. But there's a hidden fragility: these capabilities are only as good as the tools and data sources the agent can access. When every integration is a bespoke API wrapper, scaling becomes a maintenance nightmare. This brings us to the third pattern: decoupling agents from their dependencies through standardized protocols.</p>
<h2>
  
  
  Pattern 3: Protocol-First Interoperability (MCP &amp; A2A)
</h2>
<h3>
  
  
  1. The Production Problem
</h3>

<p>We are facing an <strong>"N×M Integration Trap."</strong></p>

<p>Imagine building a Customer Support Agent. It needs to check <strong>GitHub</strong> for bugs, message <strong>Slack</strong> for alerts, and update <strong>Jira</strong> tickets. Without a standard protocol, you write three custom API wrappers. When GitHub changes an endpoint, your agent breaks.</p>

<p>Now, multiply this across an enterprise. You have 10 different agents needing access to 20 different data sources. You are suddenly maintaining 200 brittle integration points. Furthermore, these agents become <strong>isolated silos</strong>—the Sales Agent has no way to dynamically discover or ask the Engineering Agent for help because they speak different "languages."</p>
<h3>
  
  
  2. The Architectural Solution
</h3>

<p>The solution is to invert the dependency. Instead of the agent knowing about the specific tool implementation, we adopt a <strong>Protocol-First Architecture</strong>.</p>

<ul>
<li>
<strong>Model Context Protocol (MCP):</strong> For <strong>Tools and Data</strong>. It decouples the agent (client) from the tool (server). The agent doesn't need to know <em>how</em> to query a Postgres DB; it just needs to know the MCP interface to ask for data.</li>
<li>
<strong>Agent2Agent (A2A):</strong> For <strong>Peers and Delegation</strong>. It allows for high-level goal delegation. An agent doesn't execute a task; it hands off a goal to another agent via a standardized handshake.</li>
<li>
<strong>Runtime Discovery:</strong> Instead of hardcoding tools, agents query an MCP Server or an Agent Card at runtime to discover capabilities dynamically.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3p8gnabnvxbb7d91d3cy.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3p8gnabnvxbb7d91d3cy.webp" alt="Pattern 3" width="800" height="547"></a></p>
<h3>
  
  
  3. Implementation Details: Code Examples from the ADK
</h3>

<p>The ADK abstracts the heavy lifting of these protocols.</p>

<p><strong>Connecting Data via MCP</strong><br>
We don't write API wrappers. We instantiate an <code>McpToolset</code>. The ADK handles the handshake, lists the available tools, and injects their schemas into the context window automatically.</p>

<p>The <strong>Model Context Protocol (MCP)</strong> is used to connect an agent to external tools and data sources without writing custom API clients. In ADK, we use <code>McpToolset</code> to wrap an MCP server configuration.</p>

<p><strong>Example: Connecting an agent to the "Everything" MCP server:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google.adk.agents</span> <span class="kn">import</span> <span class="n">LlmAgent</span>
<span class="kn">from</span> <span class="n">google.adk.tools</span> <span class="kn">import</span> <span class="n">McpToolset</span>
<span class="kn">from</span> <span class="n">google.adk.tools.mcp_tool.mcp_session_manager</span> <span class="kn">import</span> <span class="n">StdioConnectionParams</span>
<span class="kn">from</span> <span class="n">google.adk.tools.mcp_tool.mcp_session_manager</span> <span class="kn">import</span> <span class="n">StdioServerParameters</span>
<span class="kn">from</span> <span class="n">google.adk.runners</span> <span class="kn">import</span> <span class="n">Runner</span> <span class="c1"># Assuming Runner is defined elsewhere
</span>
<span class="c1"># 1. Define the connection to the MCP Server
# Here we use 'npx' to run a Node-based MCP server directly
</span><span class="n">mcp_toolset</span> <span class="o">=</span> <span class="nc">McpToolset</span><span class="p">(</span>
    <span class="n">connection_params</span><span class="o">=</span><span class="nc">StdioConnectionParams</span><span class="p">(</span>
        <span class="n">server_params</span><span class="o">=</span><span class="nc">StdioServerParameters</span><span class="p">(</span>
            <span class="n">command</span><span class="o">=</span><span class="sh">"</span><span class="s">npx</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">-y</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">@modelcontextprotocol/server-everything</span><span class="sh">"</span><span class="p">]</span>
        <span class="p">),</span>
        <span class="n">timeout</span><span class="o">=</span><span class="mf">10.0</span> <span class="c1"># Optional: specify a timeout for connection establishment
</span>    <span class="p">),</span>
    <span class="c1"># Optionally filter to specific tools provided by the server
</span>    <span class="n">tool_filter</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">getTinyImage</span><span class="sh">"</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># 2. Add the MCP tools to your Agent
</span><span class="n">agent</span> <span class="o">=</span> <span class="nc">LlmAgent</span><span class="p">(</span>
    <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">image_agent</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gemini-2.0-flash</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">instruction</span><span class="o">=</span><span class="sh">"</span><span class="s">You can generate tiny images using the tools provided.</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># The toolset exposes the MCP capabilities as standard ADK tools
</span>    <span class="n">tools</span><span class="o">=</span><span class="p">[</span><span class="n">mcp_toolset</span><span class="p">]</span> <span class="c1"># tools expects a list of ToolUnion
</span><span class="p">)</span>

<span class="c1"># 3. Run the agent
# The agent can now call 'getTinyImage' as if it were a local Python function
</span><span class="n">runner</span> <span class="o">=</span> <span class="nc">Runner</span><span class="p">(</span><span class="n">agent</span><span class="o">=</span><span class="n">agent</span><span class="p">,</span> <span class="p">...)</span> <span class="c1"># Fill in Runner details to run
</span>
</code></pre>

</div>



<p><strong>Delegating via A2A (Agent-to-Agent)</strong></p>

<p>The <strong>Agent2Agent (A2A)</strong> protocol is used to enable collaboration between different autonomous agents, potentially running on different servers or frameworks.</p>

<p><strong>A. Exposing an Agent (<code>to_a2a</code>)</strong><br>
This converts a local ADK agent into an A2A-compliant server that publishes an Agent Card.</p>

<p>To make an agent discoverable, we wrap it using the <code>to_a2a()</code> utility. This generates an <strong>Agent Card</strong>—a standardized manifest hosted at <code>.well-known/agent-card.json</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google.adk.agents</span> <span class="kn">import</span> <span class="n">LlmAgent</span>
<span class="kn">from</span> <span class="n">google.adk.a2a.utils.agent_to_a2a</span> <span class="kn">import</span> <span class="n">to_a2a</span>
<span class="kn">from</span> <span class="n">google.adk.tools.tool_context</span> <span class="kn">import</span> <span class="n">ToolContext</span>
<span class="kn">from</span> <span class="n">google.genai</span> <span class="kn">import</span> <span class="n">types</span>
<span class="kn">import</span> <span class="n">random</span>

<span class="c1"># Define the tools
</span><span class="k">def</span> <span class="nf">roll_die</span><span class="p">(</span><span class="n">sides</span><span class="p">:</span> <span class="nb">int</span><span class="p">,</span> <span class="n">tool_context</span><span class="p">:</span> <span class="n">ToolContext</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">int</span><span class="p">:</span>
  <span class="sh">"""</span><span class="s">Roll a die and return the rolled result.

  Args:
    sides: The integer number of sides the die has.
    tool_context: the tool context
  Returns:
    An integer of the result of rolling the die.
  </span><span class="sh">"""</span>
  <span class="n">result</span> <span class="o">=</span> <span class="n">random</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="n">sides</span><span class="p">)</span>
  <span class="k">if</span> <span class="ow">not</span> <span class="sh">'</span><span class="s">rolls</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">tool_context</span><span class="p">.</span><span class="n">state</span><span class="p">:</span>
    <span class="n">tool_context</span><span class="p">.</span><span class="n">state</span><span class="p">[</span><span class="sh">'</span><span class="s">rolls</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="p">[]</span>

  <span class="n">tool_context</span><span class="p">.</span><span class="n">state</span><span class="p">[</span><span class="sh">'</span><span class="s">rolls</span><span class="sh">'</span><span class="p">]</span> <span class="o">=</span> <span class="n">tool_context</span><span class="p">.</span><span class="n">state</span><span class="p">[</span><span class="sh">'</span><span class="s">rolls</span><span class="sh">'</span><span class="p">]</span> <span class="o">+</span> <span class="p">[</span><span class="n">result</span><span class="p">]</span>
  <span class="k">return</span> <span class="n">result</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">check_prime</span><span class="p">(</span><span class="n">nums</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="nb">int</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
  <span class="sh">"""</span><span class="s">Check if a given list of numbers are prime.

  Args:
    nums: The list of numbers to check.

  Returns:
    A str indicating which number is prime.
  </span><span class="sh">"""</span>
  <span class="n">primes</span> <span class="o">=</span> <span class="nf">set</span><span class="p">()</span>
  <span class="k">for</span> <span class="n">number</span> <span class="ow">in</span> <span class="n">nums</span><span class="p">:</span>
    <span class="n">number</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">number</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">number</span> <span class="o">&lt;=</span> <span class="mi">1</span><span class="p">:</span>
      <span class="k">continue</span>
    <span class="n">is_prime</span> <span class="o">=</span> <span class="bp">True</span>
    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="nf">int</span><span class="p">(</span><span class="n">number</span><span class="o">**</span><span class="mf">0.5</span><span class="p">)</span> <span class="o">+</span> <span class="mi">1</span><span class="p">):</span>
      <span class="k">if</span> <span class="n">number</span> <span class="o">%</span> <span class="n">i</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>
        <span class="n">is_prime</span> <span class="o">=</span> <span class="bp">False</span>
        <span class="k">break</span>
    <span class="k">if</span> <span class="n">is_prime</span><span class="p">:</span>
      <span class="n">primes</span><span class="p">.</span><span class="nf">add</span><span class="p">(</span><span class="n">number</span><span class="p">)</span>
  <span class="nf">return </span><span class="p">(</span>
      <span class="sh">'</span><span class="s">No prime numbers found.</span><span class="sh">'</span>
      <span class="k">if</span> <span class="ow">not</span> <span class="n">primes</span>
      <span class="k">else</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nf">str</span><span class="p">(</span><span class="n">num</span><span class="p">)</span> <span class="k">for</span> <span class="n">num</span> <span class="ow">in</span> <span class="n">primes</span><span class="p">)</span><span class="si">}</span><span class="s"> are prime numbers.</span><span class="sh">"</span>
  <span class="p">)</span>

<span class="c1"># 1. Define your local agent with relevant tools and instructions
# This example uses the 'hello_world' agent's logic for rolling dice and checking primes.
</span><span class="n">root_agent</span> <span class="o">=</span> <span class="nc">LlmAgent</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">'</span><span class="s">gemini-2.0-flash</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">name</span><span class="o">=</span><span class="sh">'</span><span class="s">hello_world_agent</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">description</span><span class="o">=</span><span class="p">(</span>
        <span class="sh">'</span><span class="s">hello world agent that can roll a die of 8 sides and check prime</span><span class="sh">'</span>
        <span class="sh">'</span><span class="s"> numbers.</span><span class="sh">'</span>
    <span class="p">),</span>
    <span class="n">instruction</span><span class="o">=</span><span class="sh">"""</span><span class="s">
      You roll dice and answer questions about the outcome of the dice rolls.
      When you are asked to roll a die, you must call the roll_die tool with the number of sides.
      When checking prime numbers, call the check_prime tool with a list of integers.
    </span><span class="sh">"""</span><span class="p">,</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[</span>
        <span class="n">roll_die</span><span class="p">,</span>
        <span class="n">check_prime</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="n">generate_content_config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
        <span class="n">safety_settings</span><span class="o">=</span><span class="p">[</span>
            <span class="n">types</span><span class="p">.</span><span class="nc">SafetySetting</span><span class="p">(</span>
                <span class="n">category</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="n">HarmCategory</span><span class="p">.</span><span class="n">HARM_CATEGORY_DANGEROUS_CONTENT</span><span class="p">,</span>
                <span class="n">threshold</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="n">HarmBlockThreshold</span><span class="p">.</span><span class="n">OFF</span><span class="p">,</span>
            <span class="p">),</span>
        <span class="p">]</span>
    <span class="p">),</span>
<span class="p">)</span>

<span class="c1"># 2. Convert to A2A application
# This automatically generates the Agent Card and sets up the HTTP endpoints
</span><span class="n">a2a_app</span> <span class="o">=</span> <span class="nf">to_a2a</span><span class="p">(</span><span class="n">root_agent</span><span class="p">,</span> <span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">localhost</span><span class="sh">"</span><span class="p">,</span> <span class="n">port</span><span class="o">=</span><span class="mi">8001</span><span class="p">)</span>

<span class="c1"># To run this application, save it as a Python file (e.g., `my_a2a_agent.py`)
# and execute it using uvicorn:
# uvicorn my_a2a_agent:a2a_app --host localhost --port 8001
</span></code></pre>

</div>



<p><em>The Agent Card (Discovery):</em></p>

<p>The <strong>Agent Card</strong> is a standardized JSON file that acts as a "business card" for an agent, allowing other agents to discover its capabilities, security requirements, and endpoints.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hello_world_agent"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hello world agent that can roll a die of 8 sides and check prime numbers. You roll dice and answer questions about the outcome of the dice rolls. When you are asked to roll a die, you must call the roll_die tool with the number of sides. When checking prime numbers, call the check_prime tool with a list of integers."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"doc_url"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
  </span><span class="nl">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"http://localhost:8001/"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"0.0.1"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"capabilities"</span><span class="p">:</span><span class="w"> </span><span class="p">{},</span><span class="w">
  </span><span class="nl">"skills"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hello_world_agent"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"model"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hello world agent that can roll a die of 8 sides and check prime numbers. I roll dice and answer questions about the outcome of the dice rolls. When I am asked to roll a die, I must call the roll_die tool with the number of sides. When checking prime numbers, call the check_prime tool with a list of integers."</span><span class="p">,</span><span class="w">
      </span><span class="nl">"examples"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"input_modes"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"output_modes"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"tags"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"llm"</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hello_world_agent-roll_die"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"roll_die"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Roll a die and return the rolled result."</span><span class="p">,</span><span class="w">
      </span><span class="nl">"examples"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"input_modes"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"output_modes"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"tags"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"llm"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"tools"</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hello_world_agent-check_prime"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"check_prime"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Check if a given list of numbers are prime."</span><span class="p">,</span><span class="w">
      </span><span class="nl">"examples"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"input_modes"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"output_modes"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
      </span><span class="nl">"tags"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"llm"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"tools"</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"default_input_modes"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"text/plain"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"default_output_modes"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"text/plain"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"supports_authenticated_extended_card"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"provider"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
  </span><span class="nl">"security_schemes"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="w">
</span><span class="p">}</span><span class="w">

</span></code></pre>

</div>



<p><strong>B. Consuming a Remote Agent (<code>RemoteA2aAgent</code>)</strong></p>

<p>To consume this, the parent agent simply points to the URL. The ADK treats the remote agent exactly like a local tool.</p>

<p>This allows a local agent to delegate tasks to a remote agent by reading its Agent Card.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google.adk.agents</span> <span class="kn">import</span> <span class="n">LlmAgent</span>
<span class="kn">from</span> <span class="n">google.adk.agents.remote_a2a_agent</span> <span class="kn">import</span> <span class="n">AGENT_CARD_WELL_KNOWN_PATH</span>
<span class="kn">from</span> <span class="n">google.adk.agents.remote_a2a_agent</span> <span class="kn">import</span> <span class="n">RemoteA2aAgent</span>

<span class="c1"># 1. Define the remote agent interface
# Points to the .well-known/agent.json of the running A2A server
</span><span class="n">prime_agent</span> <span class="o">=</span> <span class="nc">RemoteA2aAgent</span><span class="p">(</span>
    <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">remote_prime_agent</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Agent that handles checking if numbers are prime.</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">agent_card</span><span class="o">=</span><span class="sa">f</span><span class="sh">"</span><span class="s">http://localhost:8001/</span><span class="si">{</span><span class="n">AGENT_CARD_WELL_KNOWN_PATH</span><span class="si">}</span><span class="sh">"</span>
<span class="p">)</span>

<span class="c1"># 2. Use the remote agent as a sub-agent
</span><span class="n">root_agent</span> <span class="o">=</span> <span class="nc">LlmAgent</span><span class="p">(</span>
    <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">coordinator</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gemini-2.0-flash</span><span class="sh">"</span><span class="p">,</span> <span class="c1"># Explicitly define the model
</span>    <span class="n">instruction</span><span class="o">=</span><span class="sh">"""</span><span class="s">
      You are a coordinator agent.
      Your primary task is to delegate any requests related to prime number checking to the </span><span class="sh">'</span><span class="s">remote_prime_agent</span><span class="sh">'</span><span class="s">.
      Do not attempt to check prime numbers yourself.
      Ensure to pass the numbers to be checked to the </span><span class="sh">'</span><span class="s">remote_prime_agent</span><span class="sh">'</span><span class="s"> correctly.
      Clarify the results from the </span><span class="sh">'</span><span class="s">remote_prime_agent</span><span class="sh">'</span><span class="s"> to the user.
      </span><span class="sh">"""</span><span class="p">,</span>
    <span class="n">sub_agents</span><span class="o">=</span><span class="p">[</span><span class="n">prime_agent</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># You can then use this root_agent with a Runner, for example:
# from google.adk.runners import Runner
# runner = Runner(agent=root_agent)
# async for event in runner.run_async(user_id="test_user", session_id="test_session", new_message="Is 13 a prime number?"):
#     print(event)
</span>
</code></pre>

</div>



<p>While both protocols connect AI systems, they operate at different levels of abstraction.</p>

<p><strong>When to use which?</strong> </p>

<ul>
<li>Use <strong>MCP</strong> when you need deterministic execution of specific functions (stateless).</li>
<li>Use <strong>A2A</strong> when you need to offload a fuzzy goal that requires reasoning and state management (stateful).</li>
</ul>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th><strong>MCP (Model Context Protocol)</strong></th>
<th><strong>A2A (Agent2Agent Protocol)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Primary Domain</strong></td>
<td>
<strong>Tools &amp; Resources</strong>.</td>
<td>
<strong>Autonomous Agents</strong>.</td>
</tr>
<tr>
<td><strong>Interaction</strong></td>
<td>
<strong>"Do this specific thing"</strong>. Stateless execution of functions (e.g., "query database," "fetch file").</td>
<td>
<strong>"Achieve this complex goal"</strong>. Stateful, multi-turn collaboration where the remote agent plans and reasons.</td>
</tr>
<tr>
<td><strong>Abstraction</strong></td>
<td>
<strong>Low-level plumbing.</strong> Connects LLMs to data sources and APIs (like a USB-C port for AI).</td>
<td>
<strong>High-level collaboration.</strong> Connects intelligent agents to other intelligent agents to delegate responsibility.</td>
</tr>
<tr>
<td><strong>Standard</strong></td>
<td>Standardizes <strong>tool definitions</strong>, <strong>prompts</strong>, and <strong>resource reading</strong>.</td>
<td>Standardizes <strong>agent discovery</strong> (Agent Card), <strong>task lifecycles</strong>, and <strong>asynchronous communication</strong>.</td>
</tr>
<tr>
<td><strong>Analogy</strong></td>
<td>Using a specific wrench or diagnostic scanner.</td>
<td>Asking a specialized mechanic to fix a car engine.</td>
</tr>
</tbody>
</table></div>

<p><strong>How they work together:</strong><br>
An application might use <strong>A2A</strong> to orchestrate high-level collaboration between a "Manager Agent" and a "Coder Agent." </p>

<p>The "Coder Agent," in turn, uses <strong>MCP</strong> internally to connect to GitHub tools and a local file system to execute the work.</p>
<h3>
  
  
  4. Production Considerations
</h3>

<p>Moving protocols from <code>stdio</code> (local process) to HTTP (production network) introduces critical security challenges.</p>

<ul>
<li>
<strong>The "Confused Deputy" Problem:</strong> Protocols decouple execution, but they also expose risks. A malicious user might trick a privileged agent (the deputy) into using an MCP file-system tool to read sensitive configs. Production architectures must enforce <strong>Least Privilege</strong> by placing MCP servers behind API Gateways that enforce policy checks <em>before</em> the tool is executed.</li>
<li>
<strong>Discovery vs. Latency:</strong> Dynamic discovery adds a round-trip latency cost at startup (handshaking). In production, we often cache tool definitions (static binding) for performance, while keeping the execution dynamic.</li>
<li>
<strong>Governance:</strong> To prevent "Tool Sprawl" where agents connect to unverified servers, enterprises need a <strong>Centralized Registry</strong>—an allowlist of approved MCP servers and Agent Cards that act as the single source of truth for capabilities.</li>
</ul>
<h3>
  
  
  5. Ethical Design Note
</h3>

<p>Protocol-first architectures are the technical foundation for <strong>Human Sovereignty</strong> and <strong>Data Portability</strong>.</p>

<p>Standardizing the interface (MCP) helps us prevent vendor lock-in, among many other advantages. A user can swap out a "Google Drive" data source for a "Local Hard Drive" source without breaking the agent, ensuring the user—not the platform—controls where the data lives and how it is accessed.</p>

<p>This abstraction acts as a bulwark against <strong>algorithmic lock-in</strong>, ensuring that an agent's reasoning capabilities are decoupled from proprietary tool implementations, preserving the user's freedom to migrate their digital ecosystem without losing their intelligent assistants.</p>

<p><strong>Concrete Failure Scenario:</strong></p>

<p>Imagine a small business builds a customer service agent tightly coupled to Salesforce's proprietary API. Over three years, the agent accumulates thousands of lines of custom integration code. When Salesforce raises prices 300%, the business wants to migrate to HubSpot—but their agent is fundamentally Salesforce-shaped. Every tool, every data query, every workflow assumption is hardcoded. Migration means rebuilding the agent from scratch, which the business can't afford. They're trapped. This is <strong>algorithmic lock-in</strong>—not just vendor lock-in of data, but vendor lock-in of intelligence. Without protocol-first design, the agent becomes a hostage to the platform, and the user loses sovereignty over their own automation.</p>
<h3>
  
  
  Key Takeaways
</h3>

<ul>
<li>
<strong>Production Principle:</strong> Agents should depend on interfaces, not implementations. Protocol-first design (MCP for tools, A2A for peers) inverts the dependency and prevents the N×M integration trap.</li>
<li>
<strong>Implementation:</strong> Use <code>McpToolset</code> to connect agents to data sources via the Model Context Protocol. Use <code>RemoteA2aAgent</code> and <code>to_a2a()</code> for agent-to-agent delegation. Cache tool definitions at startup for performance, but keep execution dynamic.</li>
<li>
<strong>Ethical Anchor:</strong> This pattern operationalizes <strong>Human Sovereignty</strong> and <strong>Data Portability</strong>—users control where their data lives and which tools their agents use, free from vendor lock-in or algorithmic hostage-taking.</li>
</ul>



<p>We now have agents that reason correctly, remember what matters, and connect to any tool or peer through standard protocols. But there's one final constraint that threatens to unravel everything: the assumption that every interaction completes in a single request-response cycle. Real business workflows don't work that way. Approvals take hours. External APIs time out. Humans need time to think. This is where our fourth pattern becomes essential: teaching agents to pause, persist, and resume across the boundaries of time itself.</p>
<h2>
  
  
  Pattern 4: Long-Running Operations &amp; Resumability
</h2>

<p>This is perhaps the most critical pattern for integrating agents into real-world business logic where human approval is non-negotiable.</p>
<h3>
  
  
  1. The Production Problem
</h3>

<p>Naive agents fall into the <strong>"Stateless Trap."</strong></p>

<p>Imagine a <strong>Procurement Agent</strong> tasked with ordering 1,000 servers. </p>

<p>The workflow is: </p>

<ol>
<li>Analyze quotes</li>
<li>Propose the best option</li>
<li><strong>Wait for CFO approval</strong></li>
<li>Place the order</li>
</ol>

<p>Here's a mermaid sequence diagram illustrating the procurement workflow:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd8ka0cekcbx0bmz2y88j.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd8ka0cekcbx0bmz2y88j.webp" alt="Pattern 4_1" width="800" height="762"></a></p>

<p>This diagram shows the sequential flow from analyzing quotes through to placing the order, with the critical approval step from the CFO in the middle.</p>

<p>If the CFO takes 2 hours to review the proposal, a standard HTTP request will time out in seconds. When the CFO finally clicks "Approve," the agent has lost its memory. It doesn't know which vendor it selected, the quote ID, or why it made that recommendation. It essentially has to start over.</p>
<h3>
  
  
  2. The Architectural Solution
</h3>

<p>The solution is a <strong>Pause, Persist, Resume</strong> architecture.</p>

<ul>
<li>
<strong>Event-Driven Interruption:</strong> The agent doesn't just "wait." It emits a specific system event (<code>adk_request_confirmation</code>) and <strong>halts execution immediately</strong>, releasing compute resources.</li>
<li>
<strong>State Persistence:</strong> The agent's full state (conversation history, tool parameters, reasoning scratchpad) is serialized and stored in a database, keyed by an <code>invocation_id</code>.</li>
<li>
<strong>The Anchor (<code>invocation_id</code>):</strong> This ID becomes the critical "bookmark." When the human acts, the system rehydrates the agent using this ID, allowing it to resume <em>exactly</em> where it left off—inside the tool call—rather than restarting the conversation.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnugzjbe27qnqyrpwyqcz.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnugzjbe27qnqyrpwyqcz.webp" alt="Pattern 4_2" width="800" height="647"></a></p>
<h3>
  
  
  3. Implementation Details: Code Insights
</h3>

<p>The ADK provides the <code>ToolContext</code> and <code>App</code> primitives to handle this complexity without writing custom state machines.</p>

<p><strong>The Three-State Tool Pattern</strong><br>
Inside your tool definition, you must handle three scenarios: </p>

<ol>
<li>Automatic approval (low stakes)</li>
<li>Initial request (pause)</li>
<li>Resumption (action)
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">place_order</span><span class="p">(</span><span class="n">num_units</span><span class="p">:</span> <span class="nb">int</span><span class="p">,</span> <span class="n">tool_context</span><span class="p">:</span> <span class="n">ToolContext</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
    <span class="c1"># Scenario 1: Small orders auto-approve
</span>    <span class="k">if</span> <span class="n">num_units</span> <span class="o">&lt;=</span> <span class="mi">5</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">approved</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">order_id</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">ORD-</span><span class="si">{</span><span class="n">num_units</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>

    <span class="c1"># Scenario 2: First call - request approval (PAUSE)
</span>    <span class="c1"># The tool checks if confirmation exists. If not, it requests it and halts.
</span>    <span class="k">if</span> <span class="ow">not</span> <span class="n">tool_context</span><span class="p">.</span><span class="n">tool_confirmation</span><span class="p">:</span>
        <span class="n">tool_context</span><span class="p">.</span><span class="nf">request_confirmation</span><span class="p">(</span>
            <span class="n">hint</span><span class="o">=</span><span class="sa">f</span><span class="sh">"</span><span class="s">Large order: </span><span class="si">{</span><span class="n">num_units</span><span class="si">}</span><span class="s"> units. Approve?</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">payload</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">num_units</span><span class="sh">"</span><span class="p">:</span> <span class="n">num_units</span><span class="p">}</span>
        <span class="p">)</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pending</span><span class="sh">"</span><span class="p">}</span>

    <span class="c1"># Scenario 3: Resume - check decision (ACTION)
</span>    <span class="c1"># The tool runs again, but this time confirmation exists.
</span>    <span class="k">if</span> <span class="n">tool_context</span><span class="p">.</span><span class="n">tool_confirmation</span><span class="p">.</span><span class="n">confirmed</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">approved</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">order_id</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">ORD-</span><span class="si">{</span><span class="n">num_units</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">rejected</span><span class="sh">"</span><span class="p">}</span>

</code></pre>

</div>


<ol>
<li>
<strong>Automatic Approval (Scenario 1):</strong> The initial <code>if num_units &lt;= 5:</code> block handles immediate, non-long-running scenarios, which is a common pattern for tools that can quickly resolve simple requests.</li>
<li>
<strong>Initial Request (Pause - Scenario 2):</strong> The <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/tools/function_tool.py#L4" rel="noopener noreferrer"><strong><code>if not tool_context.tool_confirmation:</code></strong></a> block  leverages <code>tool_context.request_confirmation()</code> to signal that the tool requires external input to proceed. The return of <code>{"status": "pending"}</code> indicates that the operation is not yet complete.</li>
<li>
<strong>Resumption (Action - Scenario 3):</strong> The final <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/tools/function_tool.py#L56" rel="noopener noreferrer"><strong><code>if tool_context.tool_confirmation.confirmed:</code></strong></a> block demonstrates how the tool re-executes, this time finding <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/tools/function_tool.py#L196" rel="noopener noreferrer"><strong><code>tool_context.tool_confirmation</code></strong></a> present, indicating that the external input has been provided. The tool then acts based on the <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/tools/tool_confirmation.py#L41" rel="noopener noreferrer"><strong><code>confirmed</code></strong></a> status. The <a href="https://codewiki.google/github.com/google/adk-python#contribution-guidelines-and-samples-human-in-the-loop" rel="noopener noreferrer">Human-in-the-Loop Workflow Samples</a> also highlights how the application constructs a <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/events/event.py#L108" rel="noopener noreferrer"><strong><code>types.FunctionResponse</code></strong></a> with the updated status and sends it back to the agent to resume its task.</li>
</ol>

<p><strong>The Application Wrapper</strong><br>
To enable persistence, we wrap the agent in an <code>App</code> with <code>ResumabilityConfig</code>. This tells the ADK to automatically handle state serialization.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google.adk.apps</span> <span class="kn">import</span> <span class="n">App</span><span class="p">,</span> <span class="n">ResumabilityConfig</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">App</span><span class="p">(</span>
    <span class="n">root_agent</span><span class="o">=</span><span class="n">procurement_agent</span><span class="p">,</span>
    <span class="n">resumability_config</span><span class="o">=</span><span class="nc">ResumabilityConfig</span><span class="p">(</span><span class="n">is_resumable</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="p">)</span>

</code></pre>

</div>



<p><strong>The Workflow Loop</strong><br>
The runner loop must detect the pause and, crucially, use the same <code>invocation_id</code> to resume.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># 1. Initial Execution
</span><span class="k">async</span> <span class="k">for</span> <span class="n">event</span> <span class="ow">in</span> <span class="n">runner</span><span class="p">.</span><span class="nf">run_async</span><span class="p">(...):</span>
    <span class="n">events</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">event</span><span class="p">)</span>

<span class="c1"># 2. Detect Pause &amp; Get ID
</span><span class="n">approval_info</span> <span class="o">=</span> <span class="nf">check_for_approval</span><span class="p">(</span><span class="n">events</span><span class="p">)</span>

<span class="k">if</span> <span class="n">approval_info</span><span class="p">:</span>
    <span class="c1"># ... Wait for user input (hours/days) ...
</span>    <span class="n">user_decision</span> <span class="o">=</span> <span class="nf">get_user_decision</span><span class="p">()</span> <span class="c1"># True/False
</span>
    <span class="c1"># 3. Resume with INTENT
</span>    <span class="c1"># We pass the original invocation_id to rehydrate state
</span>    <span class="k">async</span> <span class="k">for</span> <span class="n">event</span> <span class="ow">in</span> <span class="n">runner</span><span class="p">.</span><span class="nf">run_async</span><span class="p">(</span>
        <span class="n">invocation_id</span><span class="o">=</span><span class="n">approval_info</span><span class="p">[</span><span class="sh">"</span><span class="s">invocation_id</span><span class="sh">"</span><span class="p">],</span>
        <span class="n">new_message</span><span class="o">=</span><span class="nf">create_approval_response</span><span class="p">(</span><span class="n">user_decision</span><span class="p">)</span>
    <span class="p">):</span>
        <span class="c1"># Agent continues execution from inside place_order()
</span>        <span class="k">pass</span>

</code></pre>

</div>



<p>This workflow shows the mechanism for resuming an agent's execution:</p>

<ul>
<li>
<strong>Initial Execution:</strong> The first <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/AGENTS.md?plain=1#L45" rel="noopener noreferrer"><strong><code>runner.run_async()</code></strong></a> call initiates the agent's interaction, which eventually leads to the <code>place_order</code> tool returning a "pending" status.</li>
<li>
<strong>Detecting Pause &amp; Getting ID:</strong> Detect the "pending" state and extract the <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/agents/readonly_context.py#L44" rel="noopener noreferrer"><strong><code>invocation_id</code></strong></a>. Check the <a href="https://codewiki.google/github.com/google/adk-python#agent-components-and-types-invocation-context-and-state-management" rel="noopener noreferrer">Invocation Context and State Management</a> code wiki section to check  how <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/agents/invocation_context.py#L98" rel="noopener noreferrer"><strong><code>InvocationContext</code></strong></a> tracks an agent's state and supports resumable operations.</li>
<li>
<strong>Resuming with Intent:</strong> The crucial part is calling <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/AGENTS.md?plain=1#L45" rel="noopener noreferrer"><strong><code>runner.run_async()</code></strong></a> again with the <em>same</em> <a href="https://github.com/google/adk-python/blob/2b6471550591ee7fc5f70f79e66a6e4080df442b/src/google/adk/agents/readonly_context.py#L44" rel="noopener noreferrer"><strong><code>invocation_id</code></strong></a>. This tells the ADK to rehydrate the session state and resume the execution from where it left off, providing the new message (the approval decision) as input. This behavior is used in the <a href="https://codewiki.google/github.com/google/adk-python#contribution-guidelines-and-samples-human-in-the-loop" rel="noopener noreferrer">Human-in-the-Loop Workflow Samples</a>,  where the runner orchestrates agent execution and handles multi-agent coordination.</li>
</ul>

<h3>
  
  
  4. Production Considerations
</h3>

<ul>
<li>
<strong>Persistence Strategy:</strong> <code>InMemorySessionService</code> is insufficient for production resumability because a server restart kills pending approvals. You must use a persistent store like <strong>Redis</strong> or <strong>PostgreSQL</strong> to save the serialized agent state.</li>
<li>
<strong>UI Signaling:</strong> The <code>adk_request_confirmation</code> event should trigger a real-time notification (via WebSockets) to the user's frontend, rendering an "Approve/Reject" card.</li>
<li>
<strong>Time-To-Live (TTL):</strong> Pending approvals shouldn't live forever. Implement a TTL policy (e.g., 24 hours) after which the state is garbage collected and the order is auto-rejected to prevent stale context rehydration.</li>
</ul>

<h3>
  
  
  5. Ethical Design Note
</h3>

<p>This pattern is the technical implementation of <strong>Meaningful Human Control</strong>.</p>

<p>It ensures high-stakes actions (Agency) remain subservient to human authorization (Sovereignty), preventing "rogue actions" where an agent executes irreversible decisions (like spending budget) without explicit oversight.</p>

<p><strong>Concrete Failure Scenario:</strong></p>

<p>Imagine a financial trading agent receives a signal to liquidate a portfolio position. Without resumability, the agent operates in a stateless, atomic transaction: detect signal → execute trade. There's no pause for human review. If the signal is based on a data glitch (a "flash crash"), or if market conditions have changed in the seconds between signal and execution, the agent completes an irreversible $10M trade that wipes out a quarter's earnings. The human operator sees the confirmation <em>after</em> the damage is done. Worse, if the system crashes mid-execution, the agent loses context and might try to execute the same trade twice, compounding the disaster. Without <strong>Meaningful Human Control</strong> embedded in the architecture, the agent becomes a runaway train.</p>

<h3>
  
  
  Key Takeaways
</h3>

<ul>
<li>
<strong>Production Principle:</strong> High-stakes actions require human-in-the-loop workflows. Design agents that can pause, wait for approval, and resume execution without losing context—spanning hours or days, not just seconds.</li>
<li>
<strong>Implementation:</strong> Use <code>ToolContext.request_confirmation()</code> for tools that need approval. Configure <code>ResumabilityConfig</code> in your <code>App</code> to enable state persistence. Use the <code>invocation_id</code> to resume execution from the exact point of interruption. Store state in Redis or PostgreSQL, never in-memory.</li>
<li>
<strong>Ethical Anchor:</strong> This pattern operationalizes <strong>Meaningful Human Control</strong>—we architecturally prevent agents from executing irreversible, high-stakes actions without explicit human authorization, preserving human sovereignty over consequential decisions.</li>
</ul>




<h2>
  
  
  Conclusion
</h2>

<p>The Google &amp; Kaggle Intensive was a masterclass not just in coding, but in <strong>thinking</strong>.</p>

<p>Building agents is not just about chaining prompts; it is about designing resilient systems that can handle the messiness of the real world.</p>

<ul>
<li>
<strong>Evaluation</strong> ensures we trust the process, not just the result.</li>
<li>
<strong>Dual-Layer Memory</strong> solves the economic and context limits of LLMs.</li>
<li>
<strong>Protocol-First (MCP)</strong> prevents integration spaghetti and silos.</li>
<li>
<strong>Resumability</strong> allows agents to participate in human-speed workflows safely.</li>
</ul>

<h3>
  
  
  Where to Start: A Prioritization Guide
</h3>

<p>If you're moving your first agent from prototype to production, consider implementing these patterns in order:</p>

<ol>
<li>
<strong>Start with Pattern 1 (Evaluation).</strong> Without trajectory validation, you're flying blind. Capture a handful of golden trajectories from your <code>adk web</code> sessions, configure a <code>TrajectoryEvaluator</code>, and establish your evaluation baseline before writing another line of agent code.</li>
<li>
<strong>Add Pattern 4 (Resumability) early</strong> if your agent performs <em>any</em> action that requires human approval or waits on external systems (payment processing, legal review, third-party APIs). The cost of refactoring a stateless agent into a resumable one later is enormous. Build with <code>invocation_id</code> and <code>ToolContext.request_confirmation()</code> from day one.</li>
<li>
<strong>Implement Pattern 2 (Dual-Layer Memory)</strong> when your agent starts handling multi-turn conversations or personalization. If you see users repeating themselves across sessions ("I'm allergic to shellfish" → 3 months later → "I'm allergic to shellfish"), or if your context costs are climbing, it's time for the Workbench/Filing Cabinet split.</li>
<li>
<strong>Adopt Pattern 3 (Protocol-First Interoperability)</strong> when you need to integrate your <em>second</em> data source or agent. The first integration is always bespoke; the second is where you refactor to MCP/A2A or accept technical debt forever. Don't wait until you have ten brittle integrations to wish you'd used protocols.</li>
</ol>

<h3>
  
  
  The Architect's Responsibility
</h3>

<p>As we move forward, our job as architects is to ensure these systems are not just smart, but <strong>reliable, efficient, and ethical.</strong></p>

<p>We are not just building tools—we are defining the interface between human intention and machine action. Every architectural decision we make either preserves or erodes human sovereignty, privacy, and meaningful control.</p>

<p>When you choose to validate trajectories, you're not just improving test coverage—you're building <strong>fiduciary responsibility</strong> into the system.</p>

<p>When you separate session from memory, you're not just optimizing token costs—you're designing for <strong>privacy by default</strong>.</p>

<p>When you adopt MCP and A2A, you're not just reducing integration complexity—you're preserving <strong>user freedom</strong> from algorithmic lock-in.</p>

<p>When you implement resumability, you're not just handling timeouts—you're enforcing <strong>meaningful human control</strong> over consequential actions.</p>

<p>These patterns are not neutral technical choices. They are ethical choices encoded in architecture.</p>

<p><em>Let's build responsibly.</em></p>

