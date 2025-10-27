---
Title: 🕒 Automating Periodic Tasks with EventBridge Scheduler, .NET, and CloudFormation
Description: 
Author: Bassem Hussein 
Date: 2025-10-27T21:35:10.000Z
Robots: noindex,nofollow
Template: index
---
<p>Many applications need to perform tasks on a recurring schedule — refreshing data, syncing APIs, or cleaning up records. Instead of managing EC2 cron jobs or containers, AWS gives you a fully managed option: <strong>Amazon EventBridge Scheduler</strong>.  </p>

<p>In this post, we’ll build a simple proof of concept (POC) showing how to trigger a <strong>.NET Lambda function</strong> on a schedule using <strong>AWS CloudFormation</strong>.<br><br>
We’ll also discuss what to do if your job takes longer than the Lambda time limit of <strong>15 minutes</strong>.</p>


<h2>
  
  
  🧩 What You’ll Build
</h2>

<p>You’ll deploy:</p>

<ul>
<li>A <strong>.NET 8 Lambda function</strong> that simulates fetching stock prices
</li>
<li>A <strong>CloudFormation template</strong> that defines:

<ul>
<li>The Lambda function and its IAM role
</li>
<li>An <strong>EventBridge Scheduler</strong> that runs every 10 minutes
</li>
<li>An <strong>SQS Dead Letter Queue (DLQ)</strong> for failed invocations
</li>
</ul>
</li>
</ul>

<p>Here’s the architecture:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz83cbyvoyy5w6fmvyn4q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz83cbyvoyy5w6fmvyn4q.png" alt="Architecture" width="462" height="451"></a></p>


<h2>
  
  
  ⚙️ Step 1. Create a .NET Lambda Function
</h2>

<p>Start by creating a simple .NET 8 Lambda that logs a simulated stock price.</p>
<h3>
  
  
  🧱 Project Setup
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>dotnet new lambda.empty-function <span class="nt">-n</span> StockPriceUpdater
<span class="nb">cd </span>StockPriceUpdater
</code></pre>

</div>

<h3>
  
  
  🧠 Function Code (<code>Function.cs</code>)
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System.Text.Json</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Amazon.Lambda.Core</span><span class="p">;</span>

<span class="p">[</span><span class="n">assembly</span><span class="p">:</span> <span class="nf">LambdaSerializer</span><span class="p">(</span><span class="k">typeof</span><span class="p">(</span><span class="n">Amazon</span><span class="p">.</span><span class="n">Lambda</span><span class="p">.</span><span class="n">Serialization</span><span class="p">.</span><span class="n">SystemTextJson</span><span class="p">.</span><span class="n">DefaultLambdaJsonSerializer</span><span class="p">))]</span>

