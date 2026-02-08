---
Title: Every AI agent framework focuses on making agents smarter. None of them ask what happens when agents screw up.
Description: 
Author: yaron torjeman
Date: 2026-02-08T21:59:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>New agent framework drops every week. CrewAI. LangChain. AutoGen. OpenAI Agents SDK.</p>

<p>They all compete on the same axis: <strong>how smart can we make the agent?</strong></p>

<p>Nobody's competing on: <strong>how do we stop it from deleting prod?</strong></p>

<p>I work in DevOps. I manage Kubernetes clusters, CI/CD pipelines, and cloud infrastructure for a living. I've watched teams build incredible AI agent demos that can manage infrastructure, write code, push deployments.</p>

<p>Then the security review happens.</p>

<p>"So this thing can just… run <code>kubectl delete namespace production</code>?"</p>

<p>"Well technically"</p>

<p>"No."</p>

<p>Demo stays in staging. Forever. I've seen this three times this year alone.</p>

<h2>
  
  
  The gap nobody's filling
</h2>

<p>Here's the thing. Airflow, Temporal, n8n — they're great at <em>running</em> stuff. But they don't care <em>what</em> they're running. Safety is your problem.</p>

<p>Agent frameworks? They care about reasoning, tool selection, memory. They don't care what happens when the reasoning is wrong.</p>

<p>There's a gap between "the agent decided to do X" and "X actually happened in production." Nobody owns that gap.</p>

<p>So I built something to own it.</p>

<h2>
  
  
  Cordum
</h2>

<p><a href="https://cordum.io" rel="noopener noreferrer">Cordum</a> is an open-source control plane that sits between AI agents and infrastructure. Every intent gets intercepted and evaluated <strong>before</strong> execution.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Agent: "I want to run kubectl delete namespace production"
    ↓
Cordum Safety Kernel: evaluates against policy
    ↓
Decision: DENY — "Destructive operations on production are not allowed"
    ↓
Result: Command never reaches your cluster.
</code></pre>

</div>



<p>That's the entire idea. Policy-as-code, evaluated pre-dispatch, with a full audit trail.</p>

<h2>
  
  
  What it looks like in practice
</h2>

<p>You write policies in YAML:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">rules</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">block-destructive-prod</span>
    <span class="na">match</span><span class="pi">:</span>
      <span class="na">risk_tags</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">destructive</span><span class="pi">,</span> <span class="nv">production</span><span class="pi">]</span>
    <span class="na">decision</span><span class="pi">:</span> <span class="s">deny</span>
    <span class="na">reason</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Destructive</span><span class="nv"> </span><span class="s">prod</span><span class="nv"> </span><span class="s">operations</span><span class="nv"> </span><span class="s">blocked"</span>

  <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">approve-prod-writes</span>
    <span class="na">match</span><span class="pi">:</span>
      <span class="na">risk_tags</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">production</span><span class="pi">,</span> <span class="nv">write</span><span class="pi">]</span>
    <span class="na">decision</span><span class="pi">:</span> <span class="s">require_approval</span>
    <span class="na">reason</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Production</span><span class="nv"> </span><span class="s">writes</span><span class="nv"> </span><span class="s">need</span><span class="nv"> </span><span class="s">human</span><span class="nv"> </span><span class="s">sign-off"</span>
</code></pre>

</div>



<p>Four possible outcomes:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Decision</th>
<th>What happens</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>allow</code></td>
<td>Action proceeds, fully logged</td>
</tr>
<tr>
<td><code>deny</code></td>
<td>Blocked. Never reaches infrastructure</td>
</tr>
<tr>
<td><code>require_approval</code></td>
<td>Human gets pinged. Action waits</td>
</tr>
<tr>
<td><code>constrain</code></td>
<td>Allowed, but with enforced limits</td>
</tr>
</tbody>
</table></div>

<p>The kernel evaluates every action in under 5ms. Not a sidecar. Not a webhook. It's in the execution path.</p>

<h2>
  
  
  Architecture
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────┐
│  Agent (Claude, GPT, CrewAI, whatever)  │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────┴───────────────────────┐
│  Cordum Control Plane                   │
│  ┌──────────────────────────────────┐   │
│  │  Safety Kernel (policy gate)     │   │
│  │  → allow / deny / approve / cap  │   │
│  └──────────────────────────────────┘   │
│  ┌──────────────────────────────────┐   │
│  │  Scheduler + Workflow Engine     │   │
│  └──────────────────────────────────┘   │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────┴───────────────────────┐
│  Infrastructure (K8s, AWS, GitHub, etc) │
└─────────────────────────────────────────┘
</code></pre>

</div>



<p>Built in Go. NATS JetStream for durable messaging. Redis for state. Not a Python wrapper around an LLM API call.</p>

<h2>
  
  
  What Cordum is NOT
</h2>

<p>This matters:</p>

<p><strong>It's not an agent framework.</strong> It doesn't replace CrewAI or LangChain. It governs them. Use whatever agent framework you want — Cordum sits underneath.</p>

<p><strong>It's not a workflow engine.</strong> Airflow runs DAGs. Cordum decides whether a step in your DAG is <em>allowed to run</em>.</p>

<p><strong>It's not post-hoc logging.</strong> By the time you're reading logs, the damage is done. Cordum blocks bad actions before they execute.</p>

<h2>
  
  
  The pack system
</h2>

<p>Want to add capabilities? Install a pack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cordum pack <span class="nb">install </span>slack
cordum pack <span class="nb">install </span>kubernetes
cordum pack <span class="nb">install </span>github
</code></pre>

</div>



<p>16 packs available. Each ships as a signed OCI container. Installs with policy overlays — so new capabilities come with governance built in.</p>

<h2>
  
  
  Rough edges
</h2>

<p>It's v0.1.0. I'm not going to pretend otherwise.</p>

<p>The docs need work. Some error messages are cryptic. The dashboard is functional, not pretty. I'm one developer building this between my day job and too much coffee.</p>

<p>But the core — the Safety Kernel, the CAP protocol, the policy engine — that works. It's in production. The architecture decisions are ones I'd make again.</p>

<h2>
  
  
  Why I think this matters
</h2>

<p>We're in a weird moment. Everyone's racing to give agents more autonomy. More tool access. More decision-making power. And the governance story is basically: "we'll figure it out later."</p>

<p>Later is a <code>kubectl delete</code> away from being too late.</p>




<p>⭐ <strong>GitHub:</strong> <a href="https://github.com/cordum-io/cordum" rel="noopener noreferrer">github.com/cordum-io/cordum</a><br>
🌐 <strong>Website:</strong> <a href="https://cordum.io" rel="noopener noreferrer">cordum.io</a><br>
📖 <strong>Docs:</strong> <a href="https://cordum.io/docs" rel="noopener noreferrer">cordum.io/docs</a></p>

<p>If you've ever had an agent demo killed by a security review — I built this for you.</p>

<p>What's your team's approach to agent governance? Genuinely curious. I keep hearing "we just don't deploy them to prod" and that can't be the final answer.</p>

