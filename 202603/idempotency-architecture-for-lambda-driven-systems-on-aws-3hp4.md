---
Title: Idempotency Architecture for Lambda-Driven Systems on AWS
Description: 
Author: Renaldi
Date: 2026-03-22T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Duplicate processing is one of those problems that looks small in a diagram and very expensive in production.</p>

<p>I have seen teams build clean event-driven and Lambda-based systems, only to run into duplicate charges, duplicated emails, repeated downstream writes, or inconsistent state once retries and redrives start happening. The tricky part is that the system is often behaving <em>as designed</em>. AWS services are doing what they should do: retrying, buffering, redriving, and favoring delivery durability.</p>

<p>This is exactly why I consider idempotency architecture one of the most important and most underexplained topics in serverless engineering.</p>

<p>In this post, I will walk through how I design idempotency for Lambda-driven systems on AWS, including:</p>

<ul>
<li>the <strong>exactly-once myth</strong> vs the <strong>at-least-once reality</strong>
</li>
<li>how to choose <strong>idempotency keys</strong> and <strong>dedupe windows</strong>
</li>
<li>a <strong>DynamoDB-based idempotency store</strong> design</li>
<li>using <strong>AWS Lambda Powertools idempotency utility</strong>
</li>
<li>handling retries from <strong>API Gateway</strong>, <strong>SQS</strong>, and <strong>EventBridge</strong>
</li>
<li>an <strong>end-to-end implementation walkthrough</strong> with code</li>
</ul>

<p>I will keep this practical and architecture-heavy, so you can adapt it to real workloads instead of only toy examples.</p>




<h2>
  
  
  The core idea in one sentence
</h2>

<p><strong>Idempotency means I can safely process the same logical request more than once and still end up with the same intended outcome.</strong></p>

<p>That does <strong>not</strong> mean the system only receives the request once. It means my system is resilient when it receives it multiple times.</p>




<h2>
  
  
  Exactly-once is usually not the right mental model
</h2>

<p>A lot of production mistakes start with the assumption that a serverless flow will process each request exactly once.</p>

<p>In practice, in distributed systems, what I usually get is:</p>

<ul>
<li>
<strong>at-least-once delivery</strong> (common with queues/events)</li>
<li>
<strong>retries</strong> at multiple layers (client SDKs, AWS services, Lambda, Step Functions, etc.)</li>
<li>
<strong>timeouts and ambiguous outcomes</strong> (did the function finish but the caller timed out?)</li>
<li>
<strong>redrives / replay</strong> (DLQ reprocessing, archive replay, manual reruns)</li>
<li>
<strong>duplicate submissions</strong> from clients (double-click, refresh, mobile reconnect)</li>
</ul>

<p>So instead of trying to force an “exactly-once” guarantee everywhere, I design for:</p>

<ol>
<li><strong>at-least-once delivery</strong></li>
<li><strong>idempotent handlers</strong></li>
<li><strong>safe retries</strong></li>
<li><strong>observability around duplicates</strong></li>
</ol>

<p>That mental shift makes the architecture much more robust.</p>




<h2>
  
  
  Where duplicates come from in AWS Lambda-driven systems
</h2>

<p>Before I show the solution, I like to make the duplicate paths explicit.</p>

<h3>
  
  
  API Gateway -&gt; Lambda
</h3>

<p>Duplicates can happen when:</p>

<ul>
<li>the client retries after a timeout</li>
<li>the network drops after the backend succeeded</li>
<li>the user taps “Submit” multiple times</li>
<li>an upstream reverse proxy retries</li>
</ul>

<h3>
  
  
  SQS -&gt; Lambda
</h3>

<p>Duplicates can happen when:</p>

<ul>
<li>Lambda fails and the message becomes visible again</li>
<li>processing exceeds visibility timeout</li>
<li>partial batch failures cause some records to be retried</li>
<li>DLQ redrive sends records back later</li>
<li>standard queues deliver the same message more than once</li>
</ul>

<h3>
  
  
  EventBridge -&gt; Lambda
</h3>

<p>Duplicates can happen when:</p>

<ul>
<li>target invocation is retried</li>
<li>a publisher emits semantically duplicate events</li>
<li>archive/replay is used</li>
<li>consumers reprocess historical events intentionally</li>
</ul>

<p>That is why I architect idempotency at the <strong>business operation</strong> level, not at just one transport layer.</p>




<h2>
  
  
  What a good idempotency architecture looks like
</h2>

<p>At a high level, I want:</p>

<ul>
<li>a <strong>stable idempotency key</strong> for each logical operation</li>
<li>a <strong>dedupe window</strong> (TTL) appropriate for the business</li>
<li>a <strong>persistence store</strong> to track processing status and cached results</li>
<li>
<strong>conditional writes</strong> to prevent concurrent duplicate execution</li>
<li>
<strong>response replay</strong> for safe duplicate requests when appropriate</li>
<li>
<strong>clear behavior</strong> for mismatched payloads using the same key</li>
</ul>




<h2>
  
  
  Architecture at a glance
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn4lcms01s4mj7ge8rxh8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn4lcms01s4mj7ge8rxh8.png" alt=" " width="800" height="242"></a></p>




<h2>
  
  
  End-to-end walkthrough (the scenario I will implement)
</h2>

<p>To make this concrete, I will use a common example:</p>

<p><strong>“Create payment intent / order processing”</strong> style operation.</p>

<h3>
  
  
  Flow
</h3>

<ol>
<li>A client sends <code>POST /payments</code> with an <code>Idempotency-Key</code> header.</li>
<li>API Gateway invokes Lambda.</li>
<li>Lambda checks DynamoDB idempotency table.</li>
<li>If the key is new, Lambda acquires an <strong>IN_PROGRESS lock</strong> and processes the request.</li>
<li>Lambda writes the business result (for example, a payment record) and stores the response in the idempotency table with status <strong>COMPLETED</strong>.</li>
<li>If the same request is retried, Lambda returns the cached response instead of processing again.</li>
</ol>

<p>Then I will extend the same pattern to:</p>

<ul>
<li><strong>SQS worker retries</strong></li>
<li><strong>EventBridge consumer retries/replay</strong></li>
</ul>




<h2>
  
  
  Designing the idempotency key
</h2>

<p>This is where a lot of teams accidentally introduce bugs.</p>

<p>A good idempotency key should identify the <strong>logical operation</strong>, not just the transport envelope.</p>

<h3>
  
  
  Good key examples
</h3>

<ul>
<li><code>payment:{merchant_id}:{client_request_id}</code></li>
<li><code>order-create:{tenant_id}:{cart_checkout_id}</code></li>
<li><code>invoice-email:{invoice_id}:{template_version}</code></li>
<li>
<code>event:{event_id}</code> (if the publisher guarantees a stable event ID)</li>
</ul>

<h3>
  
  
  Risky key choices
</h3>

<ul>
<li>raw timestamp</li>
<li>Lambda <code>aws_request_id</code> (changes every invocation)</li>
<li>SQS <code>receiptHandle</code> (changes on delivery)</li>
<li>entire payload serialized without normalization (field order / formatting issues)</li>
<li>keys that are too broad (cause false dedupe)</li>
<li>keys that are too narrow (miss duplicates)</li>
</ul>