<span class="k">namespace</span> <span class="nn">StockPriceUpdater</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">ScheduledJobInput</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">JobType</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>
    <span class="k">public</span> <span class="kt">object</span><span class="p">?</span> <span class="n">Payload</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">Payload</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">AccountCode</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="n">Empty</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">Function</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">InvokeAsync</span><span class="p">(</span><span class="n">ScheduledJobInput</span> <span class="n">input</span><span class="p">,</span> <span class="n">ILambdaContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">context</span><span class="p">.</span><span class="n">Logger</span><span class="p">.</span><span class="nf">LogLine</span><span class="p">(</span><span class="s">$"[StockPriceUpdater] Triggered for JobType: </span><span class="p">{</span><span class="n">input</span><span class="p">.</span><span class="n">JobType</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>

        <span class="k">if</span> <span class="p">(</span><span class="n">input</span><span class="p">.</span><span class="n">Payload</span> <span class="k">is</span> <span class="n">JsonElement</span> <span class="n">payloadJson</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="kt">var</span> <span class="n">payload</span> <span class="p">=</span> <span class="n">payloadJson</span><span class="p">.</span><span class="n">Deserialize</span><span class="p">&lt;</span><span class="n">Payload</span><span class="p">&gt;();</span>
            <span class="n">context</span><span class="p">.</span><span class="n">Logger</span><span class="p">.</span><span class="nf">LogLine</span><span class="p">(</span><span class="s">$"AccountCode: </span><span class="p">{</span><span class="n">payload</span><span class="p">?.</span><span class="n">AccountCode</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="k">await</span> <span class="nf">UpdateStockPricesAsync</span><span class="p">(</span><span class="n">input</span><span class="p">.</span><span class="n">JobType</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="n">Task</span> <span class="nf">UpdateStockPricesAsync</span><span class="p">(</span><span class="kt">string</span> <span class="n">jobType</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">price</span> <span class="p">=</span> <span class="n">Math</span><span class="p">.</span><span class="nf">Round</span><span class="p">(</span><span class="k">new</span> <span class="nf">Random</span><span class="p">().</span><span class="nf">NextDouble</span><span class="p">()</span> <span class="p">*</span> <span class="m">100</span><span class="p">,</span> <span class="m">2</span><span class="p">);</span>
        <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">$"[</span><span class="p">{</span><span class="n">jobType</span><span class="p">}</span><span class="s">] Simulated stock price: $</span><span class="p">{</span><span class="n">price</span><span class="p">}</span><span class="s"> at </span><span class="p">{</span><span class="n">DateTime</span><span class="p">.</span><span class="n">UtcNow</span><span class="p">:</span><span class="n">O</span><span class="p">}</span><span class="s">"</span><span class="p">);</span>
        <span class="k">return</span> <span class="n">Task</span><span class="p">.</span><span class="n">CompletedTask</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>


<p>This function accepts a small JSON payload and logs a random “stock price.”<br><br>
It’s a simple stand-in for any periodic background task you might need.</p>


<h2>
  
  
  📜 Step 2. Deploy with CloudFormation
</h2>

<p>Save the following as <strong><code>stock-price-updater.yml</code></strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">AWSTemplateFormatVersion</span><span class="pi">:</span> <span class="s1">'</span><span class="s">2010-09-09'</span>
<span class="na">Description</span><span class="pi">:</span> <span class="s">Lambda triggered by EventBridge Scheduler for periodic stock updates</span>

<span class="na">Parameters</span><span class="pi">:</span>
  <span class="na">ResourcePrefix</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">String</span>
    <span class="na">Default</span><span class="pi">:</span> <span class="s">stock-updater</span>
  <span class="na">LambdaCodeBucket</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">String</span>
    <span class="na">Description</span><span class="pi">:</span> <span class="s">S3 bucket where Lambda zip is stored</span>
  <span class="na">LambdaCodeKey</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">String</span>
    <span class="na">Description</span><span class="pi">:</span> <span class="s">Key (file name) of Lambda zip in S3</span>

<span class="na">Resources</span><span class="pi">:</span>
  <span class="c1">##################################################</span>
  <span class="c1"># IAM Role for Lambda</span>
  <span class="c1">##################################################</span>
  <span class="na">StockPriceUpdaterRole</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::IAM::Role</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">RoleName</span><span class="pi">:</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">${ResourcePrefix}-lambda-role"</span>
      <span class="na">AssumeRolePolicyDocument</span><span class="pi">:</span>
        <span class="na">Version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">2012-10-17'</span>
        <span class="na">Statement</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">Effect</span><span class="pi">:</span> <span class="s">Allow</span>
            <span class="na">Principal</span><span class="pi">:</span>
              <span class="na">Service</span><span class="pi">:</span> <span class="s">lambda.amazonaws.com</span>
            <span class="na">Action</span><span class="pi">:</span> <span class="s">sts:AssumeRole</span>
      <span class="na">ManagedPolicyArns</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole</span>

  <span class="c1">##################################################</span>
  <span class="c1"># Lambda Function</span>
  <span class="c1">##################################################</span>
  <span class="na">StockPriceUpdaterFunction</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::Lambda::Function</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">FunctionName</span><span class="pi">:</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">${ResourcePrefix}-function"</span>
      <span class="na">Handler</span><span class="pi">:</span> <span class="s">StockPriceUpdater::StockPriceUpdater.Function::InvokeAsync</span>
      <span class="na">Role</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">StockPriceUpdaterRole.Arn</span>
      <span class="na">Runtime</span><span class="pi">:</span> <span class="s">dotnet8</span>
      <span class="na">Timeout</span><span class="pi">:</span> <span class="m">600</span> <span class="c1"># 10 minutes (max is 900 = 15 minutes)</span>
      <span class="na">MemorySize</span><span class="pi">:</span> <span class="m">512</span>
      <span class="na">Code</span><span class="pi">:</span>
        <span class="na">S3Bucket</span><span class="pi">:</span> <span class="kt">!Ref</span> <span class="s">LambdaCodeBucket</span>
        <span class="na">S3Key</span><span class="pi">:</span> <span class="kt">!Ref</span> <span class="s">LambdaCodeKey</span>

  <span class="c1">##################################################</span>
  <span class="c1"># Dead Letter Queue (DLQ)</span>
  <span class="c1">##################################################</span>
  <span class="na">StockPriceUpdaterDLQ</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::SQS::Queue</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">QueueName</span><span class="pi">:</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">${ResourcePrefix}-dlq"</span>
      <span class="na">MessageRetentionPeriod</span><span class="pi">:</span> <span class="m">1209600</span> <span class="c1"># 14 days</span>

  <span class="c1">##################################################</span>
  <span class="c1"># Role for EventBridge Scheduler</span>
  <span class="c1">##################################################</span>
  <span class="na">SchedulerInvokeRole</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::IAM::Role</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">RoleName</span><span class="pi">:</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">${ResourcePrefix}-scheduler-role"</span>
      <span class="na">AssumeRolePolicyDocument</span><span class="pi">:</span>
        <span class="na">Version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2012-10-17"</span>
        <span class="na">Statement</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">Effect</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Allow"</span>
            <span class="na">Principal</span><span class="pi">:</span>
              <span class="na">Service</span><span class="pi">:</span> <span class="s2">"</span><span class="s">scheduler.amazonaws.com"</span>
            <span class="na">Action</span><span class="pi">:</span> <span class="s2">"</span><span class="s">sts:AssumeRole"</span>
      <span class="na">Policies</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">PolicyName</span><span class="pi">:</span> <span class="s">SchedulerInvokeLambdaAndDLQ</span>
          <span class="na">PolicyDocument</span><span class="pi">:</span>
            <span class="na">Version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">2012-10-17"</span>
            <span class="na">Statement</span><span class="pi">:</span>
              <span class="pi">-</span> <span class="na">Effect</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Allow"</span>
                <span class="na">Action</span><span class="pi">:</span> <span class="s2">"</span><span class="s">lambda:InvokeFunction"</span>
                <span class="na">Resource</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">StockPriceUpdaterFunction.Arn</span>
              <span class="pi">-</span> <span class="na">Effect</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Allow"</span>
                <span class="na">Action</span><span class="pi">:</span> <span class="s2">"</span><span class="s">sqs:SendMessage"</span>
                <span class="na">Resource</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">StockPriceUpdaterDLQ.Arn</span>

  <span class="c1">##################################################</span>
  <span class="c1"># EventBridge Scheduler</span>
  <span class="c1">##################################################</span>
  <span class="na">StockPriceUpdaterSchedule</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::Scheduler::Schedule</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">Name</span><span class="pi">:</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">${ResourcePrefix}-scheduler"</span>
      <span class="na">Description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Triggers</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">Lambda</span><span class="nv"> </span><span class="s">every</span><span class="nv"> </span><span class="s">10</span><span class="nv"> </span><span class="s">minutes"</span>
      <span class="na">FlexibleTimeWindow</span><span class="pi">:</span>
        <span class="na">Mode</span><span class="pi">:</span> <span class="s2">"</span><span class="s">OFF"</span>
      <span class="na">ScheduleExpression</span><span class="pi">:</span> <span class="s2">"</span><span class="s">rate(10</span><span class="nv"> </span><span class="s">minutes)"</span>
      <span class="na">State</span><span class="pi">:</span> <span class="s2">"</span><span class="s">ENABLED"</span>
      <span class="na">Target</span><span class="pi">:</span>
        <span class="na">Arn</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">StockPriceUpdaterFunction.Arn</span>
        <span class="na">RoleArn</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">SchedulerInvokeRole.Arn</span>
        <span class="na">DeadLetterConfig</span><span class="pi">:</span>
          <span class="na">Arn</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">StockPriceUpdaterDLQ.Arn</span>
        <span class="na">Input</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">{</span>
            <span class="s">"JobType": "StockUpdate",</span>
            <span class="s">"Payload": { "AccountCode": "Primary" }</span>
          <span class="s">}</span>

<span class="na">Outputs</span><span class="pi">:</span>
  <span class="na">LambdaFunctionName</span><span class="pi">:</span>
    <span class="na">Value</span><span class="pi">:</span> <span class="kt">!Ref</span> <span class="s">StockPriceUpdaterFunction</span>
    <span class="na">Description</span><span class="pi">:</span> <span class="s">Name of the Lambda function</span>
</code></pre>

</div>






<h2>
  
  
  🚀 Step 3. Package and Deploy
</h2>

<ol>
<li>
<strong>Package the Lambda</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>dotnet lambda package <span class="nt">--output-package</span> ./StockPriceUpdater.zip
aws s3 <span class="nb">cp</span> ./StockPriceUpdater.zip s3://your-bucket/
</code></pre>

</div>



<ol>
<li>
<strong>Deploy the stack</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws cloudformation deploy <span class="se">\</span>
  <span class="nt">--template-file</span> stock-price-updater.yml <span class="se">\</span>
  <span class="nt">--stack-name</span> StockPriceUpdater <span class="se">\</span>
  <span class="nt">--capabilities</span> CAPABILITY_IAM <span class="se">\</span>
  <span class="nt">--parameter-overrides</span> <span class="se">\</span>
      <span class="nv">ResourcePrefix</span><span class="o">=</span>stock-updater <span class="se">\</span>
      <span class="nv">LambdaCodeBucket</span><span class="o">=</span>your-bucket <span class="se">\</span>
      <span class="nv">LambdaCodeKey</span><span class="o">=</span>StockPriceUpdater.zip
</code></pre>

</div>






<h2>
  
  
  🧾 Step 4. Verify It Works
</h2>

<p>After deployment:</p>

<ul>
<li>Go to <strong>Amazon EventBridge → Scheduler → Schedules</strong> — you’ll see one schedule.</li>
<li>Open <strong>CloudWatch Logs</strong> for your Lambda and verify logs every 10 minutes:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[StockPriceUpdater] Triggered for JobType: StockUpdate
AccountCode: Primary
[StockUpdate] Simulated stock price: $74.82 at 2025-10-27T09:00:00Z
</code></pre>

</div>



<p>Failures are sent automatically to the <strong>DLQ (SQS)</strong>.</p>




<h2>
  
  
  ⚠️ Lambda Execution Time Limit (15 Minutes)
</h2>

<p>AWS Lambda has a <strong>hard timeout limit of 15 minutes</strong> per execution.  </p>

<p>If your background job might exceed this:</p>

<ul>
<li>
<strong>Step Functions</strong>: Chain multiple Lambdas or tasks with retries and parallel steps.
</li>
<li>
<strong>AWS Fargate</strong>: Run containerized tasks without worrying about execution time.
</li>
<li>
<strong>AWS Batch</strong>: Run large-scale batch jobs with automatic compute scaling.
</li>
</ul>

<p>For long-running tasks, <strong>replace Lambda with Step Functions, Fargate, or Batch</strong> triggered by EventBridge Scheduler.</p>




<h2>
  
  
  🧭 Wrapping Up
</h2>

<p>You learned how to:</p>

<ul>
<li>Use <strong>EventBridge Scheduler</strong> to trigger a <strong>.NET Lambda</strong> periodically
</li>
<li>Handle failures with an <strong>SQS DLQ</strong>
</li>
<li>Deploy everything with <strong>CloudFormation</strong>
</li>
</ul>

<p>This approach is <strong>serverless, scalable, and fully managed</strong>, ideal for background jobs, periodic updates, or monitoring tasks.</p>




<h3>
  
  
  ✅ Key Takeaways
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>EventBridge Scheduler</strong></td>
<td>Replaces cron jobs with a managed scheduler</td>
</tr>
<tr>
<td><strong>Lambda</strong></td>
<td>Ideal for short, frequent tasks</td>
</tr>
<tr>
<td><strong>DLQ (SQS)</strong></td>
<td>Captures failed events for later inspection</td>
</tr>
<tr>
<td><strong>CloudFormation</strong></td>
<td>Declarative, repeatable deployments</td>
</tr>
<tr>
<td><strong>Limit</strong></td>
<td>Max Lambda duration = 15 minutes — use Step Functions, Fargate, or Batch for longer jobs</td>
</tr>
</tbody>
</table></div>

