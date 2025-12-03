---
Title: AWS Lambda Durable Functions: Build Workflows That Last
Description: 
Author: Eric D Johnson
Date: 2025-12-03T21:33:02.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Long-running workflows without managing infrastructure</em></p>

<p>Your Lambda function needs to wait for a human approval. Or retry a failed API call with exponential backoff. Or orchestrate multiple steps that span hours. How do you build that without managing servers or databases?</p>

<p>AWS Lambda Durable Functions solve this. Write your workflow in your programming language—Node.js, TypeScript, Python, with more coming—using straightforward async code. Lambda handles the rest: checkpointing state, resuming after waits, retrying issues, and scaling automatically. Workflows can run for up to a year, and you only pay for actual execution time—not while waiting.</p>

<h2>
  
  
  What Are Durable Functions?
</h2>

<p>Durable functions are Lambda functions that can pause and resume. When your function waits for a callback or sleeps for an hour, Lambda checkpoints its state and stops execution. When it's time to continue, Lambda resumes exactly where it left off—with all variables and context intact.</p>

<p>This isn't a new compute model. It's regular Lambda with automatic state management. You write normal async/await code. Lambda makes it durable.</p>

<h2>
  
  
  A Simple Example
</h2>

<p>Here's a workflow that creates an order, waits 5 minutes, then sends a notification:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">DurableContext</span><span class="p">,</span> <span class="nx">withDurableExecution</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@aws/durable-execution-sdk-js</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="nf">withDurableExecution</span><span class="p">(</span>
  <span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="kr">any</span><span class="p">,</span> <span class="nx">context</span><span class="p">:</span> <span class="nx">DurableContext</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">order</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">create-order</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">return</span> <span class="nf">createOrder</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">items</span><span class="p">);</span>
    <span class="p">});</span>

    <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">wait</span><span class="p">({</span> <span class="na">seconds</span><span class="p">:</span> <span class="mi">300</span> <span class="p">});</span>

    <span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">send-notification</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">return</span> <span class="nf">sendEmail</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">customerId</span><span class="p">,</span> <span class="nx">order</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
    <span class="p">});</span>

    <span class="k">return</span> <span class="p">{</span> <span class="na">orderId</span><span class="p">:</span> <span class="nx">order</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">completed</span><span class="dl">'</span> <span class="p">};</span>
  <span class="p">}</span>
<span class="p">);</span>
</code></pre>

</div>



<p>That's it. No state machines to configure, no databases to manage, no polling loops. The function pauses during the wait, costs nothing while idle, and resumes automatically after 5 minutes.</p>

<h2>
  
  
  Key Capabilities
</h2>

<p><strong>Long execution times</strong> - Functions can run for up to 1 year. Individual invocations are still limited to 15 minutes, but the workflow continues across multiple invocations.</p>

<p><strong>Automatic checkpointing</strong> - Lambda saves your function's state at each step. If something fails, the function resumes from the last checkpoint—not from the beginning.</p>

<p><strong>Built-in retries</strong> - Configure retry strategies with exponential backoff. Lambda handles the retry logic and timing automatically.</p>

<p><strong>Wait for callbacks</strong> - Pause execution until an external event arrives. Perfect for human approvals, webhook responses, or async API results.</p>

<p><strong>Parallel execution</strong> - Run multiple operations concurrently and wait for all to complete. Lambda manages the coordination.</p>

<p><strong>Nested workflows</strong> - Invoke other durable functions and compose complex workflows from simple building blocks.</p>

<h2>
  
  
  How It Works: The Replay Model
</h2>

<p>Durable functions use a replay-based execution model. When your function resumes, Lambda replays it from the start—but instead of re-executing operations, it uses checkpointed results.</p>

<p>Here's what happens:</p>

<ol>
<li>
<strong>First invocation</strong> - Your function runs, executing each step and checkpointing results</li>
<li>
<strong>Wait or callback</strong> - Function pauses, Lambda saves state and stops execution</li>
<li>
<strong>Resume</strong> - Lambda invokes your function again, replaying from the start</li>
<li>
<strong>Replay</strong> - Operations return checkpointed results instantly instead of re-executing</li>
<li>
<strong>Continue</strong> - Function continues past the wait with all context intact</li>
</ol>

<p>This model ensures your function always sees consistent state, even across issues and restarts. Operations are deterministic—they execute once and replay with the same result.</p>

<p><strong>Learn more:</strong> <a href="https://dev.to/aws/the-replay-model-how-aws-lambda-durable-functions-actually-work-2a79">Understanding the Replay Model</a> explains how replay works, why operations must be deterministic, and how to handle non-deterministic code safely.</p>

<h2>
  
  
  Common Use Cases
</h2>

<p><strong>Approval workflows</strong> - Wait for human approval before proceeding. The function pauses until someone clicks approve or reject.</p>

<p><strong>Saga patterns</strong> - Coordinate distributed transactions with compensating actions. If a step fails, automatically roll back previous steps.</p>

<p><strong>Scheduled tasks</strong> - Wait for specific times or intervals. Process data at midnight, send reminders after 24 hours, or retry every 5 minutes.</p>

<p><strong>API orchestration</strong> - Call multiple APIs with retries and error handling. Coordinate responses and handle partial issues gracefully.</p>

<p><strong>Data processing pipelines</strong> - Process large datasets in stages with checkpoints. Resume from the last successful stage if something fails.</p>

<p><strong>Event-driven workflows</strong> - React to external events like webhooks, IoT signals, or user actions. Wait for events and continue processing when they arrive.</p>