<h3>
  
  
  My rule of thumb
</h3>

<p>I choose a key from <strong>business identity + operation intent</strong>, and I define it explicitly in the contract.</p>

<p>For APIs, that often means:</p>

<ul>
<li>require an <code>Idempotency-Key</code> header from the client, and</li>
<li>validate that it maps to a stable request intent.</li>
</ul>

<p>For asynchronous consumers, that often means:</p>

<ul>
<li>use the publisher’s stable <code>eventId</code>, or</li>
<li>derive a deterministic business key (for example <code>order_id + action</code>).</li>
</ul>




<h2>
  
  
  Dedupe windows (TTL): how long should I remember a key?
</h2>

<p>There is no universal value. I set the dedupe window based on business and retry patterns.</p>

<h3>
  
  
  What affects the dedupe window
</h3>

<ul>
<li>expected client retry duration</li>
<li>SQS redrive timing and replay operations</li>
<li>EventBridge replay windows / operational reruns</li>
<li>whether duplicates after long periods are still harmful</li>
<li>cost of storing idempotency records</li>
</ul>

<h3>
  
  
  Practical examples
</h3>

<ul>
<li>
<strong>API payment creation</strong>: 24 hours to 7 days</li>
<li>
<strong>Webhook ingestion</strong>: 1 to 7 days (depends on provider retry policy)</li>
<li>
<strong>Batch event processing</strong>: hours to days</li>
<li>
<strong>high-volume telemetry</strong>: maybe minutes to hours (if duplicate impact is low)</li>
</ul>

<h3>
  
  
  Important nuance
</h3>

<p>TTL in DynamoDB is <strong>eventually applied</strong>, not immediate deletion. I design my logic so:</p>

<ul>
<li>
<code>expiry_timestamp</code> is authoritative in code, and</li>
<li>TTL is the cleanup mechanism.</li>
</ul>

<p>In other words, I do not depend on the item disappearing exactly at expiry time.</p>




<h2>
  
  
  DynamoDB-based idempotency store design (recommended pattern)
</h2>

<p>I prefer DynamoDB for idempotency state in Lambda workloads because it gives me:</p>

<ul>
<li>low-latency key lookups</li>
<li>conditional writes</li>
<li>TTL support</li>
<li>simple scaling</li>
<li>good fit for stateless Lambda functions</li>
</ul>

<h3>
  
  
  Table design (single-purpose table)
</h3>

<p>A dedicated table keeps the pattern easy to reason about.</p>

<p><strong>Partition key</strong></p>

<ul>
<li>
<code>id</code> (string): the idempotency key</li>
</ul>

<p><strong>Recommended attributes</strong></p>

<ul>
<li>
<code>status</code> (<code>IN_PROGRESS</code>, <code>COMPLETED</code>, optionally <code>EXPIRED</code>)</li>
<li>
<code>expiryTimestamp</code> (epoch seconds for dedupe window)</li>
<li>
<code>inProgressExpiryTimestamp</code> (shorter lock expiry to recover from crashed executions)</li>
<li>
<code>payloadHash</code> (optional but highly recommended)</li>
<li>
<code>responseData</code> (optional, cached result or safe response envelope)</li>
<li><code>createdAt</code></li>
<li><code>updatedAt</code></li>
<li>
<code>source</code> (api / sqs / eventbridge)</li>
<li>
<code>functionName</code> (optional for ops visibility)</li>
</ul>

<h3>
  
  
  Why <code>payloadHash</code> matters
</h3>

<p>If a client reuses the same idempotency key with a <strong>different payload</strong>, I want to detect that and reject it. Otherwise I can accidentally return a cached response for the wrong request.</p>

<p>This is a subtle but critical best practice.</p>




<h2>
  
  
  State transitions I use
</h2>

<p>Here is the lifecycle I generally implement:</p>

<ol>
<li>
<strong>No record exists</strong>

<ul>
<li>Try conditional write -&gt; create <code>IN_PROGRESS</code>
</li>
</ul>
</li>
<li>
<strong><code>IN_PROGRESS</code> exists</strong>

<ul>
<li>Another invocation is already processing (or crashed recently)</li>
<li>Return a retryable outcome or fail fast depending on source</li>
</ul>
</li>
<li>
<strong><code>COMPLETED</code> exists and not expired</strong>

<ul>
<li>Return cached result (or safe ack)</li>
</ul>
</li>
<li>
<strong>Expired</strong>

<ul>
<li>Treat as a new request (depending on business policy)</li>
</ul>
</li>
</ol>

<p>That gives me concurrency safety <em>and</em> retry safety.</p>




<h2>
  
  
  Infrastructure example (SAM / CloudFormation snippets)
</h2>

<p>Below is a minimal setup for:</p>

<ul>
<li>Lambda function</li>
<li>DynamoDB idempotency table</li>
<li>IAM permissions</li>
<li>env vars for configuration
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">AWSTemplateFormatVersion</span><span class="pi">:</span> <span class="s1">'</span><span class="s">2010-09-09'</span>
<span class="na">Transform</span><span class="pi">:</span> <span class="s">AWS::Serverless-2016-10-31</span>
<span class="na">Description</span><span class="pi">:</span> <span class="s">Idempotent Lambda API example</span>

<span class="na">Resources</span><span class="pi">:</span>
  <span class="na">PaymentsFunction</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::Serverless::Function</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">Runtime</span><span class="pi">:</span> <span class="s">python3.12</span>
      <span class="na">Handler</span><span class="pi">:</span> <span class="s">app.lambda_handler</span>
      <span class="na">CodeUri</span><span class="pi">:</span> <span class="s">src/</span>
      <span class="na">MemorySize</span><span class="pi">:</span> <span class="m">512</span>
      <span class="na">Timeout</span><span class="pi">:</span> <span class="m">29</span>
      <span class="na">Policies</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">Statement</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">Effect</span><span class="pi">:</span> <span class="s">Allow</span>
              <span class="na">Action</span><span class="pi">:</span>
                <span class="pi">-</span> <span class="s">dynamodb:GetItem</span>
                <span class="pi">-</span> <span class="s">dynamodb:PutItem</span>
                <span class="pi">-</span> <span class="s">dynamodb:UpdateItem</span>
                <span class="pi">-</span> <span class="s">dynamodb:DeleteItem</span>
              <span class="na">Resource</span><span class="pi">:</span> <span class="kt">!GetAtt</span> <span class="s">IdempotencyTable.Arn</span>
      <span class="na">Environment</span><span class="pi">:</span>
        <span class="na">Variables</span><span class="pi">:</span>
          <span class="na">IDEMPOTENCY_TABLE_NAME</span><span class="pi">:</span> <span class="kt">!Ref</span> <span class="s">IdempotencyTable</span>
          <span class="na">IDEMPOTENCY_EXPIRES_SECONDS</span><span class="pi">:</span> <span class="s2">"</span><span class="s">86400"</span> <span class="c1"># 24h</span>
      <span class="na">Events</span><span class="pi">:</span>
        <span class="na">CreatePaymentApi</span><span class="pi">:</span>
          <span class="na">Type</span><span class="pi">:</span> <span class="s">Api</span>
          <span class="na">Properties</span><span class="pi">:</span>
            <span class="na">Path</span><span class="pi">:</span> <span class="s">/payments</span>
            <span class="na">Method</span><span class="pi">:</span> <span class="s">post</span>

  <span class="na">IdempotencyTable</span><span class="pi">:</span>
    <span class="na">Type</span><span class="pi">:</span> <span class="s">AWS::DynamoDB::Table</span>
    <span class="na">Properties</span><span class="pi">:</span>
      <span class="na">BillingMode</span><span class="pi">:</span> <span class="s">PAY_PER_REQUEST</span>
      <span class="na">AttributeDefinitions</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">AttributeName</span><span class="pi">:</span> <span class="s">id</span>
          <span class="na">AttributeType</span><span class="pi">:</span> <span class="s">S</span>
      <span class="na">KeySchema</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">AttributeName</span><span class="pi">:</span> <span class="s">id</span>
          <span class="na">KeyType</span><span class="pi">:</span> <span class="s">HASH</span>
      <span class="na">TimeToLiveSpecification</span><span class="pi">:</span>
        <span class="na">AttributeName</span><span class="pi">:</span> <span class="s">expiration</span>
        <span class="na">Enabled</span><span class="pi">:</span> <span class="kc">true</span>
      <span class="na">PointInTimeRecoverySpecification</span><span class="pi">:</span>
        <span class="na">PointInTimeRecoveryEnabled</span><span class="pi">:</span> <span class="kc">true</span>
      <span class="na">SSESpecification</span><span class="pi">:</span>
        <span class="na">SSEEnabled</span><span class="pi">:</span> <span class="kc">true</span>

