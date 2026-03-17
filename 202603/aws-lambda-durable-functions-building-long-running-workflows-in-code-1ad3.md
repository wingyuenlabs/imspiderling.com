---
Title: AWS Lambda Durable Functions: Building Long-Running Workflows in Code
Description: 
Author: Gunnar Grosch
Date: 2026-03-17T22:06:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you've built anything non-trivial on AWS Lambda, you've hit the wall. The function runs for 15 minutes and it's stateless. Any multi-step workflow requires stitching together Step Functions, SQS queues, DynamoDB tables for state, and a whole lot of glue. It works, but it's a lot of infrastructure for what should be straightforward sequential logic.</p>

<p>AWS Lambda Durable Functions, <a href="https://aws.amazon.com/blogs/aws/build-multi-step-applications-and-ai-workflows-with-aws-lambda-durable-functions/" rel="noopener noreferrer">launched at re:Invent 2025</a>, change that. You write sequential code in a single Lambda function. The SDK handles checkpointing, failure recovery, and suspension. Your function can run for up to a year, and you only pay for active compute time. During waits (human approvals, timers, external callbacks), the function suspends and compute charges stop.</p>

<p>In this post, I'll walk through what problem durable functions solve, how the checkpoint/replay model works, and then dig into a complete AI-powered support ticket workflow in TypeScript that demonstrates every primitive in action.</p>

<h2>
  
  
  What Problem This Solves
</h2>

<p>Here's a scenario most teams deal with: a support ticket arrives, someone needs to triage it, figure out who should handle it, wait for them to respond, and then close the loop with the customer. Before durable functions, you had a few options:</p>

<p><strong>Step Functions:</strong> Define an ASL state machine with states for each step, configure IAM for each integration, manage the state machine as a separate resource. Great for cross-service orchestration, but heavyweight for application logic that naturally reads as sequential code.</p>

<p><strong>SQS + multiple Lambda functions:</strong> Break the workflow into separate functions connected by queues. Now you're managing message formats, dead-letter queues, idempotency, and correlating state across function boundaries.</p>

<p><strong>Polling loop with DynamoDB:</strong> One function writes state to DynamoDB, another polls for changes. Works, but you're paying for polling compute and managing your own state machine.</p>

<p>All three approaches take what should be straightforward sequential logic and spread it across multiple services, IAM policies, and configuration files.</p>

<p>With durable functions, that same workflow looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nx">TicketEvent</span><span class="p">,</span> <span class="nx">context</span><span class="p">:</span> <span class="nx">DurableContext</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">analysis</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">analyze</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">analyzeTicket</span><span class="p">(</span><span class="nx">event</span><span class="p">));</span>
  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">waitForCallback</span><span class="p">(</span><span class="dl">"</span><span class="s2">agent-review</span><span class="dl">"</span><span class="p">,</span>
    <span class="k">async </span><span class="p">(</span><span class="nx">callbackId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">notifyAgent</span><span class="p">(</span><span class="nx">callbackId</span><span class="p">,</span> <span class="nx">analysis</span><span class="p">)</span>
  <span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">analysis</span><span class="p">.</span><span class="nx">needsEscalation</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">waitForCallback</span><span class="p">(</span><span class="dl">"</span><span class="s2">specialist-review</span><span class="dl">"</span><span class="p">,</span>
      <span class="k">async </span><span class="p">(</span><span class="nx">callbackId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">notifySpecialist</span><span class="p">(</span><span class="nx">callbackId</span><span class="p">,</span> <span class="nx">analysis</span><span class="p">)</span>
    <span class="p">);</span>
  <span class="p">}</span>
  <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">parallel</span><span class="p">(</span><span class="dl">"</span><span class="s2">close-ticket</span><span class="dl">"</span><span class="p">,</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">reply</span><span class="dl">"</span><span class="p">,</span> <span class="na">func</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">ctx</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">ctx</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">send-reply</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">sendReply</span><span class="p">(</span><span class="nx">response</span><span class="p">))</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">survey</span><span class="dl">"</span><span class="p">,</span> <span class="na">func</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">ctx</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">ctx</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">send-survey</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">sendSurvey</span><span class="p">(</span><span class="nx">event</span><span class="p">))</span> <span class="p">},</span>
  <span class="p">]);</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="dl">"</span><span class="s2">resolved</span><span class="dl">"</span><span class="p">,</span> <span class="na">ticketId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span> <span class="p">};</span>
<span class="p">};</span>
</code></pre>

</div>



<p>One function. Sequential code. The SDK handles checkpointing each step, suspending during the human review waits, and resuming when the callbacks arrive.</p>

<h2>
  
  
  How Checkpoint/Replay Works
</h2>

<p>This is the part that makes everything else make sense. Durable functions use a checkpoint and replay model. Here's how it works:</p>

<ol>
<li>
<strong>First invocation:</strong> Your handler runs from the beginning. Each <code>context.step()</code> executes your code and checkpoints the result.</li>
<li>
<strong>Suspension:</strong> When <code>context.wait()</code> (fixed-duration pause) or <code>context.waitForCallback()</code> (external signal) is called, the function terminates. Compute charges stop.</li>
<li>
<strong>Resumption:</strong> When the wait completes or a callback arrives, Lambda invokes your handler again from the beginning. But this time, completed steps return their cached results instantly without re-executing. Execution picks up from the first non-checkpointed operation.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>First invocation:
  analyze     -&gt;  [executes Bedrock call, checkpoints result]
  agent-review -&gt;  [creates callback, function suspends]

Second invocation (agent responds):
  analyze     -&gt;  [returns cached result, skips Bedrock call]
  agent-review -&gt;  [returns callback result]
  close-ticket -&gt;  [sends reply + survey in parallel]
</code></pre>

</div>



<p>There's one critical rule that falls out of this: <strong>code outside steps re-executes on every replay and must be deterministic.</strong> If you use <code>Date.now()</code>, <code>Math.random()</code>, or <code>crypto.randomUUID()</code> outside a step, you'll get different values on each replay. Wrap non-deterministic operations in steps.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Wrong: different value on each replay</span>
<span class="kd">const</span> <span class="nx">id</span> <span class="o">=</span> <span class="nx">crypto</span><span class="p">.</span><span class="nf">randomUUID</span><span class="p">();</span>

<span class="c1">// Right: checkpointed, same value on every replay</span>
<span class="kd">const</span> <span class="nx">id</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen-id</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">crypto</span><span class="p">.</span><span class="nf">randomUUID</span><span class="p">());</span>
</code></pre>

</div>



<h2>
  
  
  What You'll Build
</h2>

