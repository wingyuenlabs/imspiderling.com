---
Title: The Bounded Task Principle: Why Constrained AI Agents Outperform Open-Ended Ones
Description: 
Author: Patrick
Date: 2026-03-08T21:55:56.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  The Bounded Task Principle: Why Constrained AI Agents Outperform Open-Ended Ones
</h1>

<p>Claude just found 22 vulnerabilities in Firefox in two weeks — including 14 high-severity ones.</p>

<p>People are talking about model capability. They're missing the real lesson.</p>

<p><strong>It worked because the task was bounded.</strong></p>

<p>Defined scope. Defined output format. Defined success criteria. The agent wasn't asked to "improve Firefox security" — it was given specific parameters, a specific surface area, and a clear definition of what a finding looks like.</p>

<p>That's the principle. And it applies to every agent you build.</p>




<h2>
  
  
  Why Vague Tasks Break Agents
</h2>

<p>Most agents fail not because of model quality — but because the task spec is under-defined.</p>

<p>When a task can mean five different things, an agent picks one interpretation. Often the wrong one. Then it optimizes hard for that interpretation while the user wanted something else entirely.</p>

<p>Symptoms of a vague task spec:</p>

<ul>
<li>Agent loops longer than expected</li>
<li>Output looks "reasonable" but misses the point</li>
<li>Different runs produce wildly different results</li>
<li>Agent asks clarifying questions at the wrong time (mid-task, not before)</li>
</ul>




<h2>
  
  
  The Four Fields of a Bounded Task
</h2>

<p>Add these four fields to every task definition — whether it's in a prompt, a current-task.json, or a SOUL.md instruction:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"scope"</span><span class="p">:</span><span class="w"> </span><span class="s2">"What is in bounds. Be explicit about what is NOT in scope."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"output_format"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Exactly what the result should look like."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"done_when"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Observable condition that defines completion."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"success_criteria"</span><span class="p">:</span><span class="w"> </span><span class="s2">"How a human will evaluate whether the output is correct."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Scope</strong> prevents the agent from doing too much. It answers: "What are the edges?"</p>

<p><strong>Output format</strong> prevents interpretation drift. It answers: "What does done look like structurally?"</p>

<p><strong>Done when</strong> prevents infinite loops. It answers: "When should the agent stop?"</p>

<p><strong>Success criteria</strong> closes the feedback loop. It answers: "How will we know it worked?"</p>




<h2>
  
  
  The Firefox Example Applied
</h2>

<p>Here's how the Anthropic/Firefox audit maps to these four fields:</p>

<ul>
<li>
<strong>Scope:</strong> Firefox source code — specific modules, not the whole web</li>
<li>
<strong>Output format:</strong> Structured vulnerability reports with severity, description, reproduction steps</li>
<li>
<strong>Done when:</strong> All in-scope modules reviewed, or X hours elapsed</li>
<li>
<strong>Success criteria:</strong> Human security engineer can reproduce and confirm each finding</li>
</ul>

<p>Notice what isn't in there: "make Firefox more secure." That's an aspiration, not a task spec. The bounded version is what produced 22 actionable findings.</p>




<h2>
  
  
  Constraint Is the Capability
</h2>

<p>This runs counter to how most people think about agents. They assume more capability = better results. So they give the agent more tools, wider scope, longer context windows.</p>

<p>The opposite is usually true.</p>

<p>A well-constrained agent with a bounded task:</p>

<ul>
<li>Spends less time reasoning about what to do</li>
<li>Produces more consistent outputs</li>
<li>Fails faster when something is wrong (instead of drifting)</li>
<li>Is dramatically cheaper to run</li>
</ul>

<p>The agents that perform best in production aren't the ones with the most tools. They're the ones with the clearest task definitions.</p>




<h2>
  
  
  The 3-Minute Audit
</h2>

<p>For any agent you're running today:</p>

<ol>
<li>Can you state the scope in one sentence? (If not, the scope is too vague)</li>
<li>Can you describe what the output looks like before the agent runs? (If not, the format isn't defined)</li>
<li>Can you tell — without ambiguity — when the agent should stop? (If not, you don't have a done condition)</li>
<li>Would you know if the output was wrong? (If not, you don't have success criteria)</li>
</ol>

<p>If you can't answer all four, tighten the task spec before adding any new tools or capabilities.</p>




<p>The battle-tested configs for bounded task patterns — including current-task.json templates and SOUL.md escalation rules — are in the Ask Patrick Library.</p>

<p>→ <strong><a href="https://askpatrick.co/library" rel="noopener noreferrer">askpatrick.co/library</a></strong></p>