<span class="na">Outputs</span><span class="pi">:</span>
  <span class="na">ApiUrl</span><span class="pi">:</span>
    <span class="na">Value</span><span class="pi">:</span> <span class="kt">!Sub</span> <span class="s2">"</span><span class="s">https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/payments"</span>
</code></pre>

</div>



<h3>
  
  
  Notes on the table schema vs Powertools
</h3>

<p>AWS Lambda Powertools idempotency utility has its own default attribute names and record model. If I use the library, I usually let it manage the item shape and only customize when I truly need to.</p>

<p>That said, I still think through the conceptual schema above so the team understands what is being stored and why.</p>




<h2>
  
  
  Implementation option 1 (recommended): AWS Lambda Powertools idempotency utility
</h2>

<p>If I am using Python Lambda functions, AWS Lambda Powertools is my default choice. It saves me from reimplementing concurrency locks, conditional checks, and record state transitions from scratch.</p>

<h3>
  
  
  Install
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>aws-lambda-powertools[boto3]
</code></pre>

</div>






<h2>
  
  
  API Gateway example with idempotency (Python)
</h2>

<p>This example assumes the client sends:</p>

<ul>
<li>
<code>Idempotency-Key</code> header</li>
<li>JSON body containing payment details</li>
</ul>

<p>I use:</p>

<ul>
<li>
<code>event_key_jmespath</code> to extract the idempotency key from headers</li>
<li>
<code>payload_validation_jmespath</code> to validate that key reuse with different payloads is detected</li>
<li>a DynamoDB persistence layer
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Any</span><span class="p">,</span> <span class="n">Dict</span>

<span class="kn">from</span> <span class="n">aws_lambda_powertools</span> <span class="kn">import</span> <span class="n">Logger</span>
<span class="kn">from</span> <span class="n">aws_lambda_powertools.utilities.idempotency</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">DynamoDBPersistenceLayer</span><span class="p">,</span>
    <span class="n">IdempotencyConfig</span><span class="p">,</span>
    <span class="n">idempotent</span><span class="p">,</span>
<span class="p">)</span>
<span class="kn">from</span> <span class="n">aws_lambda_powertools.utilities.idempotency.exceptions</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">IdempotencyValidationError</span><span class="p">,</span>
<span class="p">)</span>

<span class="n">logger</span> <span class="o">=</span> <span class="nc">Logger</span><span class="p">(</span><span class="n">service</span><span class="o">=</span><span class="sh">"</span><span class="s">payments-api</span><span class="sh">"</span><span class="p">)</span>

<span class="n">TABLE_NAME</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="sh">"</span><span class="s">IDEMPOTENCY_TABLE_NAME</span><span class="sh">"</span><span class="p">]</span>
<span class="n">EXPIRES_SECONDS</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">IDEMPOTENCY_EXPIRES_SECONDS</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">86400</span><span class="sh">"</span><span class="p">))</span>

<span class="n">persistence_layer</span> <span class="o">=</span> <span class="nc">DynamoDBPersistenceLayer</span><span class="p">(</span><span class="n">table_name</span><span class="o">=</span><span class="n">TABLE_NAME</span><span class="p">)</span>

<span class="n">config</span> <span class="o">=</span> <span class="nc">IdempotencyConfig</span><span class="p">(</span>
    <span class="c1"># API Gateway/Lambda proxy event header lookup (normalize to lower-case in code if needed)
</span>    <span class="n">event_key_jmespath</span><span class="o">=</span><span class="sh">"</span><span class="s">headers.idempotency-key</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># Payload fields that should remain consistent when reusing the same key
</span>    <span class="n">payload_validation_jmespath</span><span class="o">=</span><span class="sh">"</span><span class="s">powertools_json(body).[customerId, amount, currency]</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">expires_after_seconds</span><span class="o">=</span><span class="n">EXPIRES_SECONDS</span><span class="p">,</span>
    <span class="n">use_local_cache</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
<span class="p">)</span>

<span class="k">def</span> <span class="nf">create_payment_intent</span><span class="p">(</span><span class="n">request</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="c1"># Replace this with your real business logic / external payment call.
</span>    <span class="c1"># The critical point is that this function is wrapped with idempotency.
</span>    <span class="n">payment_id</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">pay_</span><span class="si">{</span><span class="n">request</span><span class="p">[</span><span class="sh">'</span><span class="s">customerId</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">_</span><span class="si">{</span><span class="n">request</span><span class="p">[</span><span class="sh">'</span><span class="s">amount</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="s">_</span><span class="si">{</span><span class="n">request</span><span class="p">[</span><span class="sh">'</span><span class="s">currency</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span>
    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">paymentId</span><span class="sh">"</span><span class="p">:</span> <span class="n">payment_id</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">AUTHORIZED</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">amount</span><span class="sh">"</span><span class="p">:</span> <span class="n">request</span><span class="p">[</span><span class="sh">"</span><span class="s">amount</span><span class="sh">"</span><span class="p">],</span>
        <span class="sh">"</span><span class="s">currency</span><span class="sh">"</span><span class="p">:</span> <span class="n">request</span><span class="p">[</span><span class="sh">"</span><span class="s">currency</span><span class="sh">"</span><span class="p">],</span>
    <span class="p">}</span>

<span class="nd">@idempotent</span><span class="p">(</span><span class="n">config</span><span class="o">=</span><span class="n">config</span><span class="p">,</span> <span class="n">persistence_store</span><span class="o">=</span><span class="n">persistence_layer</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">process_request</span><span class="p">(</span><span class="n">event</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">],</span> <span class="n">context</span><span class="p">:</span> <span class="n">Any</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="n">body</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">event</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">)</span> <span class="ow">or</span> <span class="sh">"</span><span class="s">{}</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">required</span> <span class="o">=</span> <span class="p">[</span><span class="sh">"</span><span class="s">customerId</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">amount</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">currency</span><span class="sh">"</span><span class="p">]</span>
    <span class="n">missing</span> <span class="o">=</span> <span class="p">[</span><span class="n">k</span> <span class="k">for</span> <span class="n">k</span> <span class="ow">in</span> <span class="n">required</span> <span class="k">if</span> <span class="n">k</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">body</span><span class="p">]</span>
    <span class="k">if</span> <span class="n">missing</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">statusCode</span><span class="sh">"</span><span class="p">:</span> <span class="mi">400</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">({</span><span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Missing required fields: </span><span class="si">{</span><span class="n">missing</span><span class="si">}</span><span class="sh">"</span><span class="p">})</span>
        <span class="p">}</span>

    <span class="n">result</span> <span class="o">=</span> <span class="nf">create_payment_intent</span><span class="p">(</span><span class="n">body</span><span class="p">)</span>

    <span class="c1"># Powertools can persist and return this response for duplicate calls