<p>A support ticket triage workflow where AI handles the first pass and humans make the final call. This is the pattern that makes durable functions click: the AI analysis takes seconds, but the human reviews take hours or days. Without durable functions, you'd need to persist state somewhere and wire up resumption logic. With them, you just write <code>await context.waitForCallback()</code> and the function suspends until the human responds.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Primitive</th>
<th>What It Does</th>
<th>Where You'll See It</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>step()</code></td>
<td>Execute and checkpoint an atomic operation</td>
<td>AI ticket analysis with Bedrock</td>
</tr>
<tr>
<td><code>waitForCallback()</code></td>
<td>Suspend until an external system responds</td>
<td>Agent review, specialist escalation</td>
</tr>
<tr>
<td><code>parallel()</code></td>
<td>Run multiple branches concurrently</td>
<td>Customer reply + satisfaction survey</td>
</tr>
<tr>
<td>Retry strategies</td>
<td>Automatic retry with exponential backoff</td>
<td>Bedrock API calls</td>
</tr>
<tr>
<td><code>context.logger</code></td>
<td>Replay-aware structured logging (suppresses duplicate output during replay)</td>
<td>Throughout</td>
</tr>
</tbody>
</table></div>

<p>The complete source code is on GitHub: <a href="https://github.com/gunnargrosch/durable-support-triage" rel="noopener noreferrer">github.com/gunnargrosch/durable-support-triage</a>. Clone the repo and run <code>npm run demo</code> to try the full workflow locally with mocked Bedrock responses, or deploy to AWS and run it with real Bedrock.</p>

<h2>
  
  
  Getting Started
</h2>

<h3>
  
  
  You'll need:
</h3>

<ul>
<li>An AWS account with credentials configured</li>
<li>
<a href="https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html" rel="noopener noreferrer">AWS SAM CLI</a> 1.153.1 or later (minimum version with <code>DurableConfig</code> support)</li>
<li>Node.js 24 or later</li>
<li>Access to Amazon Bedrock with Claude Haiku 4.5 enabled in your region</li>
</ul>

<h3>
  
  
  Clone and install
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/gunnargrosch/durable-support-triage.git
<span class="nb">cd </span>durable-support-triage
npm <span class="nb">install</span>
</code></pre>

</div>



<h2>
  
  
  The SAM Template
</h2>

<p>Here's the <code>template.yaml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">AWSTemplateFormatVersion</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2010-09-09"</span>
<span class="na">Transform</span><span class="pi">:</span> <span class="s">AWS::Serverless-2016-10-31</span>
<span class="na">Description</span><span class="pi">:</span> <span class="s">AI-powered support ticket triage with durable functions</span>

<span class="na">Parameters</span><span class="pi">:</span>
  <span class="na">BedrockModelId</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">String</span>
    <span class="na">Default</span><span class="pi">:</span> <span class="s">anthropic.claude-haiku-4-5-20251001-v1:0</span>
    <span class="na">Description</span><span class="pi">:</span> <span class="s">Bedrock foundation model ID (uses global inference profile prefix automatically)</span>

<span class="na">Globals</span><span class="pi">:</span>
  <span class="na">Function</span><span class="pi">:</span>
    <span class="na">Timeout</span><span class="pi">:</span> <span class="m">120</span>
    <span class="na">MemorySize</span><span class="pi">:</span> <span class="m">256</span>
    <span class="na">Runtime</span><span class="pi">:</span> <span class="s">nodejs24.x</span>

<span class="na">Resources</span><span class="pi">:</span>
  <span class="na">SupportTriageFunction</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::Serverless::Function</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">FunctionName</span><span class="pi">:</span> <span class="s">durable-support-triage</span>
      <span class="na">Handler</span><span class="pi">:</span> <span class="s">index.handler</span>
      <span class="na">CodeUri</span><span class="pi">:</span> <span class="s">src/</span>
      <span class="na">DurableConfig</span><span class="pi">:</span>
        <span class="na">ExecutionTimeout</span><span class="pi">:</span> <span class="m">604800</span>
        <span class="na">RetentionPeriodInDays</span><span class="pi">:</span> <span class="m">14</span>
      <span class="na">Policies</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">arn:aws:iam::aws:policy/service-role/AWSLambdaBasicDurableExecutionRolePolicy</span>
        <span class="pi">-</span> <span class="na">Version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2012-10-17"</span>
          <span class="na">Statement</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">Effect</span><span class="pi">:</span> <span class="s">Allow</span>
              <span class="na">Action</span><span class="pi">:</span>
                <span class="pi">-</span> <span class="s">bedrock:InvokeModel</span>
              <span class="na">Resource</span><span class="pi">:</span>
                <span class="pi">-</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">arn:aws:bedrock:*::foundation-model/${BedrockModelId}"</span>
                <span class="pi">-</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">arn:aws:bedrock:${AWS::Region}:${AWS::AccountId}:inference-profile/global.${BedrockModelId}"</span>
      <span class="na">AutoPublishAlias</span><span class="pi">:</span> <span class="s">live</span>
      <span class="na">Environment</span><span class="pi">:</span>
        <span class="na">Variables</span><span class="pi">:</span>
          <span class="na">BEDROCK_MODEL_ID</span><span class="pi">:</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">global.${BedrockModelId}"</span>
    <span class="na">Metadata</span><span class="pi">:</span>
      <span class="na">BuildMethod</span><span class="pi">:</span> <span class="s">esbuild</span>
      <span class="na">BuildProperties</span><span class="pi">:</span>
        <span class="na">Minify</span><span class="pi">:</span> <span class="kc">false</span>
        <span class="na">Target</span><span class="pi">:</span> <span class="s">es2022</span>
        <span class="na">EntryPoints</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">index.ts</span>

<span class="na">Outputs</span><span class="pi">:</span>
  <span class="na">FunctionArn</span><span class="pi">:</span>
    <span class="na">Value</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">SupportTriageFunction.Arn</span>
  <span class="na">AliasArn</span><span class="pi">:</span>
    <span class="na">Value</span><span class="pi">:</span> <span class="kt">!Ref</span> <span class="s">SupportTriageFunctionAliaslive</span>
</code></pre>

</div>



<p>A few things to note about this template:</p>

<ul>
<li>
<strong><code>Globals.Timeout: 120</code></strong> is the standard Lambda invocation timeout. It applies to each individual invocation (each replay round), not the overall workflow. Two minutes is plenty for each replay round. <code>ExecutionTimeout</code> in <code>DurableConfig</code> is the total wall-clock time for the entire durable execution.</li>
<li>
<strong><code>DurableConfig</code></strong> is the only new property compared to a standard Lambda function. <code>ExecutionTimeout</code> is in seconds (604,800 = 7 days). The individual callback timeouts handle the per-step boundaries, but the execution timeout is your outer safety net. A ticket that needs specialist review might sit over a weekend, so 7 days gives headroom. <code>RetentionPeriodInDays</code> controls how long execution history is kept (1 to 90 days).</li>
<li>
<strong><code>AutoPublishAlias: live</code></strong> automatically creates a Lambda version and alias on each deploy. This is important for two reasons: durable functions require a qualified ARN (with version or alias) for invocation, and Lambda pins each execution to the version that started it. If you deploy new code while an execution is suspended, replay still uses the original version. This prevents inconsistencies from code changes mid-workflow.</li>
<li>
<strong><code>AWSLambdaBasicDurableExecutionRolePolicy</code></strong> is an AWS managed policy that grants the checkpoint and state permissions your function needs (<code>lambda:CheckpointDurableExecutions</code>, <code>lambda:GetDurableExecutionState</code>) plus the standard CloudWatch Logs permissions.</li>
<li>
<strong>Bedrock IAM</strong> uses a cross-region inference profile (<code>global.</code> prefix on the model ID) so that Bedrock routes requests to whichever region has capacity. The policy needs two resource ARNs: the foundation model (wildcard region, no account ID) and the inference profile (your region and account). The <code>BedrockModelId</code> parameter defaults to Claude Haiku 4.5 but you can override it at deploy time with <code>--parameter-overrides BedrockModelId=&lt;model-id&gt;</code>. Check <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/models-regions.html" rel="noopener noreferrer">Bedrock model availability</a> for what's enabled in your region.</li>
</ul>