<h2>
  
  
  Testing Your Workflows
</h2>

<p>Testing long-running workflows doesn't mean waiting hours. The Durable Execution SDK includes a testing library that runs your functions locally in milliseconds:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">LocalDurableTestRunner</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@aws/durable-execution-sdk-js-testing</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">runner</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LocalDurableTestRunner</span><span class="p">({</span>
  <span class="na">handlerFunction</span><span class="p">:</span> <span class="nx">handler</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">execution</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">runner</span><span class="p">.</span><span class="nf">run</span><span class="p">();</span>

<span class="nf">expect</span><span class="p">(</span><span class="nx">execution</span><span class="p">.</span><span class="nf">getStatus</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">SUCCEEDED</span><span class="dl">'</span><span class="p">);</span>
<span class="nf">expect</span><span class="p">(</span><span class="nx">execution</span><span class="p">.</span><span class="nf">getResult</span><span class="p">()).</span><span class="nf">toEqual</span><span class="p">({</span> <span class="na">orderId</span><span class="p">:</span> <span class="dl">'</span><span class="s1">123</span><span class="dl">'</span><span class="p">,</span> <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">completed</span><span class="dl">'</span> <span class="p">});</span>
</code></pre>

</div>



<p>The test runner simulates checkpoints, skips time-based waits, and lets you inspect every operation. You can test callbacks, retries, and issues without deploying to AWS.</p>

<p><strong>Learn more:</strong> <a href="https://dev.to/aws/testing-aws-lambda-durable-functions-in-typescript-5bj2">Testing Durable Functions</a> covers local testing, cloud integration tests, debugging techniques, and best practices.</p>

<h2>
  
  
  Deploying with AWS SAM
</h2>

<p>Deploy durable functions using AWS SAM with a few key configurations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">Resources</span><span class="pi">:</span>
  <span class="na">OrderProcessorFunction</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::Serverless::Function</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">CodeUri</span><span class="pi">:</span> <span class="s">src/order-processor</span>
      <span class="na">Handler</span><span class="pi">:</span> <span class="s">index.handler</span>
      <span class="na">Runtime</span><span class="pi">:</span> <span class="s">nodejs22.x</span>
      <span class="na">DurableConfig</span><span class="pi">:</span>
        <span class="na">ExecutionTimeout</span><span class="pi">:</span> <span class="m">900</span>
        <span class="na">RetentionPeriodInDays</span><span class="pi">:</span> <span class="m">7</span>
    <span class="na">Metadata</span><span class="pi">:</span>
      <span class="na">BuildMethod</span><span class="pi">:</span> <span class="s">esbuild</span>
      <span class="na">BuildProperties</span><span class="pi">:</span>
        <span class="na">EntryPoints</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="s">index.ts</span>
</code></pre>

</div>



<p>The <code>DurableConfig</code> property enables durable execution and sets the workflow timeout. SAM automatically handles IAM permissions for checkpointing and state management.</p>

<p><strong>Learn more:</strong> <a href="https://dev.to/aws/developing-aws-lambda-durable-functions-with-aws-sam-ga9">Deploying Durable Functions with SAM</a> covers template configuration, permissions, build settings, and deployment best practices.</p>

<h2>
  
  
  When to Use Durable Functions
</h2>

<ul>
<li>Your workflow spans multiple steps with waits or callbacks</li>
<li>You need automatic retries with exponential backoff</li>
<li>You want to coordinate multiple async operations</li>
<li>Your process requires human approval or external events</li>
<li>You need to handle long-running tasks without managing state</li>
<li>You prefer writing workflows as code rather than configuration</li>
</ul>

<h2>
  
  
  Getting Started
</h2>

<ol>
<li>
<strong>Install the SDK</strong>: <code>npm install @aws/durable-execution-sdk-js</code>
</li>
<li>
<strong>Write your function</strong>: Wrap your handler with <code>withDurableExecution()</code>
</li>
<li>
<strong>Use durable operations</strong>: <code>context.step()</code>, <code>context.wait()</code>, <code>context.waitForCallback()</code>
</li>
<li>
<strong>Test locally</strong>: Use <code>LocalDurableTestRunner</code> for fast iteration</li>
<li>
<strong>Deploy with SAM</strong>: Add <code>DurableConfig</code> to your template</li>
<li>
<strong>Monitor execution</strong>: Use Amazon CloudWatch and AWS X-Ray for observability</li>
</ol>

<h2>
  
  
  Learn More
</h2>

<ul>
<li>
<strong><a href="https://dev.to/aws/the-replay-model-how-aws-lambda-durable-functions-actually-work-2a79">Understanding the Replay Model</a></strong> - Deep dive into how durable functions work under the hood</li>
<li>
<strong><a href="https://dev.to/aws/testing-aws-lambda-durable-functions-in-typescript-5bj2">Testing Durable Functions</a></strong> - Comprehensive guide to testing workflows locally and in the cloud</li>
<li>
<strong><a href="https://dev.to/aws/developing-aws-lambda-durable-functions-with-aws-sam-ga9">Deploying with AWS SAM</a></strong> - Complete deployment guide with templates and best practices</li>
</ul>

<h2>
  
  
  Summary
</h2>

<p>AWS Lambda Durable Functions let you build long-running workflows without managing infrastructure. Write straightforward async code, and Lambda handles state management, retries, and resumption. Your functions can wait for callbacks, retry issues, and run for up to a year—all while paying only for execution time.</p>

<p>Start with simple workflows, test locally for fast iteration, and deploy with confidence knowing Lambda manages the complexity of distributed state.</p>