</span>    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">statusCode</span><span class="sh">"</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">result</span><span class="p">),</span>
        <span class="sh">"</span><span class="s">headers</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">Content-Type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">application/json</span><span class="sh">"</span><span class="p">},</span>
    <span class="p">}</span>

<span class="nd">@logger.inject_lambda_context</span><span class="p">(</span><span class="n">log_event</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">lambda_handler</span><span class="p">(</span><span class="n">event</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">],</span> <span class="n">context</span><span class="p">:</span> <span class="n">Any</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="c1"># Normalize headers so event_key_jmespath is predictable
</span>    <span class="n">headers</span> <span class="o">=</span> <span class="n">event</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">headers</span><span class="sh">"</span><span class="p">)</span> <span class="ow">or</span> <span class="p">{}</span>
    <span class="n">event</span><span class="p">[</span><span class="sh">"</span><span class="s">headers</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="nf">str</span><span class="p">(</span><span class="n">k</span><span class="p">).</span><span class="nf">lower</span><span class="p">():</span> <span class="n">v</span> <span class="k">for</span> <span class="n">k</span><span class="p">,</span> <span class="n">v</span> <span class="ow">in</span> <span class="n">headers</span><span class="p">.</span><span class="nf">items</span><span class="p">()}</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">process_request</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="n">context</span><span class="p">)</span>
    <span class="k">except</span> <span class="n">IdempotencyValidationError</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">statusCode</span><span class="sh">"</span><span class="p">:</span> <span class="mi">409</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">({</span>
                <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Idempotency key was reused with a different request payload</span><span class="sh">"</span>
            <span class="p">}),</span>
            <span class="sh">"</span><span class="s">headers</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">Content-Type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">application/json</span><span class="sh">"</span><span class="p">},</span>
        <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Why this pattern works well
</h3>

<ul>
<li>If the same request is retried, Powertools returns the stored response.</li>
<li>If the same key is reused with a different body, I return <code>409 Conflict</code>.</li>
<li>I avoid duplicate payment authorization for simple retries.</li>
</ul>




<h2>
  
  
  API client contract (important and often skipped)
</h2>

<p>Idempotency works much better when the contract is explicit.</p>

<p>For API producers/clients, I document:</p>

<ul>
<li>
<code>Idempotency-Key</code> is required for mutating operations (<code>POST</code>, sometimes <code>PATCH</code>)</li>
<li>same key + same intent/payload -&gt; safe retry</li>
<li>same key + different payload -&gt; <code>409 Conflict</code>
</li>
<li>dedupe window (for example, 24h)</li>
<li>response replay behavior (cached result may be returned)</li>
</ul>

<p>That avoids ambiguity across frontend, mobile, and backend teams.</p>




<h2>
  
  
  Handling retries from SQS (Lambda event source mapping)
</h2>

<p>SQS is one of the most common places where teams need idempotency but only discover it after duplicates happen.</p>

<h3>
  
  
  What changes for SQS?
</h3>

<ul>
<li>Lambda receives a <strong>batch</strong> of messages.</li>
<li>Some messages may succeed while others fail.</li>
<li>I should use <strong>partial batch response</strong> so only failed records are retried.</li>
<li>Each message should still be processed idempotently.</li>
</ul>

<h3>
  
  
  Key design for SQS consumers
</h3>

<p>I avoid using transient delivery metadata. I prefer:</p>

<ul>
<li>a business key in the message body (for example <code>orderId</code>)</li>
<li>or an upstream event ID included in the message</li>
</ul>

<p>Example key:</p>

<ul>
<li><code>order-paid:{orderId}</code></li>
<li><code>inventory-reservation:{reservationId}</code></li>
</ul>




<h2>
  
  
  SQS consumer example (Powertools Batch + per-record idempotency)
</h2>

<p>Below is a simplified Python example using:</p>

<ul>
<li>Powertools Batch utility for partial batch handling</li>
<li>Powertools idempotency on the per-record processing function
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Any</span><span class="p">,</span> <span class="n">Dict</span>

<span class="kn">from</span> <span class="n">aws_lambda_powertools</span> <span class="kn">import</span> <span class="n">Logger</span>
<span class="kn">from</span> <span class="n">aws_lambda_powertools.utilities.batch</span> <span class="kn">import</span> <span class="n">BatchProcessor</span><span class="p">,</span> <span class="n">EventType</span><span class="p">,</span> <span class="n">process_partial_response</span>
<span class="kn">from</span> <span class="n">aws_lambda_powertools.utilities.batch.types</span> <span class="kn">import</span> <span class="n">PartialItemFailureResponse</span>
<span class="kn">from</span> <span class="n">aws_lambda_powertools.utilities.data_classes.sqs_event</span> <span class="kn">import</span> <span class="n">SQSRecord</span>
<span class="kn">from</span> <span class="n">aws_lambda_powertools.utilities.idempotency</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">DynamoDBPersistenceLayer</span><span class="p">,</span>
    <span class="n">IdempotencyConfig</span><span class="p">,</span>
    <span class="n">idempotent_function</span><span class="p">,</span>
<span class="p">)</span>

<span class="n">logger</span> <span class="o">=</span> <span class="nc">Logger</span><span class="p">(</span><span class="n">service</span><span class="o">=</span><span class="sh">"</span><span class="s">orders-sqs-worker</span><span class="sh">"</span><span class="p">)</span>

<span class="n">TABLE_NAME</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="sh">"</span><span class="s">IDEMPOTENCY_TABLE_NAME</span><span class="sh">"</span><span class="p">]</span>
<span class="n">persistence_layer</span> <span class="o">=</span> <span class="nc">DynamoDBPersistenceLayer</span><span class="p">(</span><span class="n">table_name</span><span class="o">=</span><span class="n">TABLE_NAME</span><span class="p">)</span>

<span class="c1"># Here we build the key from the function argument "data"
# (Powertools hashes the configured subset to create an idempotency key)
</span><span class="n">idempotency_config</span> <span class="o">=</span> <span class="nc">IdempotencyConfig</span><span class="p">(</span>
    <span class="n">event_key_jmespath</span><span class="o">=</span><span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">payload_validation_jmespath</span><span class="o">=</span><span class="sh">"</span><span class="s">[orderId, action, version]</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">expires_after_seconds</span><span class="o">=</span><span class="mi">3</span> <span class="o">*</span> <span class="mi">24</span> <span class="o">*</span> <span class="mi">60</span> <span class="o">*</span> <span class="mi">60</span><span class="p">,</span>  <span class="c1"># 3 days
</span><span class="p">)</span>