<h2>
  
  
  The RISEN Prompt
</h2>

<p>The AI triage uses Amazon Bedrock with Claude Haiku 4.5 to analyze incoming tickets. I'm using the <a href="https://dev.to/gunnargrosch/writing-system-prompts-that-actually-work-the-risen-framework-for-ai-agents-4p94">RISEN framework</a> for the system prompt. RISEN structures prompts into five components: Role, Instructions, Steps, Expectation, and Narrowing. Each component serves a specific purpose, and together they produce consistent, structured output that your code can reliably parse.</p>

<p>Here's the system prompt for the triage agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">TRIAGE_SYSTEM_PROMPT</span> <span class="o">=</span> <span class="s2">`
# Role
You are a senior technical support analyst with 10 years of experience
triaging customer support tickets for a SaaS platform. You specialize in
categorizing issues by severity, identifying root causes, and drafting
professional responses.

# Instructions
Analyze the incoming support ticket and produce a structured triage
assessment with category, priority, sentiment, a suggested response,
and an escalation recommendation.

# Steps
1. Read the ticket subject and body to identify the core issue.
2. Categorize the issue (billing, technical, account, feature-request, other).
3. Assess priority based on business impact and urgency (critical, high, medium, low).
4. Evaluate customer sentiment (frustrated, neutral, positive).
5. Draft a suggested response that acknowledges the issue and outlines next steps.
6. Determine whether the ticket needs specialist escalation.

# Expectation
Return a JSON object with this exact structure:
{
  "category": "billing" | "technical" | "account" | "feature-request" | "other",
  "priority": "critical" | "high" | "medium" | "low",
  "sentiment": "frustrated" | "neutral" | "positive",
  "suggestedResponse": "string",
  "needsEscalation": boolean,
  "escalationReason": "string or null",
  "summary": "One-sentence summary of the issue"
}

# Narrowing
- Return only raw JSON. Do not wrap it in markdown code fences, backticks,
  or any other formatting. No explanation, no preamble, no commentary.
- Do not fabricate account details or order numbers not present in the ticket.
- Do not promise refunds, credits, or policy exceptions in the suggested response.
- needsEscalation MUST be false unless one of these exact conditions is met:
  1. The ticket describes confirmed or suspected data loss.
  2. The ticket describes a security breach, unauthorized access, or credential compromise.
  3. The ticket involves a legal or compliance issue.
  4. The customer tier is "enterprise".
  For all other tickets (billing issues, bugs, feature requests, general questions),
  needsEscalation MUST be false regardless of priority or sentiment.
