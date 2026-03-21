---
Title: OpenTelemetry just standardized LLM tracing. Here's what it actually looks like in code.
Description: 
Author: Albert Alov
Date: 2026-03-21T21:47:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every LLM tool invents its own tracing format. Langfuse has one. Helicone has one. Arize has one. If you built your own — congratulations, you have one too.</p>

<p>OpenTelemetry just published a standard for all of them.</p>

<p>It defines how to name spans, what attributes a tool call should have, how to log prompts without leaking PII, and which span kind to use for an agent. It's called GenAI Semantic Conventions. It's experimental. And almost nobody has written about what it actually looks like when you implement it.</p>

<p>I know because I searched. "OTel GenAI semantic conventions" gives you spec pages. Zero practical articles. "How to trace LLM agent with OpenTelemetry" gives you StackOverflow questions with no answers.</p>

<p>We implemented it. Four PRs, a gap analysis, real before/after code. We also discovered, mid-implementation, that our traces never exported at all — but that's a <a href="https://dev.to/vola-trebla/i-audited-my-tool-fixed-44-bugs-and-it-still-didnt-work-4omk">different story</a>.</p>

<p>Here's what the spec actually says, where we got it wrong, and what you should do today.</p>




<h2>
  
  
  The wild west of LLM tracing
</h2>

<p>Right now, if you trace LLM calls, you're probably doing something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">llm.provider</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">openai</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">llm.model</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">gpt-4o</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">llm.tokens.input</span><span class="dl">"</span><span class="p">,</span> <span class="mi">150</span><span class="p">);</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">llm.cost</span><span class="dl">"</span><span class="p">,</span> <span class="mf">0.003</span><span class="p">);</span>
</code></pre>

</div>



<p>That's what we did in toad-eye v1. Made sense to us. Worked fine in our dashboards.</p>

<p>Problem: nobody else's dashboards understand these attributes. Switch from Jaeger to Arize Phoenix — reconfigure everything. Export traces to Datadog — they see raw spans with no LLM context. Your tracing is a walled garden. You built vendor lock-in into your own code.</p>

<p>This is exactly what OpenTelemetry was created to solve. And now it has a spec for GenAI.</p>

<h2>
  
  
  Three types of GenAI spans
</h2>

<p>The spec defines three operations. Every LLM-related span gets one:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>chat gpt-4o                    ← model call
invoke_agent orchestrator      ← agent invocation  
execute_tool web_search        ← tool execution
</code></pre>

</div>