<span class="n">processor</span> <span class="o">=</span> <span class="nc">BatchProcessor</span><span class="p">(</span><span class="n">event_type</span><span class="o">=</span><span class="n">EventType</span><span class="p">.</span><span class="n">SQS</span><span class="p">)</span>

<span class="nd">@idempotent_function</span><span class="p">(</span><span class="n">data_keyword_argument</span><span class="o">=</span><span class="sh">"</span><span class="s">data</span><span class="sh">"</span><span class="p">,</span> <span class="n">config</span><span class="o">=</span><span class="n">idempotency_config</span><span class="p">,</span> <span class="n">persistence_store</span><span class="o">=</span><span class="n">persistence_layer</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">process_order_event</span><span class="p">(</span><span class="o">*</span><span class="p">,</span> <span class="n">data</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="c1"># Business logic here (must be safe to retry / replay via idempotency)
</span>    <span class="c1"># Example: reserve inventory, update status, publish follow-up event, etc.
</span>    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">Processing order event</span><span class="sh">"</span><span class="p">,</span> <span class="n">extra</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">:</span> <span class="n">data</span><span class="p">[</span><span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">],</span> <span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="p">:</span> <span class="n">data</span><span class="p">[</span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="p">]})</span>
    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">ok</span><span class="sh">"</span><span class="p">:</span> <span class="bp">True</span><span class="p">,</span> <span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">:</span> <span class="n">data</span><span class="p">[</span><span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">],</span> <span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="p">:</span> <span class="n">data</span><span class="p">[</span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="p">]}</span>

<span class="k">def</span> <span class="nf">record_handler</span><span class="p">(</span><span class="n">record</span><span class="p">:</span> <span class="n">SQSRecord</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="n">payload</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">record</span><span class="p">.</span><span class="n">body</span><span class="p">)</span>
    <span class="k">return</span> <span class="nf">process_order_event</span><span class="p">(</span><span class="n">data</span><span class="o">=</span><span class="n">payload</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">lambda_handler</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="n">context</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">PartialItemFailureResponse</span><span class="p">:</span>
    <span class="k">return</span> <span class="nf">process_partial_response</span><span class="p">(</span>
        <span class="n">event</span><span class="o">=</span><span class="n">event</span><span class="p">,</span>
        <span class="n">record_handler</span><span class="o">=</span><span class="n">record_handler</span><span class="p">,</span>
        <span class="n">processor</span><span class="o">=</span><span class="n">processor</span><span class="p">,</span>
        <span class="n">context</span><span class="o">=</span><span class="n">context</span><span class="p">,</span>
    <span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Best practices I apply with SQS + idempotency
</h3>

<ul>
<li>
<strong>Use partial batch response</strong> to avoid retrying already-successful records.</li>
<li>
<strong>Set visibility timeout</strong> longer than worst-case processing time (or heartbeat/extend strategy).</li>
<li>
<strong>Keep the idempotency key in the message payload</strong>, not delivery metadata.</li>
<li>
<strong>Use a dedupe window long enough</strong> to cover retries, DLQ redrive, and operational replay.</li>
</ul>




<h2>
  
  
  Handling retries from EventBridge targets
</h2>

<p>EventBridge makes event-driven architecture clean, but duplicate-safe consumers are still my responsibility.</p>

<h3>
  
  
  EventBridge-specific considerations
</h3>

<ul>
<li>EventBridge may retry target delivery.</li>
<li>Archive/replay can intentionally re-send events.</li>
<li>Different publishers may emit semantically duplicate events unless the contract is strict.</li>
</ul>

<h3>
  
  
  Key strategy for EventBridge consumers
</h3>

<p>If the event has a stable <code>id</code> or domain event ID, I use it. If not, I derive one from:</p>

<ul>
<li><code>detail-type</code></li>
<li>source/domain identifier</li>
<li>business entity ID</li>
<li>action/version</li>
</ul>

<p>Example:</p>

<ul>
<li><code>eventbridge:{source}:{detailType}:{detail.orderId}:{detail.version}</code></li>
</ul>




<h2>
  
  
  EventBridge consumer example (Python Lambda + Powertools idempotency)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Any</span><span class="p">,</span> <span class="n">Dict</span>

<span class="kn">from</span> <span class="n">aws_lambda_powertools</span> <span class="kn">import</span> <span class="n">Logger</span>
<span class="kn">from</span> <span class="n">aws_lambda_powertools.utilities.idempotency</span> <span class="kn">import</span> <span class="p">(</span>
    <span class="n">DynamoDBPersistenceLayer</span><span class="p">,</span>
    <span class="n">IdempotencyConfig</span><span class="p">,</span>
    <span class="n">idempotent</span><span class="p">,</span>
<span class="p">)</span>

<span class="n">logger</span> <span class="o">=</span> <span class="nc">Logger</span><span class="p">(</span><span class="n">service</span><span class="o">=</span><span class="sh">"</span><span class="s">eventbridge-consumer</span><span class="sh">"</span><span class="p">)</span>

<span class="n">TABLE_NAME</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="sh">"</span><span class="s">IDEMPOTENCY_TABLE_NAME</span><span class="sh">"</span><span class="p">]</span>
<span class="n">persistence_layer</span> <span class="o">=</span> <span class="nc">DynamoDBPersistenceLayer</span><span class="p">(</span><span class="n">table_name</span><span class="o">=</span><span class="n">TABLE_NAME</span><span class="p">)</span>

<span class="n">config</span> <span class="o">=</span> <span class="nc">IdempotencyConfig</span><span class="p">(</span>
    <span class="c1"># Prefer a producer-defined unique event ID if available in detail
</span>    <span class="n">event_key_jmespath</span><span class="o">=</span><span class="sh">"</span><span class="s">detail.eventId || id</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">payload_validation_jmespath</span><span class="o">=</span><span class="sh">"</span><span class="s">[source, detail-type, detail.orderId, detail.version]</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">expires_after_seconds</span><span class="o">=</span><span class="mi">7</span> <span class="o">*</span> <span class="mi">24</span> <span class="o">*</span> <span class="mi">60</span> <span class="o">*</span> <span class="mi">60</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@idempotent</span><span class="p">(</span><span class="n">config</span><span class="o">=</span><span class="n">config</span><span class="p">,</span> <span class="n">persistence_store</span><span class="o">=</span><span class="n">persistence_layer</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">handle_event</span><span class="p">(</span><span class="n">event</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">],</span> <span class="n">context</span><span class="p">:</span> <span class="n">Any</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="n">detail</span> <span class="o">=</span> <span class="n">event</span><span class="p">[</span><span class="sh">"</span><span class="s">detail</span><span class="sh">"</span><span class="p">]</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span>
        <span class="sh">"</span><span class="s">Handling EventBridge event</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">extra</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">eventId</span><span class="sh">"</span><span class="p">:</span> <span class="n">detail</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">eventId</span><span class="sh">"</span><span class="p">,</span> <span class="n">event</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">)),</span> <span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">:</span> <span class="n">detail</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">)}</span>
    <span class="p">)</span>

    <span class="c1"># Execute your domain logic here
</span>    <span class="c1"># Example: update read model, trigger notification, call downstream API, etc.
</span>    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">handled</span><span class="sh">"</span><span class="p">:</span> <span class="bp">True</span><span class="p">,</span> <span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">:</span> <span class="n">detail</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">orderId</span><span class="sh">"</span><span class="p">)}</span>

<span class="k">def</span> <span class="nf">lambda_handler</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="n">context</span><span class="p">):</span>
    <span class="k">return</span> <span class="nf">handle_event</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="n">context</span><span class="p">)</span>