- Keep the suggested response under 200 words.
`</span><span class="p">;</span>
</code></pre>

</div>



<p>The <strong>Narrowing</strong> section does the heavy lifting for reliability. The explicit numbered escalation conditions prevent the model from over-escalating standard tickets (without these, Haiku flagged a routine CSV bug for specialist review). The constraint against promising refunds keeps the AI from making commitments that only a human should make. The pipe syntax in the Expectation section (<code>"billing" | "technical" | ...</code>) is instructional for the model, not literal JSON. It tells the model which values are valid without requiring a separate schema document. Note that "return only raw JSON" doesn't guarantee it: some models still wrap output in markdown code fences despite the instruction. The handler strips them defensively before parsing.</p>

<h2>
  
  
  The Handler
</h2>

<p>Here's the handler from <code>src/index.ts</code>, trimmed to show the durable execution primitives. The full source (input validation, response parsing, integration stubs) is in the <a href="https://github.com/gunnargrosch/durable-support-triage" rel="noopener noreferrer">repo</a>. Types are defined in <code>src/types.ts</code>. The <code>TRIAGE_SYSTEM_PROMPT</code> shown in the RISEN section above is defined at module scope.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span>
  <span class="nx">withDurableExecution</span><span class="p">,</span> <span class="nx">DurableContext</span><span class="p">,</span>
  <span class="nx">createRetryStrategy</span><span class="p">,</span> <span class="nx">JitterStrategy</span><span class="p">,</span> <span class="nx">defaultSerdes</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws/durable-execution-sdk-js</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BedrockRuntimeClient</span><span class="p">,</span> <span class="nx">InvokeModelCommand</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/client-bedrock-runtime</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">TicketEvent</span><span class="p">,</span> <span class="nx">TriageResult</span><span class="p">,</span> <span class="nx">AgentReview</span><span class="p">,</span> <span class="nx">SpecialistReview</span><span class="p">,</span> <span class="nx">TicketResolution</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./types</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">bedrock</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">BedrockRuntimeClient</span><span class="p">();</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">closeTicket</span><span class="p">(</span>
  <span class="nx">context</span><span class="p">:</span> <span class="nx">DurableContext</span><span class="p">,</span>
  <span class="nx">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">email</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">ticketId</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">response</span><span class="p">:</span> <span class="kr">string</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">parallel</span><span class="p">(</span><span class="nx">name</span><span class="p">,</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">send-reply</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">func</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">ctx</span><span class="p">)</span> <span class="o">=&gt;</span>
        <span class="nx">ctx</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">reply</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="k">await</span> <span class="nf">sendCustomerReply</span><span class="p">(</span><span class="nx">email</span><span class="p">,</span> <span class="nx">ticketId</span><span class="p">,</span> <span class="nx">response</span><span class="p">);</span>
        <span class="p">}),</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">send-survey</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">func</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">ctx</span><span class="p">)</span> <span class="o">=&gt;</span>
        <span class="nx">ctx</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">survey</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="k">await</span> <span class="nf">sendSatisfactionSurvey</span><span class="p">(</span><span class="nx">email</span><span class="p">,</span> <span class="nx">ticketId</span><span class="p">);</span>
        <span class="p">}),</span>
    <span class="p">},</span>
  <span class="p">]);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="nf">withDurableExecution</span><span class="p">(</span>
  <span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nx">TicketEvent</span><span class="p">,</span> <span class="nx">context</span><span class="p">:</span> <span class="nx">DurableContext</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">TicketResolution</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">validateEvent</span><span class="p">(</span><span class="nx">event</span><span class="p">);</span>

    <span class="nx">context</span><span class="p">.</span><span class="nx">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="dl">"</span><span class="s2">Ticket received</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">ticketId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span>
      <span class="na">customerTier</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">customerTier</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="c1">// Step 1: AI analyzes the ticket using Bedrock</span>
    <span class="kd">const</span> <span class="nx">analysis</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span>
      <span class="dl">"</span><span class="s2">analyze-ticket</span><span class="dl">"</span><span class="p">,</span>
      <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">bedrock</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="k">new</span> <span class="nc">InvokeModelCommand</span><span class="p">({</span>
          <span class="na">modelId</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BEDROCK_MODEL_ID</span><span class="p">,</span>
          <span class="na">contentType</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span><span class="p">,</span>
          <span class="na">accept</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span><span class="p">,</span>
          <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
            <span class="na">anthropic_version</span><span class="p">:</span> <span class="dl">"</span><span class="s2">bedrock-2023-05-31</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">max_tokens</span><span class="p">:</span> <span class="mi">1024</span><span class="p">,</span>
            <span class="na">system</span><span class="p">:</span> <span class="nx">TRIAGE_SYSTEM_PROMPT</span><span class="p">,</span>
            <span class="na">messages</span><span class="p">:</span> <span class="p">[</span>
              <span class="p">{</span>
                <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span>
                <span class="na">content</span><span class="p">:</span> <span class="s2">`Ticket ID: </span><span class="p">${</span><span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">}</span><span class="s2">\nCustomer Tier: </span><span class="p">${</span><span class="nx">event</span><span class="p">.</span><span class="nx">customerTier</span><span class="p">}</span><span class="s2">\nSubject: </span><span class="p">${</span><span class="nx">event</span><span class="p">.</span><span class="nx">subject</span><span class="p">}</span><span class="s2">\n\n</span><span class="p">${</span><span class="nx">event</span><span class="p">.</span><span class="nx">body</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
              <span class="p">},</span>
            <span class="p">],</span>
          <span class="p">}),</span>
        <span class="p">}));</span>
        <span class="k">return</span> <span class="nf">parseBedrockResponse</span><span class="p">(</span><span class="nx">response</span><span class="p">.</span><span class="nx">body</span> <span class="k">as</span> <span class="nb">Uint8Array</span><span class="p">);</span>
      <span class="p">},</span>
      <span class="p">{</span>
        <span class="na">retryStrategy</span><span class="p">:</span> <span class="nf">createRetryStrategy</span><span class="p">({</span>
          <span class="na">maxAttempts</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span>
          <span class="na">initialDelay</span><span class="p">:</span> <span class="p">{</span> <span class="na">seconds</span><span class="p">:</span> <span class="mi">2</span> <span class="p">},</span>
          <span class="na">maxDelay</span><span class="p">:</span> <span class="p">{</span> <span class="na">seconds</span><span class="p">:</span> <span class="mi">30</span> <span class="p">},</span>
          <span class="na">backoffRate</span><span class="p">:</span> <span class="mf">2.0</span><span class="p">,</span>
          <span class="na">jitter</span><span class="p">:</span> <span class="nx">JitterStrategy</span><span class="p">.</span><span class="nx">FULL</span><span class="p">,</span>
        <span class="p">}),</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="c1">// Step 2: Support agent reviews AI suggestion</span>
    <span class="kd">const</span> <span class="nx">agentReview</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nx">waitForCallback</span><span class="o">&lt;</span><span class="nx">AgentReview</span><span class="o">&gt;</span><span class="p">(</span>
      <span class="dl">"</span><span class="s2">agent-review</span><span class="dl">"</span><span class="p">,</span>
      <span class="k">async </span><span class="p">(</span><span class="nx">callbackId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nf">notifyAgent</span><span class="p">({</span> <span class="nx">callbackId</span><span class="p">,</span> <span class="na">ticketId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span> <span class="nx">analysis</span><span class="p">,</span> <span class="na">customerTier</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">customerTier</span> <span class="p">});</span>
      <span class="p">},</span>
      <span class="p">{</span> <span class="na">timeout</span><span class="p">:</span> <span class="p">{</span> <span class="na">hours</span><span class="p">:</span> <span class="mi">8</span> <span class="p">},</span> <span class="na">serdes</span><span class="p">:</span> <span class="nx">defaultSerdes</span> <span class="p">}</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">finalResponse</span> <span class="o">=</span> <span class="nx">agentReview</span><span class="p">.</span><span class="nx">editedResponse</span> <span class="o">||</span> <span class="nx">analysis</span><span class="p">.</span><span class="nx">suggestedResponse</span><span class="p">;</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">agentReview</span><span class="p">.</span><span class="nx">approved</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">return</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="dl">"</span><span class="s2">rejected</span><span class="dl">"</span><span class="p">,</span> <span class="na">ticketId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span> <span class="na">category</span><span class="p">:</span> <span class="nx">analysis</span><span class="p">.</span><span class="nx">category</span><span class="p">,</span> <span class="na">priority</span><span class="p">:</span> <span class="nx">analysis</span><span class="p">.</span><span class="nx">priority</span><span class="p">,</span> <span class="na">finalResponse</span><span class="p">:</span> <span class="dl">""</span> <span class="p">};</span>
    <span class="p">}</span>

    <span class="c1">// Step 3: If escalation needed, wait for specialist</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">analysis</span><span class="p">.</span><span class="nx">needsEscalation</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">specialistResponse</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nx">waitForCallback</span><span class="o">&lt;</span><span class="nx">SpecialistReview</span><span class="o">&gt;</span><span class="p">(</span>
        <span class="dl">"</span><span class="s2">specialist-review</span><span class="dl">"</span><span class="p">,</span>
        <span class="k">async </span><span class="p">(</span><span class="nx">callbackId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="k">await</span> <span class="nf">notifySpecialist</span><span class="p">({</span> <span class="nx">callbackId</span><span class="p">,</span> <span class="na">ticketId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span> <span class="nx">analysis</span><span class="p">,</span> <span class="na">agentNotes</span><span class="p">:</span> <span class="nx">agentReview</span><span class="p">.</span><span class="nx">agentNotes</span> <span class="p">});</span>
        <span class="p">},</span>
        <span class="p">{</span> <span class="na">timeout</span><span class="p">:</span> <span class="p">{</span> <span class="na">days</span><span class="p">:</span> <span class="mi">3</span> <span class="p">},</span> <span class="na">serdes</span><span class="p">:</span> <span class="nx">defaultSerdes</span> <span class="p">}</span>
      <span class="p">);</span>

      <span class="kd">const</span> <span class="nx">resolvedResponse</span> <span class="o">=</span> <span class="nx">specialistResponse</span><span class="p">.</span><span class="nx">response</span> <span class="o">||</span> <span class="nx">finalResponse</span><span class="p">;</span>
      <span class="k">await</span> <span class="nf">closeTicket</span><span class="p">(</span><span class="nx">context</span><span class="p">,</span> <span class="dl">"</span><span class="s2">close-escalated-ticket</span><span class="dl">"</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">contactEmail</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span> <span class="nx">resolvedResponse</span><span class="p">);</span>
      <span class="k">return</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="dl">"</span><span class="s2">escalated</span><span class="dl">"</span><span class="p">,</span> <span class="na">ticketId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span> <span class="na">category</span><span class="p">:</span> <span class="nx">analysis</span><span class="p">.</span><span class="nx">category</span><span class="p">,</span> <span class="na">priority</span><span class="p">:</span> <span class="nx">analysis</span><span class="p">.</span><span class="nx">priority</span><span class="p">,</span> <span class="na">finalResponse</span><span class="p">:</span> <span class="nx">resolvedResponse</span> <span class="p">};</span>
    <span class="p">}</span>

    <span class="c1">// Step 4: Send reply and survey in parallel</span>
    <span class="k">await</span> <span class="nf">closeTicket</span><span class="p">(</span><span class="nx">context</span><span class="p">,</span> <span class="dl">"</span><span class="s2">close-ticket</span><span class="dl">"</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">contactEmail</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span> <span class="nx">finalResponse</span><span class="p">);</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="dl">"</span><span class="s2">resolved</span><span class="dl">"</span><span class="p">,</span> <span class="na">ticketId</span><span class="p">:</span> <span class="nx">event</span><span class="p">.</span><span class="nx">ticketId</span><span class="p">,</span> <span class="na">category</span><span class="p">:</span> <span class="nx">analysis</span><span class="p">.</span><span class="nx">category</span><span class="p">,</span> <span class="na">priority</span><span class="p">:</span> <span class="nx">analysis</span><span class="p">.</span><span class="nx">priority</span><span class="p">,</span> <span class="nx">finalResponse</span> <span class="p">};</span>
  <span class="p">}</span>