<p>The span name format is <code>{operation} {name}</code>. Not your custom format. Not <code>gen_ai.openai.gpt-4o</code> (that's what we had — no backend recognizes it).</p>

<p>Here's what we changed:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ezxomm21gsx5yjv1m5f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ezxomm21gsx5yjv1m5f.png" alt="Screenshot: Span"></a><br>
<em>Span naming migration: the old format was invisible to every GenAI-aware backend.</em></p>
<h2>
  
  
  Agent attributes — we had the paths wrong
</h2>

<p>If you're building agents (ReAct, tool-use, multi-step), the spec defines identity and tool attributes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// What OTel says:</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen_ai.agent.name</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">weather-bot</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen_ai.agent.id</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">agent-001</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen_ai.tool.name</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">search</span><span class="dl">"</span><span class="p">);</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen_ai.tool.type</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">function</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// What we had:</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen_ai.agent.tool.name</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">search</span><span class="dl">"</span><span class="p">);</span>  <span class="c1">// wrong path</span>
<span class="c1">// gen_ai.agent.name — didn't exist at all</span>
</code></pre>

</div>



<p>The <code>gen_ai.agent.tool.name</code> path looks reasonable. It even reads well. But the spec puts tool attributes at <code>gen_ai.tool.*</code> — flat, not nested under agent. Our format, again, invisible to any backend that follows the standard.</p>

<h2>
  
  
  Content recording — the spec agrees with us (feels good)
</h2>

<p>This was the one thing we got right from day one, and it's worth calling out because most teams get it wrong.</p>

<p>The spec says: <strong>don't record prompts and completions by default.</strong> Instrumentations SHOULD NOT capture content unless explicitly enabled.</p>

<p>Three official patterns:</p>

<ol>
<li>
<strong>Default: don't record.</strong> No prompt, no completion in spans. Privacy first.</li>
<li>
<strong>Opt-in via span attributes.</strong> <code>gen_ai.input.messages</code> and <code>gen_ai.output.messages</code> as JSON strings.</li>
<li>
<strong>External storage.</strong> Store content elsewhere, put a reference on the span.</li>
</ol>

<p>We had <code>recordContent: false</code> as default since v1. When the spec confirmed this approach, it was one of those rare moments where your gut feeling gets validated by a committee of very smart people.</p>

<p>If you're logging prompts in spans by default — you might want to reconsider before your security team does it for you.</p>

<h2>
  
  
  The honest gap analysis
</h2>

<p>Here's the full picture. No spin, no cherry-picking.</p>

<h3>
  
  
  What we got right from day 1
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Our attribute</th>
<th>OTel spec</th>
<th>Verdict</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>gen_ai.provider.name</code></td>
<td><code>gen_ai.provider.name</code></td>
<td>✅ Exact match</td>
</tr>
<tr>
<td><code>gen_ai.request.model</code></td>
<td><code>gen_ai.request.model</code></td>
<td>✅ Exact match</td>
</tr>
<tr>
<td><code>gen_ai.usage.input_tokens</code></td>
<td><code>gen_ai.usage.input_tokens</code></td>
<td>✅ Exact match</td>
</tr>
<tr>
<td><code>error.type</code></td>
<td><code>error.type</code></td>
<td>✅ Exact match</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  What we got wrong
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>What</th>
<th>Our version</th>
<th>OTel spec</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td>Span name</td>
<td><code>gen_ai.openai.gpt-4o</code></td>
<td><code>chat gpt-4o</code></td>
<td>Fixed</td>
</tr>
<tr>
<td>Tool name attribute</td>
<td><code>gen_ai.agent.tool.name</code></td>
<td><code>gen_ai.tool.name</code></td>
<td>Fixed</td>
</tr>
<tr>
<td>Custom attributes</td>
<td><code>gen_ai.agent.step.*</code></td>
<td>Reserved namespace</td>
<td>Moved to <code>gen_ai.toad_eye.*</code>
</td>
</tr>
<tr>
<td>Agent identity</td>
<td>Didn't exist</td>
<td><code>gen_ai.agent.name</code></td>
<td>Added</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  What we built beyond the spec
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Namespace</th>
<th>Why it's not in OTel</th>
</tr>
</thead>
<tbody>
<tr>
<td>Cost per request</td>
<td><code>gen_ai.toad_eye.cost</code></td>
<td>Pricing is vendor-specific</td>
</tr>
<tr>
<td>Budget guards</td>
<td><code>gen_ai.toad_eye.budget.*</code></td>
<td>Runtime enforcement ≠ observability</td>
</tr>
<tr>
<td>Shadow guardrails</td>
<td><code>gen_ai.toad_eye.guard.*</code></td>
<td>Validation is app-level</td>
</tr>
<tr>
<td>Semantic drift</td>
<td><code>gen_ai.toad_eye.semantic_drift</code></td>
<td>Quality metric, not trace standard</td>
</tr>
<tr>
<td>ReAct step tracking</td>
<td><code>gen_ai.toad_eye.agent.step.*</code></td>
<td>ReAct is one pattern; spec is pattern-agnostic</td>
</tr>
</tbody>
</table></div>

<p>The key insight: <strong>OTel spec covers WHAT happened. We cover WHY and HOW MUCH.</strong> Not competing — complementary. Your custom metrics go under your namespace. The spec's attributes go where backends expect them.</p>

<h2>
  
  
  The migration: dual-emit, don't break users
</h2>

<p>We didn't do a clean break. v2.4 emits both old and new attribute names:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// New (OTel spec-compliant)</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen_ai.tool.name</span><span class="dl">"</span><span class="p">,</span> <span class="nx">toolName</span><span class="p">);</span>

<span class="c1">// Old (deprecated, still emitted for backward compat)</span>
<span class="nx">span</span><span class="p">.</span><span class="nf">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen_ai.agent.tool.name</span><span class="dl">"</span><span class="p">,</span> <span class="nx">toolName</span><span class="p">);</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl4qj269knbpmegcwwtlt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl4qj269knbpmegcwwtlt.png" alt="Screenshot: Attribute prefix migration diff with @deprecated tags and dual-emit"></a><br>
<em>Dual-emit approach: old attributes get <code>@deprecated</code>, new ones follow the spec. Both emitted until v3.</em></p>

<p>An environment variable controls when to stop emitting deprecated attributes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">OTEL_SEMCONV_STABILITY_OPT_IN</span><span class="o">=</span>gen_ai_latest_experimental
</code></pre>

</div>



<p>This was four PRs (<a href="https://github.com/vola-trebla/toad-eye/pull/170" rel="noopener noreferrer">#170</a>, <a href="https://github.com/vola-trebla/toad-eye/pull/171" rel="noopener noreferrer">#171</a>, <a href="https://github.com/vola-trebla/toad-eye/pull/172" rel="noopener noreferrer">#172</a>, <a href="https://github.com/vola-trebla/toad-eye/pull/173" rel="noopener noreferrer">#173</a>). v3 will remove the deprecated aliases entirely.</p>

<h2>
  
  
  The irony
</h2>

<p>While implementing all of this, we did a round of manual testing.</p>

<p>Turns out our traces never exported. At all. Ever. The OTel <code>NodeSDK</code> silently disables trace export when you pass <code>spanProcessors: []</code>. We had 252 passing tests. All of them mocked the SDK.</p>

<p>So we standardized our attributes perfectly — for traces that nobody could see.</p>

<p>We fixed both. Published six patch versions in one day. The <a href="https://dev.to/vola-trebla/i-audited-my-tool-fixed-44-bugs-and-it-still-didnt-work-4omk">full story is in article #2</a>.</p>

<h2>
  
  
  Which backends actually support this
</h2>

<p>This is the reason to care. Emit the right attributes today → six backends visualize your traces tomorrow:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Backend</th>
<th>Recognizes GenAI spans</th>
<th>Agent visualization</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Jaeger</td>
<td>Basic (nested spans)</td>
<td>Hierarchy view</td>
<td>Free</td>
</tr>
<tr>
<td>Arize Phoenix</td>
<td>Full GenAI UI</td>
<td>Agent workflow</td>
<td>Free tier</td>
</tr>
<tr>
<td>SigNoz</td>
<td>GenAI dashboards</td>
<td>Nested spans</td>
<td>Free / Cloud</td>
</tr>
<tr>
<td>Datadog</td>
<td>LLM Observability</td>
<td>Agent tracing</td>
<td>Paid</td>
</tr>
<tr>
<td>Langfuse</td>
<td>Full GenAI UI</td>
<td>Session view</td>
<td>Free tier</td>
</tr>
<tr>
<td>Grafana + Tempo</td>
<td>Query by attributes</td>
<td>Custom dashboards</td>
<td>Free</td>
</tr>
</tbody>
</table></div>

<p>No vendor lock-in. One set of attributes. Six places to visualize them.</p>

<h2>
  
  
  What you should do today
</h2>

<p>If you're tracing LLM calls — even with custom code — aligning with the spec now saves you pain later. The conventions are experimental, but the direction is locked in.</p>

<p>Quick checklist:</p>

<ul>
<li>Set <code>gen_ai.operation.name</code> on every LLM span: <code>chat</code>, <code>invoke_agent</code>, or <code>execute_tool</code>
</li>
<li>Format span names as <code>{operation} {model_or_agent_name}</code>
</li>
<li>Use official attributes: <code>gen_ai.agent.name</code>, <code>gen_ai.tool.name</code>, <code>gen_ai.tool.type</code>
</li>
<li>Put YOUR custom attributes under YOUR namespace — not <code>gen_ai.*</code>
</li>
<li>Don't record prompt/completion by default — make it opt-in</li>
<li>Test your traces in at least 2 backends (Jaeger + one GenAI-specific like Phoenix)</li>
</ul>

<p>Full spec: <a href="https://opentelemetry.io/docs/specs/semconv/gen-ai/" rel="noopener noreferrer">OpenTelemetry GenAI Semantic Conventions</a><br>
Agent spans: <a href="https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/" rel="noopener noreferrer">GenAI Agent Spans</a></p>




<p><strong>Previous articles:</strong></p>

<ul>
<li><a href="https://dev.to/vola-trebla/my-ai-bot-burned-through-my-api-budget-overnight-so-i-built-an-open-source-tool-to-make-sure-it-2372">#1: My AI bot burned through my API budget overnight</a></li>
<li><a href="https://dev.to/vola-trebla/i-audited-my-tool-fixed-44-bugs-and-it-still-didnt-work-4omk">#2: I audited my tool, fixed 44 bugs — and it still didn't work</a></li>
</ul>

<p><strong>toad-eye</strong> — open-source LLM observability, OTel-native: <a href="https://github.com/vola-trebla/toad-eye" rel="noopener noreferrer">GitHub</a> · <a href="https://www.npmjs.com/package/toad-eye" rel="noopener noreferrer">npm</a></p>

<p>🐸👁️</p>