</code></pre>

</div>






<h2>
  
  
  When I implement idempotency manually (without Powertools)
</h2>

<p>Powertools is my default, but sometimes I implement manually when:</p>

<ul>
<li>I need a custom record schema shared across services/languages</li>
<li>I need custom conflict behavior beyond the library defaults</li>
<li>I am in a language/runtime where I am standardizing a platform abstraction</li>
<li>I want explicit control over lock and result update semantics</li>
</ul>

<p>The key principle stays the same: <strong>use DynamoDB conditional writes</strong>.</p>




<h2>
  
  
  Manual DynamoDB idempotency pattern (Python + boto3)
</h2>

<p>The pattern below shows the core idea:</p>

<ol>
<li>Try to write <code>IN_PROGRESS</code> with <code>attribute_not_exists(id)</code>
</li>
<li>If successful, execute business logic</li>
<li>Update item to <code>COMPLETED</code> and store result</li>
<li>If conditional check fails, inspect existing item and decide
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">time</span>
<span class="kn">import</span> <span class="n">hashlib</span>
<span class="kn">import</span> <span class="n">boto3</span>
<span class="kn">from</span> <span class="n">botocore.exceptions</span> <span class="kn">import</span> <span class="n">ClientError</span>

<span class="n">dynamodb</span> <span class="o">=</span> <span class="n">boto3</span><span class="p">.</span><span class="nf">client</span><span class="p">(</span><span class="sh">"</span><span class="s">dynamodb</span><span class="sh">"</span><span class="p">)</span>
<span class="n">TABLE_NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">IdempotencyTable</span><span class="sh">"</span>

<span class="k">def</span> <span class="nf">sha256_text</span><span class="p">(</span><span class="n">value</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="k">return</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="n">value</span><span class="p">.</span><span class="nf">encode</span><span class="p">(</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">)).</span><span class="nf">hexdigest</span><span class="p">()</span>

<span class="k">def</span> <span class="nf">acquire_lock</span><span class="p">(</span><span class="n">idempotency_key</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">payload_hash</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">ttl_seconds</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">86400</span><span class="p">,</span> <span class="n">lock_seconds</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">120</span><span class="p">):</span>
    <span class="n">now</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">())</span>
    <span class="n">item</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">:</span> <span class="n">idempotency_key</span><span class="p">},</span>
        <span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">IN_PROGRESS</span><span class="sh">"</span><span class="p">},</span>
        <span class="sh">"</span><span class="s">payloadHash</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">:</span> <span class="n">payload_hash</span><span class="p">},</span>
        <span class="sh">"</span><span class="s">expiration</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">N</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">now</span> <span class="o">+</span> <span class="n">ttl_seconds</span><span class="p">)},</span>
        <span class="sh">"</span><span class="s">inProgressExpiryTimestamp</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">N</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">now</span> <span class="o">+</span> <span class="n">lock_seconds</span><span class="p">)},</span>
        <span class="sh">"</span><span class="s">createdAt</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">N</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">now</span><span class="p">)},</span>
        <span class="sh">"</span><span class="s">updatedAt</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">N</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">now</span><span class="p">)},</span>
    <span class="p">}</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">dynamodb</span><span class="p">.</span><span class="nf">put_item</span><span class="p">(</span>
            <span class="n">TableName</span><span class="o">=</span><span class="n">TABLE_NAME</span><span class="p">,</span>
            <span class="n">Item</span><span class="o">=</span><span class="n">item</span><span class="p">,</span>
            <span class="n">ConditionExpression</span><span class="o">=</span><span class="sh">"</span><span class="s">attribute_not_exists(id)</span><span class="sh">"</span>
        <span class="p">)</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">acquired</span><span class="sh">"</span><span class="p">:</span> <span class="bp">True</span><span class="p">}</span>
    <span class="k">except</span> <span class="n">ClientError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">e</span><span class="p">.</span><span class="n">response</span><span class="p">[</span><span class="sh">"</span><span class="s">Error</span><span class="sh">"</span><span class="p">][</span><span class="sh">"</span><span class="s">Code</span><span class="sh">"</span><span class="p">]</span> <span class="o">!=</span> <span class="sh">"</span><span class="s">ConditionalCheckFailedException</span><span class="sh">"</span><span class="p">:</span>
            <span class="k">raise</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">acquired</span><span class="sh">"</span><span class="p">:</span> <span class="bp">False</span><span class="p">}</span>

<span class="k">def</span> <span class="nf">get_record</span><span class="p">(</span><span class="n">idempotency_key</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="n">resp</span> <span class="o">=</span> <span class="n">dynamodb</span><span class="p">.</span><span class="nf">get_item</span><span class="p">(</span>
        <span class="n">TableName</span><span class="o">=</span><span class="n">TABLE_NAME</span><span class="p">,</span>
        <span class="n">Key</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">:</span> <span class="n">idempotency_key</span><span class="p">}},</span>
        <span class="n">ConsistentRead</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
    <span class="p">)</span>
    <span class="k">return</span> <span class="n">resp</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">Item</span><span class="sh">"</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">mark_completed</span><span class="p">(</span><span class="n">idempotency_key</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">response_obj</span><span class="p">:</span> <span class="nb">dict</span><span class="p">):</span>
    <span class="n">now</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">())</span>
    <span class="n">dynamodb</span><span class="p">.</span><span class="nf">update_item</span><span class="p">(</span>
        <span class="n">TableName</span><span class="o">=</span><span class="n">TABLE_NAME</span><span class="p">,</span>
        <span class="n">Key</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">id</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">:</span> <span class="n">idempotency_key</span><span class="p">}},</span>
        <span class="n">UpdateExpression</span><span class="o">=</span><span class="sh">"</span><span class="s">SET #s = :completed, responseData = :resp, updatedAt = :now</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">ExpressionAttributeNames</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">#s</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">},</span>
        <span class="n">ExpressionAttributeValues</span><span class="o">=</span><span class="p">{</span>
            <span class="sh">"</span><span class="s">:completed</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">COMPLETED</span><span class="sh">"</span><span class="p">},</span>
            <span class="sh">"</span><span class="s">:resp</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">:</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">response_obj</span><span class="p">)},</span>
            <span class="sh">"</span><span class="s">:now</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">N</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">now</span><span class="p">)},</span>
        <span class="p">},</span>
    <span class="p">)</span>