<span class="p">);</span>
</code></pre>

</div>



<p>Let's walk through what's happening:</p>

<p><strong><code>withDurableExecution</code></strong> wraps your async function and returns a standard Lambda handler. The runtime calls it like any other handler; the SDK intercepts the execution to manage checkpoints. The <code>BedrockRuntimeClient</code> is instantiated at module scope, which is standard Lambda practice for connection reuse across warm-start invocations. Each replay is a new Lambda invocation, but it may reuse a warm container just like any regular invocation.</p>

<p><strong><code>validateEvent</code></strong> runs before the first <code>context.step()</code>. This is intentional: if the payload is malformed, the execution fails immediately instead of after the Bedrock step has already been checkpointed. Durable executions that fail after partial checkpointing are harder to reason about than ones that fail fast. Validation is deterministic and cheap, so re-running it on every replay is harmless.</p>

<p><strong><code>context.step("analyze-ticket", ...)</code></strong> calls Amazon Bedrock with the RISEN prompt and checkpoints the result. The retry strategy handles transient Bedrock API errors (throttling, temporary unavailability) with exponential backoff. <code>parseBedrockResponse</code> handles the response parsing separately: it strips markdown code fences (some models wrap JSON output despite the prompt instruction), validates the response structure, and gives clear error messages on parse failures. If the function replays later, this step returns the cached analysis without calling Bedrock again. That matters for both cost and consistency: you don't want the AI to produce a different triage on replay.</p>

<p><strong><code>context.waitForCallback("agent-review", ...)</code></strong> is where the function suspends. The SDK creates a callback ID and passes it to your submitter function (which sends it to Slack, email, or your ticketing UI). The submitter runs exactly once, on the invocation that creates the callback. On replay, the SDK skips the submitter entirely and returns the callback result directly. This is important: even though the submitter isn't wrapped in a <code>context.step()</code>, it won't re-execute on replay. The SDK then terminates the Lambda function. Compute charges stop. The agent might respond in minutes or hours. When they do, an external system calls <code>SendDurableExecutionCallbackSuccess</code> with their review. Lambda resumes the function from where it left off. The <code>serdes: defaultSerdes</code> option is required for typed callbacks. Without it, the SDK uses passthrough serialization (not <code>JSON.parse</code>) and <code>agentReview.approved</code> would be <code>undefined</code> at runtime even though TypeScript thinks it's a boolean. This isn't documented yet: <code>step()</code> defaults to JSON serdes, but <code>waitForCallback</code> defaults to passthrough. The SDK exports <code>defaultSerdes</code> for this purpose. If the callback times out (8 hours for the agent, 3 days for the specialist), the SDK throws a <code>CallbackTimeoutError</code>. In production, wrap the callback in a try/catch to handle the timeout (re-queue the ticket, notify a manager, or auto-escalate).</p>

<p><strong>If the agent rejects</strong> the AI suggestion (<code>approved: false</code>), the workflow returns early with a <code>rejected</code> status without sending a customer reply. Your ticketing system handles the next step (re-queue, reassign, or manual response).</p>

<p><strong>The escalation path</strong> adds a second <code>waitForCallback</code>. If the AI flagged the ticket for escalation (security concern, data loss, enterprise customer), the function suspends again waiting for a specialist. The specialist sends back a <code>SpecialistReview</code> with a response and notes. This callback has a 3-day timeout because specialist reviews can take time. Without durable functions, you'd need a separate state machine or database to track which tickets are waiting for specialists.</p>

<p><strong><code>context.logger</code></strong> replaces <code>console.log</code>. During replay, completed steps don't re-execute, but code outside steps does. <code>context.logger</code> suppresses duplicate log output during replay so your CloudWatch Logs stay clean. With <code>console.log</code>, you'd see the same log lines repeated on every replay invocation.</p>

<p><strong><code>closeTicket</code></strong> extracts the parallel close-out into a helper. Both the escalation and standard paths send a customer reply and satisfaction survey concurrently using <code>context.parallel</code>. Each branch gets its own child context with isolated state tracking. The helper takes a dynamic context name so the escalated and standard close-out steps are distinguishable in the execution history.</p>

<h2>
  
  
  Testing Locally
</h2>

<p>The testing SDK (<code>@aws/durable-execution-sdk-js-testing</code>) lets you run durable functions locally without deploying. Here's the key pattern from <code>src/index.test.ts</code>, showing how to drive a callback-based workflow in a test:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">LocalDurableTestRunner</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws/durable-execution-sdk-js-testing</span><span class="dl">"</span><span class="p">;</span>

<span class="c1">// jest.mock replaces BedrockRuntimeClient so analyze-ticket returns controlled results</span>
<span class="c1">// (see full mock setup in the repo)</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">handler</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./index</span><span class="dl">"</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">"</span><span class="s2">Support Triage</span><span class="dl">"</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">runner</span><span class="p">:</span> <span class="nx">LocalDurableTestRunner</span><span class="p">;</span>

  <span class="nf">beforeAll</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">LocalDurableTestRunner</span><span class="p">.</span><span class="nf">setupTestEnvironment</span><span class="p">({</span> <span class="na">skipTime</span><span class="p">:</span> <span class="kc">true</span> <span class="p">});</span>
  <span class="p">});</span>

  <span class="nf">afterAll</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">LocalDurableTestRunner</span><span class="p">.</span><span class="nf">teardownTestEnvironment</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">runner</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LocalDurableTestRunner</span><span class="p">({</span> <span class="na">handlerFunction</span><span class="p">:</span> <span class="nx">handler</span> <span class="p">});</span>
  <span class="p">});</span>

  <span class="nf">afterEach</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">runner</span><span class="p">.</span><span class="nf">reset</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">"</span><span class="s2">should resolve a standard ticket after agent review</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">runner</span><span class="p">.</span><span class="nf">run</span><span class="p">({</span>
      <span class="na">payload</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">ticketId</span><span class="p">:</span> <span class="dl">"</span><span class="s2">TKT-001</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">customerId</span><span class="p">:</span> <span class="dl">"</span><span class="s2">CUST-123</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">customerTier</span><span class="p">:</span> <span class="dl">"</span><span class="s2">pro</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">subject</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Cannot export CSV reports</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">body</span><span class="p">:</span> <span class="dl">"</span><span class="s2">When I click the export button, nothing happens.</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">contactEmail</span><span class="p">:</span> <span class="dl">"</span><span class="s2">customer@example.com</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">});</span>

    <span class="c1">// The handler suspends at waitForCallback("agent-review").</span>
    <span class="c1">// getOperation blocks until the callback is created.</span>
    <span class="kd">const</span> <span class="nx">agentCallback</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">runner</span><span class="p">.</span><span class="nf">getOperation</span><span class="p">(</span><span class="dl">"</span><span class="s2">agent-review</span><span class="dl">"</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">agentDetails</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">agentCallback</span><span class="p">.</span><span class="nf">waitForData</span><span class="p">();</span>
    <span class="k">await</span> <span class="nx">agentDetails</span><span class="p">.</span><span class="nf">sendCallbackSuccess</span><span class="p">(</span><span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
      <span class="na">approved</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="na">editedResponse</span><span class="p">:</span> <span class="dl">"</span><span class="s2">We have identified the CSV export issue and a fix is rolling out today.</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">agentNotes</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Known bug, fix in deploy pipeline</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">}));</span>

    <span class="kd">const</span> <span class="nx">output</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">result</span><span class="p">;</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">output</span><span class="p">.</span><span class="nf">getStatus</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">"</span><span class="s2">SUCCEEDED</span><span class="dl">"</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">output</span><span class="p">.</span><span class="nf">getResult</span><span class="p">()).</span><span class="nf">toMatchObject</span><span class="p">({</span> <span class="na">status</span><span class="p">:</span> <span class="dl">"</span><span class="s2">resolved</span><span class="dl">"</span><span class="p">,</span> <span class="na">ticketId</span><span class="p">:</span> <span class="dl">"</span><span class="s2">TKT-001</span><span class="dl">"</span> <span class="p">});</span>
  <span class="p">});</span>

  <span class="c1">// Additional tests in the repo: escalation flow, agent rejection, callback failure</span>
<span class="p">});</span>
</code></pre>

</div>



<p><code>setupTestEnvironment</code> and <code>teardownTestEnvironment</code> are static methods that start and stop the local checkpoint server. They run once per test file in <code>beforeAll</code>/<code>afterAll</code>. The runner instance is created per test in <code>beforeEach</code> and reset in <code>afterEach</code>. The <code>skipTime: true</code> option fast-forwards any <code>context.wait()</code> calls so tests run instantly.</p>

<p>The interesting part is the callback interaction: <code>runner.getOperation("agent-review")</code> blocks until the handler reaches <code>waitForCallback("agent-review")</code> and creates the callback. Then <code>sendCallbackSuccess</code> simulates the external system responding. This lets you test the full suspend/resume lifecycle without deploying. The repo includes tests for all three paths: standard resolution, escalation with specialist review, and agent rejection. There's also a test that uses <code>sendCallbackFailure</code> to verify error handling when an external system reports a failure.</p>

<p>Run the tests with <code>npm test</code>, or use the <code>run-durable</code> CLI to run the handler directly with a payload:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx run-durable <span class="nt">--skip-time</span> <span class="nt">--verbose</span> <span class="nt">--event</span> <span class="s1">'{"ticketId":"TKT-001","subject":"Test ticket"}'</span> src/index.ts
</code></pre>

</div>



<h2>
  
  
  Try the Demo
</h2>

<p>The repo includes an interactive demo that runs the entire workflow in your terminal, showing each stage: AI analysis, human review prompts, checkpoint history, and final resolution.</p>

<p>Run <code>npm run demo</code> to open the interactive menu, or skip it with a direct command:</p>

<h3>
  
  
  Local mode (no AWS credentials needed)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm run demo:local <span class="nt">--</span> <span class="nt">--ticket</span><span class="o">=</span>standard
</code></pre>

</div>



<p>Local mode uses mocked Bedrock responses so you can see the full workflow without calling AWS. Here's what the first few steps look like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  ──────────────────────────────────────────────
  Ticket TKT-001
  Customer:  CUST-123 (pro)
  Subject:   Cannot export CSV reports
  ──────────────────────────────────────────────

  ▶ step: analyze-ticket
    ✓ Checkpointed analyze-ticket (1204ms)

  ──────────────────────────────────────────────
  AI Triage Result
  Category:    technical
  Priority:    high
  Sentiment:   frustrated
  Escalation:  No
  ──────────────────────────────────────────────

  ⏸ waitForCallback: agent-review
    Function suspended. Compute charges stopped.
</code></pre>

</div>



<p>The demo pauses at each callback and prompts you to respond as the agent or specialist. It walks through three ticket scenarios:</p>

<ol>
<li>
<strong>Standard ticket</strong> (pro tier, CSV export bug): AI analyzes, agent approves with edits, reply sent.</li>
<li>
<strong>Enterprise escalation</strong> (security concern): AI flags for escalation, agent approves, specialist reviews, reply sent.</li>
<li>
<strong>Agent rejection</strong> (feature request): AI suggests a response, agent rejects, ticket returned for manual handling.</li>
</ol>

<p>At each human review step, the demo pauses and lets you play the role of the support agent or specialist. You approve, reject, or edit the AI's suggestion. The demo shows the execution history after each step so you can see the checkpoint/replay model in action.</p>

<h3>
  
  
  Cloud mode (real Bedrock responses)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm run demo:cloud <span class="nt">--</span> <span class="nt">--ticket</span><span class="o">=</span>standard <span class="nt">--region</span><span class="o">=</span>us-east-2
</code></pre>

</div>



<p>Cloud mode invokes the deployed Lambda function with real Bedrock calls. You'll see actual AI-generated triage analysis and can watch the durable execution checkpoints in the Lambda console. Add <code>--profile=&lt;name&gt;</code> if you're not using the default AWS profile.</p>

<h2>
  
  
  Deploying and Invoking
</h2>

<p>Build and deploy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>sam build
sam deploy <span class="nt">--guided</span>
</code></pre>

</div>



<p>Once deployed, invoke the function asynchronously with a qualified ARN. The <code>AutoPublishAlias</code> in the template created a <code>live</code> alias:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws lambda invoke <span class="se">\</span>
  <span class="nt">--function-name</span> durable-support-triage:live <span class="se">\</span>
  <span class="nt">--invocation-type</span> Event <span class="se">\</span>
  <span class="nt">--durable-execution-name</span> <span class="s2">"ticket-TKT-001"</span> <span class="se">\</span>
  <span class="nt">--cli-binary-format</span> raw-in-base64-out <span class="se">\</span>
  <span class="nt">--payload</span> <span class="s1">'{
    "ticketId": "TKT-001",
    "customerId": "CUST-123",
    "customerTier": "pro",
    "subject": "Cannot export CSV reports",
    "body": "When I click the export button, nothing happens.",
    "contactEmail": "customer@example.com"
  }'</span> <span class="se">\</span>
  response.json