<span class="k">def</span> <span class="nf">handler</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="n">context</span><span class="p">):</span>
    <span class="n">body</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">event</span><span class="p">[</span><span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">])</span>
    <span class="n">idem_key</span> <span class="o">=</span> <span class="n">event</span><span class="p">[</span><span class="sh">"</span><span class="s">headers</span><span class="sh">"</span><span class="p">][</span><span class="sh">"</span><span class="s">idempotency-key</span><span class="sh">"</span><span class="p">]</span>
    <span class="n">payload_hash</span> <span class="o">=</span> <span class="nf">sha256_text</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">({</span>
        <span class="sh">"</span><span class="s">customerId</span><span class="sh">"</span><span class="p">:</span> <span class="n">body</span><span class="p">[</span><span class="sh">"</span><span class="s">customerId</span><span class="sh">"</span><span class="p">],</span>
        <span class="sh">"</span><span class="s">amount</span><span class="sh">"</span><span class="p">:</span> <span class="n">body</span><span class="p">[</span><span class="sh">"</span><span class="s">amount</span><span class="sh">"</span><span class="p">],</span>
        <span class="sh">"</span><span class="s">currency</span><span class="sh">"</span><span class="p">:</span> <span class="n">body</span><span class="p">[</span><span class="sh">"</span><span class="s">currency</span><span class="sh">"</span><span class="p">],</span>
    <span class="p">},</span> <span class="n">sort_keys</span><span class="o">=</span><span class="bp">True</span><span class="p">))</span>

    <span class="n">lock</span> <span class="o">=</span> <span class="nf">acquire_lock</span><span class="p">(</span><span class="n">idem_key</span><span class="p">,</span> <span class="n">payload_hash</span><span class="p">)</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">lock</span><span class="p">[</span><span class="sh">"</span><span class="s">acquired</span><span class="sh">"</span><span class="p">]:</span>
        <span class="n">existing</span> <span class="o">=</span> <span class="nf">get_record</span><span class="p">(</span><span class="n">idem_key</span><span class="p">)</span>
        <span class="k">if</span> <span class="ow">not</span> <span class="n">existing</span><span class="p">:</span>
            <span class="c1"># Rare race / eventual state issue; safe to retry
</span>            <span class="k">raise</span> <span class="nc">Exception</span><span class="p">(</span><span class="sh">"</span><span class="s">Retry request</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">existing</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">payloadHash</span><span class="sh">"</span><span class="p">,</span> <span class="p">{}).</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">)</span> <span class="o">!=</span> <span class="n">payload_hash</span><span class="p">:</span>
            <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">statusCode</span><span class="sh">"</span><span class="p">:</span> <span class="mi">409</span><span class="p">,</span> <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">({</span><span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Idempotency key payload mismatch</span><span class="sh">"</span><span class="p">})}</span>
        <span class="n">status</span> <span class="o">=</span> <span class="n">existing</span><span class="p">[</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">][</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">]</span>
        <span class="k">if</span> <span class="n">status</span> <span class="o">==</span> <span class="sh">"</span><span class="s">COMPLETED</span><span class="sh">"</span><span class="p">:</span>
            <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">statusCode</span><span class="sh">"</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span> <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">existing</span><span class="p">[</span><span class="sh">"</span><span class="s">responseData</span><span class="sh">"</span><span class="p">][</span><span class="sh">"</span><span class="s">S</span><span class="sh">"</span><span class="p">]}</span>
        <span class="k">if</span> <span class="n">status</span> <span class="o">==</span> <span class="sh">"</span><span class="s">IN_PROGRESS</span><span class="sh">"</span><span class="p">:</span>
            <span class="c1"># For API workflows you might return 409 or 425-style retry signal (implementation-specific)
</span>            <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">statusCode</span><span class="sh">"</span><span class="p">:</span> <span class="mi">409</span><span class="p">,</span> <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">({</span><span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Request is already in progress</span><span class="sh">"</span><span class="p">})}</span>

    <span class="c1"># Execute business logic after lock acquired
</span>    <span class="n">result</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">paymentId</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pay_123</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">AUTHORIZED</span><span class="sh">"</span><span class="p">}</span>
    <span class="nf">mark_completed</span><span class="p">(</span><span class="n">idem_key</span><span class="p">,</span> <span class="n">result</span><span class="p">)</span>
    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">statusCode</span><span class="sh">"</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span> <span class="sh">"</span><span class="s">body</span><span class="sh">"</span><span class="p">:</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">result</span><span class="p">)}</span>
</code></pre>

</div>



<h3>
  
  
  Manual pattern caveats
</h3>

<p>If I go manual, I also need to think about:</p>

<ul>
<li>expired <code>IN_PROGRESS</code> lock recovery</li>
<li>exception handling and safe cleanup</li>
<li>serialization of cached responses</li>
<li>metrics for duplicate hits vs fresh requests</li>
<li>consistent behavior across all event sources</li>
</ul>

<p>That is exactly why Powertools is usually the better default.</p>




<h2>
  
  
  End-to-end implementation discussion (how I wire this in production)
</h2>

<p>This is the part I care about most in architecture reviews: not just the code, but where idempotency sits in the overall system.</p>

<h3>
  
  
  1) Idempotency belongs close to the handler boundary
</h3>

<p>I usually apply idempotency at the Lambda entry point (or record handler for batches), before business side effects occur.</p>

<p>Why:</p>

<ul>
<li>it prevents duplicate external calls</li>
<li>it keeps the protection broad</li>
<li>it makes retries safe by default</li>
</ul>

<h3>
  
  
  2) I still design downstream writes carefully
</h3>

<p>Idempotency at the Lambda layer is great, but if the function can partially succeed before crashing, I also check downstream safety:</p>

<ul>
<li>unique constraints in relational DBs</li>
<li>conditional writes in DynamoDB</li>
<li>provider-side idempotency support for payment APIs or webhooks</li>
</ul>

<p>Think in layers, not in a single magic switch.</p>

<h3>
  
  
  3) I define duplicate behavior per source
</h3>

<p>The response to a duplicate is not always the same.</p>

<ul>
<li>
<strong>API Gateway</strong>: return cached success response (best UX)</li>
<li>
<strong>SQS</strong>: ack success for already-processed message, avoid poison-loop</li>
<li>
<strong>EventBridge</strong>: safely no-op or return success after dedupe</li>
</ul>

<h3>
  
  
  4) I separate “idempotency key” and “correlation ID”
</h3>

<p>They are related but not identical.</p>

<ul>
<li>
<strong>Correlation ID</strong> -&gt; tracing/observability</li>
<li>
<strong>Idempotency key</strong> -&gt; duplicate suppression for a specific operation</li>
</ul>

<p>Sometimes they can be the same, but I do not assume that.</p>




<h2>
  
  
  Handling edge cases and failure modes
</h2>

<h3>
  
  
  Edge case 1: Same key, different payload
</h3>

<p>This should be treated as a contract violation.</p>

<p><strong>Best practice:</strong> return <code>409 Conflict</code> (or equivalent domain error) and log it loudly.</p>

<p>Why I do this:</p>

<ul>
<li>protects clients from accidental misuse</li>
<li>prevents serving incorrect cached results</li>
<li>surfaces integration bugs early</li>
</ul>




<h3>
  
  
  Edge case 2: Function times out after making a side effect
</h3>

<p>This is the classic ambiguous outcome problem.</p>

<p>Idempotency helps, but only if:</p>