</code></pre>

</div>



<p>A few things to note:</p>

<ul>
<li>
<strong><code>--invocation-type Event</code></strong> is required for long-running workflows. Synchronous invocation (<code>RequestResponse</code>) times out after 15 minutes.</li>
<li>
<strong><code>--durable-execution-name</code></strong> provides built-in idempotency. If you invoke the function twice with the same execution name, the second invocation returns the existing execution instead of creating a duplicate. Using the ticket ID as the execution name is a natural fit. Note: execution names must be alphanumeric, hyphens, or underscores. If your ticket IDs contain dots, slashes, or other special characters, sanitize them first.</li>
<li>
<strong><code>:live</code></strong> on the function name is the alias qualifier. Without it, you'll get <code>InvalidParameterValueException: Durable execution requires qualified function identifier</code>.</li>
</ul>

<h3>
  
  
  Monitoring execution progress
</h3>

<p>Check execution status in the Lambda console under the <strong>Durable executions</strong> tab. You'll see each step's status and timing, including when the function suspended waiting for the agent callback.</p>

<p>You can also check programmatically:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws lambda get-durable-execution <span class="se">\</span>
  <span class="nt">--durable-execution-arn</span> <span class="s2">"arn:aws:lambda:us-east-2:123456789012:function:durable-support-triage:live/durable-execution/ticket-TKT-001/&lt;run-id&gt;"</span>
</code></pre>

</div>



<p>Replace <code>&lt;run-id&gt;</code> with the run ID returned in the initial <code>invoke</code> response. You can also find it in the Lambda console under the <strong>Durable executions</strong> tab.</p>

<h3>
  
  
  Completing the callbacks
</h3>

<p>When the support agent finishes their review, send the callback from your ticketing system:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws lambda send-durable-execution-callback-success <span class="se">\</span>
  <span class="nt">--callback-id</span> <span class="s2">"your-callback-id-here"</span> <span class="se">\</span>
  <span class="nt">--cli-binary-format</span> raw-in-base64-out <span class="se">\</span>
  <span class="nt">--result</span> <span class="s1">'{
    "approved": true,
    "editedResponse": "Hi, thanks for reporting this. We have identified the issue and a fix is rolling out today.",
    "agentNotes": "Known bug in CSV export module"
  }'</span>
</code></pre>

</div>



<p>The <code>--result</code> value is a JSON string. The CLI handles serialization, so you pass the JSON object directly (unlike the test SDK, where you explicitly call <code>JSON.stringify()</code>).</p>

<p>The function resumes, checks for escalation, and continues. If a specialist callback is pending, the same pattern applies: the specialist's system calls <code>send-durable-execution-callback-success</code> when their review is complete. There's also <code>send-durable-execution-callback-failure</code> for when an external system needs to report an error (e.g., agent rejects the ticket, or an integration fails).</p>

<p>You can also react to execution state changes via EventBridge. Lambda emits events to the default event bus when executions start, succeed, fail, or time out. Create an EventBridge rule with this event pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"source"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"aws.lambda"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"detail-type"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Durable Execution Status Change"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Things I Learned Building This
</h2>

<h3>
  
  
  Determinism is not optional
</h3>

<p>This is the rule that trips people up. Code outside steps re-executes on every replay. If it produces different results each time, your workflow breaks in subtle ways.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// This generates a different ID on each replay</span>
<span class="kd">const</span> <span class="nx">requestId</span> <span class="o">=</span> <span class="nx">crypto</span><span class="p">.</span><span class="nf">randomUUID</span><span class="p">();</span>
<span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">use-id</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">saveToDb</span><span class="p">(</span><span class="nx">requestId</span><span class="p">));</span>

<span class="c1">// This generates one ID, checkpoints it, and returns the same value on replay</span>
<span class="kd">const</span> <span class="nx">requestId</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">gen-id</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">crypto</span><span class="p">.</span><span class="nf">randomUUID</span><span class="p">());</span>
<span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">use-id</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">saveToDb</span><span class="p">(</span><span class="nx">requestId</span><span class="p">));</span>
</code></pre>

</div>



<p>The same applies to <code>Date.now()</code>, <code>Math.random()</code>, API calls, and database queries. If it can return different values, wrap it in a step.</p>

<p>The ESLint plugin (<code>@aws/durable-execution-sdk-js-eslint-plugin</code>) catches common violations. Set it up early.</p>

<h3>
  
  
  Closure mutations are lost on replay
</h3>

<p>Variables you modify inside a step are not preserved across replays:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">let</span> <span class="nx">total</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
<span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">calculate</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">total</span> <span class="o">=</span> <span class="mi">42</span><span class="p">;</span> <span class="c1">// This mutation is lost on replay</span>
<span class="p">});</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">total</span><span class="p">);</span> <span class="c1">// Always 0 on replay</span>

<span class="c1">// Instead, return values from steps</span>
<span class="nx">total</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">calculate</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="mi">42</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">total</span><span class="p">);</span> <span class="c1">// Always 42</span>
</code></pre>

</div>



<p>This is because the step function doesn't re-execute on replay. It returns the cached result. But the closure variable was modified by the function body, which never ran. Return values from steps instead of modifying outer variables.</p>

<h3>
  
  
  The qualified ARN requirement is easy to miss
</h3>

<p>Durable functions require a qualified function identifier: a version number, alias, or <code>$LATEST</code>. Using an unqualified ARN silently fails or throws <code>InvalidParameterValueException</code>.</p>

<p>The <code>AutoPublishAlias</code> SAM property solves this. It creates a new version and updates the alias on every deploy. If you're using EventBridge Scheduler or other services to invoke your function, make sure they target the alias ARN, not the unqualified ARN.</p>

<h3>
  
  
  More steps means slower replay
</h3>

<p>Every time your function resumes, the SDK replays from the beginning, returning cached results for completed steps. The more steps you have, the more replay overhead per resumption.</p>

<p>This is a trade-off. More granular steps give you better debuggability and more precise retry boundaries. Fewer steps give you faster replay. In practice, group related operations into a single step unless you need separate retry behavior or checkpoint boundaries.</p>

<h3>
  
  
  context.wait() is the simplest superpower
</h3>

<p>The "How Checkpoint/Replay Works" section mentions <code>context.wait()</code> for fixed-duration pauses, but the handler only uses <code>waitForCallback()</code>. In practice, <code>context.wait()</code> is one of the most useful primitives. Need a 24-hour cooling-off period before sending a follow-up? One line:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">wait</span><span class="p">(</span><span class="dl">"</span><span class="s2">cooling-off</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">hours</span><span class="p">:</span> <span class="mi">24</span> <span class="p">});</span>
</code></pre>

</div>



<p>The function suspends, compute charges stop, and Lambda resumes it 24 hours later. No cron jobs, no EventBridge Scheduler, no polling.</p>

<h3>
  
  
  You can't enable durable execution on existing functions
</h3>

<p><code>DurableConfig</code> can only be set when creating a function. You can't toggle it on an existing function. If you need to migrate, you'll need to create a new function. Plan for this.</p>

<p>Changing <code>DurableConfig</code> in CloudFormation also requires resource replacement, not an in-place update.</p>

<h3>
  
  
  Checkpoint payloads have a 256KB limit
</h3>

<p>Each step result is serialized and stored as a checkpoint. If a step returns an object larger than 256KB, you'll get a <code>CheckpointUnrecoverableExecutionError</code>.</p>

<p>The workaround: store large data in S3 or DynamoDB and return a reference from the step.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">dataRef</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">"</span><span class="s2">store-large-data</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">key</span> <span class="o">=</span> <span class="s2">`tickets/</span><span class="p">${</span><span class="nx">ticketId</span><span class="p">}</span><span class="s2">/attachments.json`</span><span class="p">;</span>
  <span class="k">await</span> <span class="nx">s3</span><span class="p">.</span><span class="nf">putObject</span><span class="p">({</span> <span class="na">Bucket</span><span class="p">:</span> <span class="nx">bucket</span><span class="p">,</span> <span class="na">Key</span><span class="p">:</span> <span class="nx">key</span><span class="p">,</span> <span class="na">Body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">largeData</span><span class="p">)</span> <span class="p">});</span>
  <span class="k">return</span> <span class="p">{</span> <span class="nx">bucket</span><span class="p">,</span> <span class="nx">key</span> <span class="p">};</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Know your failure modes
</h3>

<p>If a step throws an unrecoverable error (after all retries are exhausted), the execution moves to a <code>FAILED</code> state. You can inspect the error in the Lambda console or via the <code>get-durable-execution</code> API. For cases where you want to fail immediately without retries, throw an <code>UnrecoverableInvocationError</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">UnrecoverableInvocationError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws/durable-execution-sdk-js</span><span class="dl">"</span><span class="p">;</span>
<span class="k">throw</span> <span class="k">new</span> <span class="nc">UnrecoverableInvocationError</span><span class="p">(</span><span class="dl">"</span><span class="s2">Customer account not found</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<p>Failed executions can't be resumed. You'd need to start a new execution. Design your workflows so that steps are idempotent in case you need to re-run from scratch.</p>

<h3>
  
  
  Use Lambda versions for deploy safety
</h3>

<p>If you update your function code while an execution is suspended, replay uses the version that started the execution. This prevents inconsistencies from code changes mid-workflow. <code>AutoPublishAlias</code> handles this, but it's worth understanding why: if your new code changes a step's return shape or removes a step, replay on the old version still works because Lambda pins executions to their starting version.</p>

<p>Version pinning protects your function code, but it doesn't protect external schemas. If an execution suspends on Monday waiting for a callback, and on Wednesday your ticketing system starts sending a different JSON structure in the callback payload, the Monday execution will fail when it resumes. Keep callback payloads backwards compatible for as long as executions can be in flight.</p>

<h3>
  
  
  Plan your observability early
</h3>

<p>For production workflows that can run for days, go beyond basic CloudWatch Logs. Set up CloudWatch alarms on stuck executions (no state change within expected timeframes), use the EventBridge integration to track execution lifecycle events, and consider CloudWatch Logs Insights queries for filtering by execution name across replays.</p>

<h2>
  
  
  When to Use What
</h2>

<p>Durable functions and Step Functions are not competing. They solve different problems.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th></th>
<th>Durable Functions</th>
<th>Step Functions</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Workflow definition</strong></td>
<td>Sequential code in your language</td>
<td>Amazon States Language (JSON/YAML)</td>
</tr>
<tr>
<td><strong>Best for</strong></td>
<td>Application logic, tightly coupled workflows</td>
<td>Cross-service orchestration</td>
</tr>
<tr>
<td><strong>Service integrations</strong></td>
<td>Via SDK in your code</td>
<td>220+ native integrations</td>
</tr>
<tr>
<td><strong>Debugging</strong></td>
<td>CloudWatch Logs, execution history</td>
<td>Visual console, step-by-step</td>
</tr>
<tr>
<td><strong>Infrastructure</strong></td>
<td>Single Lambda function</td>
<td>State machine + Lambda functions</td>
</tr>
<tr>
<td><strong>Scaling</strong></td>
<td>Lambda concurrency limits</td>
<td>Distributed Map for large-scale parallel processing</td>
</tr>
<tr>
<td><strong>Mental model</strong></td>
<td>Write code</td>
<td>Design state machines</td>
</tr>
</tbody>
</table></div>

<p>On cost: durable functions use standard Lambda pricing for active compute time. During waits, compute charges stop. Step Functions charges per state transition, which adds up for high-volume workflows. See <a href="https://aws.amazon.com/lambda/pricing/" rel="noopener noreferrer">Lambda pricing</a> and <a href="https://aws.amazon.com/step-functions/pricing/" rel="noopener noreferrer">Step Functions pricing</a> for current details.</p>

<p><strong>Use durable functions when</strong> your workflow is application logic that reads naturally as sequential code. Support ticket triage, approval workflows, AI agent loops, saga patterns.</p>

<p><strong>Use Step Functions when</strong> you're orchestrating across multiple AWS services, need visual debugging, or need the 220+ native integrations. ETL pipelines, media processing, infrastructure provisioning.</p>

<p><strong>Use both together</strong> when you have a high-level orchestration (Step Functions) that delegates to individual workflows (durable functions) for complex application logic.</p>

<h2>
  
  
  What's Next
</h2>

<p>This post covered the fundamentals: what durable functions are, how checkpoint/replay works, and how to build and test a complete AI-powered workflow with human-in-the-loop callbacks. In the next post, I'll use durable functions to build a multi-agent orchestration workflow where multiple AI agents collaborate on complex tasks with checkpointed reasoning.</p>

<h2>
  
  
  Additional Resources
</h2>

<ul>
<li><a href="https://docs.aws.amazon.com/lambda/latest/dg/durable-functions.html" rel="noopener noreferrer">AWS Lambda Durable Functions documentation</a></li>
<li><a href="https://aws.amazon.com/blogs/aws/build-multi-step-applications-and-ai-workflows-with-aws-lambda-durable-functions/" rel="noopener noreferrer">AWS Blog: Build multi-step applications and AI workflows</a></li>
<li><a href="https://github.com/aws/aws-durable-execution-sdk-js" rel="noopener noreferrer">Durable Execution SDK for JavaScript (GitHub)</a></li>
<li><a href="https://github.com/aws-samples/sample-lambda-durable-functions" rel="noopener noreferrer">Sample Lambda Durable Functions (GitHub)</a></li>
<li><a href="https://dev.to/gunnargrosch/writing-system-prompts-that-actually-work-the-risen-framework-for-ai-agents-4p94">The RISEN Framework for AI Agent System Prompts</a></li>
<li><a href="https://github.com/gunnargrosch/durable-support-triage" rel="noopener noreferrer">Demo repository for this post</a></li>
</ul>

<p>What workflows are you thinking about building with durable functions? Let me know in the comments!</p>