<ul>
<li>the side effect can be detected or safely repeated, and/or</li>
<li>the result gets persisted before timeout</li>
</ul>

<p>Best practices:</p>

<ul>
<li>keep timeouts realistic</li>
<li>use downstream idempotency where available</li>
<li>break long operations into Step Functions when needed</li>
<li>persist progress checkpoints for multi-step work</li>
</ul>




<h3>
  
  
  Edge case 3: <code>IN_PROGRESS</code> records stuck after crashes
</h3>

<p>If a function crashes after acquiring the lock, duplicates may keep seeing <code>IN_PROGRESS</code>.</p>

<p>Best practices:</p>

<ul>
<li>use an <strong>in-progress lock expiry</strong>
</li>
<li>make retries back off</li>
<li>alert on sustained <code>IN_PROGRESS</code> accumulation</li>
<li>evaluate whether the operation is safe to re-attempt after lock expiry</li>
</ul>




<h3>
  
  
  Edge case 4: Replay and backfill
</h3>

<p>Operational replay is common and healthy. I design for it intentionally.</p>

<p>Best practices:</p>

<ul>
<li>choose a dedupe window that matches replay expectations</li>
<li>if replay should <em>re-run</em> effects, use a different idempotency namespace/version</li>
<li>document replay semantics for ops teams</li>
</ul>

<p>Example:</p>

<ul>
<li>normal key: <code>invoice-email:{invoiceId}</code>
</li>
<li>forced replay key namespace: <code>invoice-email:replay:{jobId}:{invoiceId}</code>
</li>
</ul>




<h2>
  
  
  Observability: what I monitor for idempotency
</h2>

<p>Idempotency is not just a code concern. I want to know how often it is being exercised and whether it is hiding a deeper issue.</p>

<h3>
  
  
  Metrics I like to emit
</h3>

<ul>
<li><code>IdempotencyFreshRequests</code></li>
<li><code>IdempotencyDuplicateHits</code></li>
<li><code>IdempotencyInProgressConflicts</code></li>
<li>
<code>IdempotencyValidationConflicts</code> (same key, different payload)</li>
<li><code>IdempotencyStoreErrors</code></li>
<li>handler latency split by fresh vs duplicate</li>
</ul>

<h3>
  
  
  Logs I always include
</h3>

<ul>
<li>idempotency key (or redacted/hash if sensitive)</li>
<li>source (<code>api</code>, <code>sqs</code>, <code>eventbridge</code>)</li>
<li>dedupe outcome (<code>fresh</code>, <code>duplicate_completed</code>, <code>duplicate_in_progress</code>, <code>validation_mismatch</code>)</li>
<li>business identifier (<code>orderId</code>, <code>paymentId</code>, etc.)</li>
</ul>

<p>This makes incident triage much faster.</p>




<h2>
  
  
  Practical best practices checklist (the part I use in reviews)
</h2>

<h3>
  
  
  Key selection
</h3>

<ul>
<li>[ ] Key represents a <strong>business operation</strong>, not transport metadata</li>
<li>[ ] Key is stable across retries/replays</li>
<li>[ ] Key cardinality is high enough to avoid false collisions</li>
<li>[ ] Payload mismatch with same key is detected</li>
</ul>

<h3>
  
  
  Dedupe window
</h3>

<ul>
<li>[ ] TTL matches retry + redrive + replay realities</li>
<li>[ ] Expiry is checked in code (not only by DynamoDB TTL cleanup)</li>
<li>[ ] Window is documented in API/event contract</li>
</ul>

<h3>
  
  
  Store design
</h3>

<ul>
<li>[ ] DynamoDB conditional write used for first writer wins</li>
<li>[ ] <code>IN_PROGRESS</code> and <code>COMPLETED</code> states are handled explicitly</li>
<li>[ ] Cached response/ack strategy is defined</li>
<li>[ ] Encryption, backups/PITR, and least privilege are configured</li>
</ul>

<h3>
  
  
  Source-specific behavior
</h3>

<ul>
<li>[ ] API duplicates return deterministic response</li>
<li>[ ] SQS uses partial batch response</li>
<li>[ ] EventBridge consumer supports replay safely</li>
<li>[ ] Redrive/replay semantics are documented for ops</li>
</ul>

<h3>
  
  
  Operations
</h3>

<ul>
<li>[ ] Metrics and alarms exist for duplicate spikes and store failures</li>
<li>[ ] Logs include dedupe outcomes and business IDs</li>
<li>[ ] Runbooks cover stale <code>IN_PROGRESS</code> records and replay scenarios</li>
</ul>




<h2>
  
  
  Common mistakes I see (and how I avoid them)
</h2>

<h3>
  
  
  Mistake 1: Using Lambda <code>requestId</code> as the idempotency key
</h3>

<p>That only identifies the invocation, not the logical request.</p>

<p><strong>Fix:</strong> use business operation identity or client-provided idempotency key.</p>




<h3>
  
  
  Mistake 2: Assuming FIFO queue means I do not need idempotency
</h3>

<p>FIFO helps with ordering and deduplication windows, but it does not replace end-to-end idempotency for all side effects and replay paths.</p>

<p><strong>Fix:</strong> still make the consumer idempotent.</p>




<h3>
  
  
  Mistake 3: Dedupe only at the API layer
</h3>

<p>Then an async worker downstream duplicates the side effect anyway.</p>

<p><strong>Fix:</strong> apply idempotency where side effects happen, especially in SQS/EventBridge consumers.</p>




<h3>
  
  
  Mistake 4: No payload validation on key reuse
</h3>

<p>This can return the wrong cached response and create hidden data integrity issues.</p>

<p><strong>Fix:</strong> validate a stable subset of the payload with the idempotency key.</p>




<h3>
  
  
  Mistake 5: Too-short TTL
</h3>

<p>The key expires before retries/redrives finish, so duplicates sneak through.</p>

<p><strong>Fix:</strong> pick TTL based on actual operational timelines, not guesswork.</p>




<h2>
  
  
  Final thoughts
</h2>

<p>If I had to summarize production-grade idempotency architecture in one line, it would be this:</p>

<p><strong>Design for duplicate delivery as normal behavior, then make your Lambda handlers safe, deterministic, and observable.</strong></p>

<p>AWS gives us excellent building blocks for this:</p>

<ul>
<li>Lambda</li>
<li>SQS / EventBridge / API Gateway</li>
<li>DynamoDB conditional writes</li>
<li>AWS Lambda Powertools idempotency utility</li>
</ul>

<p>When I combine them intentionally, retries stop being scary and start being a reliability feature instead of a data integrity risk.</p>

<p>If you are building Lambda-driven systems that write to money, inventory, notifications, or customer state, idempotency is not optional. It is a core part of the architecture.</p>

<p>That placement helps readers understand the flow before diving into implementation details.</p>




<h2>
  
  
  References
</h2>

<ul>
<li>AWS Lambda Powertools (Python) documentation</li>
<li>AWS Lambda developer guide</li>
<li>Amazon SQS developer guide (Lambda event source mappings / retries / partial batch response)</li>
<li>Amazon EventBridge documentation (retries, targets, replay/archive)</li>
<li>Amazon DynamoDB documentation (conditional writes, TTL, PITR)</li>
</ul>

