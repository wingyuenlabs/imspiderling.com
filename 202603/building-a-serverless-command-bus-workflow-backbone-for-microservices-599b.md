---
Title: Building a Serverless Command Bus / Workflow Backbone for Microservices
Description: 
Author: Renaldi
Date: 2026-03-15T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>When I design microservice platforms on AWS, one of the recurring problems I see is not “how do I trigger a Lambda,” but rather “how do I keep a growing set of services coordinated, observable, and governed without turning everything into tight coupling?”</p>

<p>That is where a <strong>serverless command bus plus workflow backbone</strong> becomes a powerful pattern.</p>

<p>In this post, I will walk through how I build this in practice using <strong>Amazon EventBridge</strong> and <strong>AWS Step Functions</strong>, and I will cover:</p>

<ul>
<li>Commands vs events</li>
<li>EventBridge vs direct invoke</li>
<li>Step Functions for orchestrated flows</li>
<li>Auditability and tracing</li>
<li>Contract evolution</li>
<li>Governance and domain boundaries</li>
</ul>

<p>I will also include an <strong>end-to-end implementation walkthrough</strong>, code samples, and architecture guidance so this is not just conceptual.</p>




<h2>
  
  
  Why this pattern is worth building
</h2>

<p>As systems grow, teams usually end up with some mixture of:</p>

<ul>
<li>synchronous service-to-service calls</li>
<li>ad hoc retries</li>
<li>duplicated orchestration logic</li>
<li>inconsistent logging/correlation IDs</li>
<li>unclear ownership of contracts</li>
<li>event naming drift</li>
<li>“mystery failures” across multiple services</li>
</ul>

<p>A command bus/workflow backbone gives you a more intentional architecture:</p>

<ul>
<li>
<strong>Commands</strong> are routed to the right handler or orchestrator</li>
<li>
<strong>Workflows</strong> coordinate multi-step business actions</li>
<li>
<strong>Events</strong> are published as business facts for downstream consumers</li>
<li>
<strong>Tracing/audit</strong> become standard instead of optional</li>
<li>
<strong>Contract governance</strong> is enforced at boundaries rather than after incidents</li>
</ul>

<p>This is especially effective for domains like:</p>

<ul>
<li>order fulfillment</li>
<li>onboarding/KYC</li>
<li>claims processing</li>
<li>content moderation pipelines</li>
<li>payment settlement</li>
<li>inventory reservation</li>
<li>fulfillment and notification fan-out</li>
</ul>




<h2>
  
  
  What I mean by “Command Bus” in a serverless context
</h2>

<p>In classic application architecture, a command bus is often an in-process dispatching mechanism. In distributed systems, I use the term a bit differently:</p>

<ul>
<li>A <strong>command</strong> represents an intent to perform a business action</li>
<li>The <strong>command bus</strong> is the routing layer that accepts commands and sends them to the correct handler/orchestrator</li>
<li>A <strong>workflow backbone</strong> is the orchestration mechanism that executes multi-step business flows and emits events</li>
</ul>

<p>On AWS, a practical implementation is:</p>

<ul>
<li>
<strong>API Gateway + Lambda</strong> for ingress</li>
<li>
<strong>EventBridge</strong> as the command routing fabric</li>
<li>
<strong>Step Functions</strong> as the workflow engine</li>
<li>
<strong>EventBridge (domain event bus)</strong> for business events</li>
<li>
<strong>DynamoDB / S3 / CloudWatch / X-Ray / OTEL</strong> for auditability and observability</li>
</ul>




<h2>
  
  
  Commands vs Events (and why this distinction matters)
</h2>

<p>This is the first thing I standardize with teams, because it affects naming, routing, retries, and ownership.</p>

<h3>
  
  
  Commands
</h3>

<p>A command is an instruction or request to do something.</p>

<p>Examples:</p>

<ul>
<li><code>CreateOrder</code></li>
<li><code>ApproveLoanApplication</code></li>
<li><code>ReserveInventory</code></li>
<li><code>InitiateRefund</code></li>
</ul>

<p>Commands are typically:</p>

<ul>
<li><strong>intent-driven</strong></li>
<li>often addressed to <strong>one owning domain/service</strong>
</li>
<li>can be <strong>accepted/rejected</strong>
</li>
<li>may need <strong>validation and idempotency</strong>
</li>
<li>often require <strong>stronger governance</strong> at the boundary</li>
</ul>

<p>A command implies someone is asking for work to be done.</p>

<h3>
  
  
  Events
</h3>

<p>An event is a fact that something already happened.</p>

<p>Examples:</p>

<ul>
<li><code>OrderCreated</code></li>
<li><code>InventoryReserved</code></li>
<li><code>PaymentAuthorized</code></li>
<li><code>RefundInitiated</code></li>
</ul>

<p>Events are typically:</p>

<ul>
<li><strong>past-tense facts</strong></li>
<li>can have <strong>many subscribers</strong>
</li>
<li>should be treated as <strong>immutable</strong>
</li>
<li>support <strong>decoupled reactions</strong>
</li>
<li>are excellent for <strong>read models, analytics, notifications, integrations</strong>
</li>
</ul>

<p>An event should not be phrased as an instruction.</p>

<h3>
  
  
  A rule I use with teams
</h3>

<p>If the payload is saying, “please do this,” it is a <strong>command</strong>.<br><br>
If the payload is saying, “this happened,” it is an <strong>event</strong>.</p>

<p>That sounds simple, but it prevents a lot of design drift.</p>


<h2>
  
  
  EventBridge vs direct invoke (when to use each)
</h2>

<p>I do not recommend forcing everything through a bus. Some interactions should remain direct and synchronous.</p>
<h3>
  
  
  Use EventBridge when you want
</h3>

<ul>
<li>loose coupling between producers and consumers</li>
<li>routing based on message metadata</li>
<li>asynchronous execution</li>
<li>fan-out to multiple consumers</li>
<li>cross-account integration patterns</li>
<li>central governance on message flow</li>
<li>easy addition of new downstream consumers later</li>
</ul>
<h3>
  
  
  Use direct invoke (HTTP/gRPC/Lambda invoke) when you need
</h3>

<ul>
<li>immediate response to the caller</li>
<li>low latency request/response semantics</li>
<li>tight transactional dependencies (within reason)</li>
<li>user-facing UX that cannot tolerate async polling/callbacks</li>
<li>simple service composition where orchestration overhead is not justified</li>
</ul>
<h3>
  
  
  My practical guidance
</h3>

<p>I usually use a <strong>hybrid</strong> model:</p>

<ul>
<li>
<strong>Direct invoke</strong> for request/response reads and tightly-coupled synchronous operations</li>
<li>
<strong>Command bus + Step Functions</strong> for business workflows and long-running actions</li>
<li>
<strong>Event bus</strong> for domain events and downstream side effects</li>
</ul>

<p>That gives you the best of both worlds without ideological overreach.</p>


<h2>
  
  
  Reference Architecture
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffxumnxndus9urwvskf9c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffxumnxndus9urwvskf9c.png" alt=" " width="800" height="293"></a></p>
<h3>
  
  
  Architecture overview
</h3>

<p>At a high level, the flow is:</p>

<ol>
<li>Client sends a command (e.g., <code>CreateOrder</code>)</li>
<li>API layer validates, stamps correlation metadata, applies idempotency</li>
<li>Command is published to a <strong>Command Bus</strong> (EventBridge)</li>
<li>Rule routes command to a <strong>Step Functions</strong> state machine</li>
<li>Workflow invokes domain steps (fraud, payment, inventory, order write)</li>
<li>Workflow emits <strong>domain events</strong> (e.g., <code>OrderCreated</code> or <code>OrderRejected</code>)</li>
<li>Event consumers react independently (notifications, analytics, CRM, projections)</li>
<li>Audit and tracing data is captured throughout</li>
</ol>

<p>This separates:</p>

<ul>
<li><strong>request intent ingestion</strong></li>
<li><strong>workflow orchestration</strong></li>
<li><strong>event publication</strong></li>
<li><strong>consumer reactions</strong></li>
<li><strong>governance/observability</strong></li>
</ul>


<h2>
  
  
  End-to-End Walkthrough: <code>CreateOrder</code> command
</h2>

<p>To make this concrete, I will use an order workflow.</p>
<h3>
  
  
  Business scenario
</h3>

<p>A client submits an order. The system needs to:</p>

<ol>
<li>Validate the request and check idempotency</li>
<li>Run a fraud check</li>
<li>Authorize payment</li>
<li>Reserve inventory</li>
<li>Persist the order</li>
<li>Publish the resulting business event(s)</li>
<li>Notify downstream systems</li>
</ol>
<h3>
  
  
  Step 1: Client submits a command
</h3>

<p>The client calls an API endpoint such as:</p>

<ul>
<li><code>POST /orders</code></li>
</ul>

<p>Internally, I treat this as a <code>CreateOrderCommand</code>.</p>

<p>The API layer does not directly orchestrate business logic. Its job is to:</p>

<ul>
<li>authenticate/authorize</li>
<li>validate payload shape</li>
<li>assign <code>commandId</code> and <code>correlationId</code>
</li>
<li>apply idempotency safeguards</li>
<li>publish the command to the bus</li>
</ul>
<h3>
  
  
  Step 2: Command is routed on the command bus
</h3>

<p>The command lands on an EventBridge bus (logical “command bus”).</p>

<p>A rule matches:</p>

<ul>
<li>source</li>
<li>detail-type</li>
<li>possibly domain/version metadata</li>
</ul>

<p>That rule starts a Step Functions state machine for <code>CreateOrder</code>.</p>

<p>This gives me routing flexibility without wiring the producer directly to the orchestrator.</p>
<h3>
  
  
  Step 3: Step Functions orchestrates the business flow
</h3>

<p>The state machine runs each step with retries and error handling.</p>

<p>Typical flow:</p>

<ul>
<li><code>FraudCheck</code></li>
<li><code>AuthorizePayment</code></li>
<li><code>ReserveInventory</code></li>
<li><code>PersistOrder</code></li>
<li><code>PublishOrderCreatedEvent</code></li>
</ul>

<p>If any step fails:</p>

<ul>
<li>compensate if needed (e.g., release reservation / void payment)</li>
<li>emit failure event (e.g., <code>OrderRejected</code>, <code>PaymentFailed</code>)</li>
<li>write audit trail</li>
</ul>
<h3>
  
  
  Step 4: Domain events are published
</h3>

<p>Once the workflow succeeds or fails, the workflow emits a domain event to an EventBridge <strong>event bus</strong> (logical “domain event bus”).</p>

<p>Examples:</p>

<ul>
<li><code>OrderCreated</code></li>
<li><code>OrderRejected</code></li>
<li><code>PaymentFailed</code></li>
</ul>

<p>These events can fan out to multiple consumers without changing the workflow code.</p>
<h3>
  
  
  Step 5: Consumers react independently
</h3>

<p>Downstream services subscribe and act:</p>

<ul>
<li>Notifications send email/SMS/push</li>
<li>Analytics ingests event into a lake/warehouse</li>
<li>CRM updates lifecycle state</li>
<li>Projection service updates a query/read model</li>
</ul>

<p>None of these consumers need to be in the critical path of order creation.</p>


<h2>
  
  
  Implementation discussion (practical, not just theoretical)
</h2>

<p>Below is a concrete implementation shape I use and recommend.</p>


<h2>
  
  
  Message contract design (command and event envelopes)
</h2>

<p>I strongly recommend standardizing a message envelope early. It pays off in observability, replay, governance, and contract evolution.</p>
<h3>
  
  
  Command envelope example
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"meta"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"messageType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"commandName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CreateOrder"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"schemaVersion"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"commandId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"cmd_01HSYJQF3A6M0R6G0J7VY7A1J9"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"correlationId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"corr_2f1d3a07-3d89-4a62-9f6c-9f29f4e96f7e"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"causationId"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
    </span><span class="nl">"occurredAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T09:20:00Z"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"tenantId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tenant-001"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"initiator"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user-123"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"routing"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"domain"</span><span class="p">:</span><span class="w"> </span><span class="s2">"orders"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"target"</span><span class="p">:</span><span class="w"> </span><span class="s2">"orders.workflow"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"priority"</span><span class="p">:</span><span class="w"> </span><span class="s2">"normal"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"orderId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ORD-100045"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"customerId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CUST-9001"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"currency"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AUD"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"items"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"sku"</span><span class="p">:</span><span class="w"> </span><span class="s2">"SKU-123"</span><span class="p">,</span><span class="w"> </span><span class="nl">"qty"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w"> </span><span class="nl">"unitPrice"</span><span class="p">:</span><span class="w"> </span><span class="mf">49.95</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w"> </span><span class="nl">"sku"</span><span class="p">:</span><span class="w"> </span><span class="s2">"SKU-999"</span><span class="p">,</span><span class="w"> </span><span class="nl">"qty"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w"> </span><span class="nl">"unitPrice"</span><span class="p">:</span><span class="w"> </span><span class="mf">19.95</span><span class="w"> </span><span class="p">}</span><span class="w">
    </span><span class="p">],</span><span class="w">
    </span><span class="nl">"paymentMethodToken"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tok_abc123"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>

<h3>
  
  
  Event envelope example
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"meta"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"messageType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"event"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"eventName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"OrderCreated"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"schemaVersion"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.1"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"eventId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"evt_01HSYK2GQKCNZC3Q0Z6R3T9M7K"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"correlationId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"corr_2f1d3a07-3d89-4a62-9f6c-9f29f4e96f7e"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"causationId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"cmd_01HSYJQF3A6M0R6G0J7VY7A1J9"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"occurredAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T09:20:03Z"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"producer"</span><span class="p">:</span><span class="w"> </span><span class="s2">"orders.workflow"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"subject"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"domain"</span><span class="p">:</span><span class="w"> </span><span class="s2">"orders"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ORD-100045"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"orderId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ORD-100045"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CREATED"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"paymentStatus"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AUTHORIZED"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"inventoryStatus"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RESERVED"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"totalAmount"</span><span class="p">:</span><span class="w"> </span><span class="mf">119.85</span><span class="p">,</span><span class="w">
    </span><span class="nl">"currency"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AUD"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>

<h3>
  
  
  Why I standardize envelopes
</h3>

<p>This makes it much easier to implement:</p>

<ul>
<li>trace propagation</li>
<li>replay tooling</li>
<li>audit logging</li>
<li>schema validation</li>
<li>version migration</li>
<li>tenant-aware governance</li>
</ul>


<h2>
  
  
  API ingress (Lambda) for command intake
</h2>

<p>This example shows a command-ingress Lambda that validates input, applies idempotency, and publishes to EventBridge.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/handlers/create-order-command.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">APIGatewayProxyEventV2</span><span class="p">,</span> <span class="nx">APIGatewayProxyStructuredResultV2</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-lambda</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">EventBridgeClient</span><span class="p">,</span> <span class="nx">PutEventsCommand</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/client-eventbridge</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">DynamoDBClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/client-dynamodb</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">DynamoDBDocumentClient</span><span class="p">,</span> <span class="nx">PutCommand</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/lib-dynamodb</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">randomUUID</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">crypto</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">eb</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">EventBridgeClient</span><span class="p">({});</span>
<span class="kd">const</span> <span class="nx">ddb</span> <span class="o">=</span> <span class="nx">DynamoDBDocumentClient</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="k">new</span> <span class="nc">DynamoDBClient</span><span class="p">({}));</span>

<span class="kd">const</span> <span class="nx">COMMAND_BUS_NAME</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">COMMAND_BUS_NAME</span><span class="o">!</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">IDEMPOTENCY_TABLE</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">IDEMPOTENCY_TABLE</span><span class="o">!</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">COMMAND_SOURCE</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">com.example.orders.api</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span>
  <span class="nx">event</span><span class="p">:</span> <span class="nx">APIGatewayProxyEventV2</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">APIGatewayProxyStructuredResultV2</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">correlationId</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">headers</span><span class="p">[</span><span class="dl">"</span><span class="s2">x-correlation-id</span><span class="dl">"</span><span class="p">]</span> <span class="o">??</span> <span class="s2">`corr_</span><span class="p">${</span><span class="nf">randomUUID</span><span class="p">()}</span><span class="s2">`</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">idempotencyKey</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">headers</span><span class="p">[</span><span class="dl">"</span><span class="s2">idempotency-key</span><span class="dl">"</span><span class="p">]</span> <span class="o">??</span> <span class="s2">`idem_</span><span class="p">${</span><span class="nf">randomUUID</span><span class="p">()}</span><span class="s2">`</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">event</span><span class="p">.</span><span class="nx">body</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">statusCode</span><span class="p">:</span> <span class="mi">400</span><span class="p">,</span> <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Missing request body</span><span class="dl">"</span> <span class="p">})</span> <span class="p">};</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">body</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">body</span><span class="p">);</span>

  <span class="c1">// Minimal shape validation (use JSON Schema in production)</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">body</span><span class="p">.</span><span class="nx">customerId</span> <span class="o">||</span> <span class="o">!</span><span class="nb">Array</span><span class="p">.</span><span class="nf">isArray</span><span class="p">(</span><span class="nx">body</span><span class="p">.</span><span class="nx">items</span><span class="p">)</span> <span class="o">||</span> <span class="nx">body</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">statusCode</span><span class="p">:</span> <span class="mi">400</span><span class="p">,</span> <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Invalid order payload</span><span class="dl">"</span> <span class="p">})</span> <span class="p">};</span>
  <span class="p">}</span>

  <span class="c1">// Idempotency write (fail if key already exists)</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">ddb</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span>
      <span class="k">new</span> <span class="nc">PutCommand</span><span class="p">({</span>
        <span class="na">TableName</span><span class="p">:</span> <span class="nx">IDEMPOTENCY_TABLE</span><span class="p">,</span>
        <span class="na">Item</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">pk</span><span class="p">:</span> <span class="nx">idempotencyKey</span><span class="p">,</span>
          <span class="na">createdAt</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">(),</span>
          <span class="nx">correlationId</span>
        <span class="p">},</span>
        <span class="na">ConditionExpression</span><span class="p">:</span> <span class="dl">"</span><span class="s2">attribute_not_exists(pk)</span><span class="dl">"</span>
      <span class="p">})</span>
    <span class="p">);</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="na">err</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">err</span><span class="p">.</span><span class="nx">name</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">ConditionalCheckFailedException</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">return</span> <span class="p">{</span>
        <span class="na">statusCode</span><span class="p">:</span> <span class="mi">409</span><span class="p">,</span>
        <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Duplicate request</span><span class="dl">"</span><span class="p">,</span> <span class="nx">correlationId</span><span class="p">,</span> <span class="nx">idempotencyKey</span> <span class="p">})</span>
      <span class="p">};</span>
    <span class="p">}</span>
    <span class="k">throw</span> <span class="nx">err</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">commandId</span> <span class="o">=</span> <span class="s2">`cmd_</span><span class="p">${</span><span class="nf">randomUUID</span><span class="p">()}</span><span class="s2">`</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">now</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">();</span>

  <span class="kd">const</span> <span class="nx">commandEnvelope</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">meta</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">messageType</span><span class="p">:</span> <span class="dl">"</span><span class="s2">command</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">commandName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">CreateOrder</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">schemaVersion</span><span class="p">:</span> <span class="dl">"</span><span class="s2">1.0</span><span class="dl">"</span><span class="p">,</span>
      <span class="nx">commandId</span><span class="p">,</span>
      <span class="nx">correlationId</span><span class="p">,</span>
      <span class="na">causationId</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span>
      <span class="na">occurredAt</span><span class="p">:</span> <span class="nx">now</span><span class="p">,</span>
      <span class="na">tenantId</span><span class="p">:</span> <span class="dl">"</span><span class="s2">tenant-001</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">initiator</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user-123</span><span class="dl">"</span> <span class="p">}</span>
    <span class="p">},</span>
    <span class="na">routing</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">domain</span><span class="p">:</span> <span class="dl">"</span><span class="s2">orders</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">target</span><span class="p">:</span> <span class="dl">"</span><span class="s2">orders.workflow</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">priority</span><span class="p">:</span> <span class="dl">"</span><span class="s2">normal</span><span class="dl">"</span>
    <span class="p">},</span>
    <span class="na">data</span><span class="p">:</span> <span class="nx">body</span>
  <span class="p">};</span>

  <span class="k">await</span> <span class="nx">eb</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span>
    <span class="k">new</span> <span class="nc">PutEventsCommand</span><span class="p">({</span>
      <span class="na">Entries</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="na">EventBusName</span><span class="p">:</span> <span class="nx">COMMAND_BUS_NAME</span><span class="p">,</span>
          <span class="na">Source</span><span class="p">:</span> <span class="nx">COMMAND_SOURCE</span><span class="p">,</span>
          <span class="na">DetailType</span><span class="p">:</span> <span class="dl">"</span><span class="s2">CreateOrderCommand</span><span class="dl">"</span><span class="p">,</span>
          <span class="na">Time</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="nx">now</span><span class="p">),</span>
          <span class="na">Detail</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">commandEnvelope</span><span class="p">)</span>
        <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">})</span>
  <span class="p">);</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">statusCode</span><span class="p">:</span> <span class="mi">202</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
      <span class="dl">"</span><span class="s2">content-type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span><span class="p">,</span>
      <span class="dl">"</span><span class="s2">x-correlation-id</span><span class="dl">"</span><span class="p">:</span> <span class="nx">correlationId</span>
    <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
      <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Command accepted</span><span class="dl">"</span><span class="p">,</span>
      <span class="nx">commandId</span><span class="p">,</span>
      <span class="nx">correlationId</span>
    <span class="p">})</span>
  <span class="p">};</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  A note on API response semantics
</h3>

<p>I usually return <strong><code>202 Accepted</code></strong> for async workflows. That is clearer than pretending the business operation is already complete.</p>




<h2>
  
  
  EventBridge routing for the command bus
</h2>

<p>I use EventBridge rules to map command types to orchestrators or handlers.</p>

<h3>
  
  
  Routing rule concept
</h3>

<ul>
<li><code>source = com.example.orders.api</code></li>
<li><code>detail-type = CreateOrderCommand</code></li>
<li>target = Step Functions state machine</li>
</ul>

<p>This is a clean handoff between ingress and orchestration.</p>




<h2>
  
  
  Step Functions for orchestrated flows
</h2>

<p>This is the backbone of the pattern for multi-step business actions.</p>

<p>Step Functions gives you:</p>

<ul>
<li>retries/backoff</li>
<li>explicit error branches</li>
<li>timeouts</li>
<li>compensation flows</li>
<li>state tracking</li>
<li>execution history</li>
<li>integration with Lambda and AWS services</li>
</ul>

<h3>
  
  
  Example state machine (Amazon States Language)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Comment"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CreateOrder workflow"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"StartAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"FraudCheck"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"States"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"FraudCheck"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::lambda:invoke"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"FunctionName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"${FraudCheckFnArn}"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"Payload.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultSelector"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"fraud.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.Payload"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultPath"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.fraudResult"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Retry"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">{</span><span class="w">
          </span><span class="nl">"ErrorEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Lambda.ServiceException"</span><span class="p">,</span><span class="w"> </span><span class="s2">"Lambda.SdkClientException"</span><span class="p">,</span><span class="w"> </span><span class="s2">"States.TaskFailed"</span><span class="p">],</span><span class="w">
          </span><span class="nl">"IntervalSeconds"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
          </span><span class="nl">"BackoffRate"</span><span class="p">:</span><span class="w"> </span><span class="mf">2.0</span><span class="p">,</span><span class="w">
          </span><span class="nl">"MaxAttempts"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"FraudDecision"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"FraudDecision"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Choice"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Choices"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">{</span><span class="w">
          </span><span class="nl">"Variable"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.fraudResult.fraud.approved"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"BooleanEquals"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
          </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AuthorizePayment"</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="nl">"Default"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PublishOrderRejected"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"AuthorizePayment"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::lambda:invoke"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"FunctionName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"${AuthorizePaymentFnArn}"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"Payload.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultSelector"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"payment.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.Payload"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultPath"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.paymentResult"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Catch"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">{</span><span class="w">
          </span><span class="nl">"ErrorEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"States.ALL"</span><span class="p">],</span><span class="w">
          </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PublishPaymentFailed"</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ReserveInventory"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"ReserveInventory"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::lambda:invoke"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"FunctionName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"${ReserveInventoryFnArn}"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"Payload.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultSelector"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"inventory.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.Payload"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultPath"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.inventoryResult"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Catch"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="p">{</span><span class="w">
          </span><span class="nl">"ErrorEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"States.ALL"</span><span class="p">],</span><span class="w">
          </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PublishOrderRejected"</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PersistOrder"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"PersistOrder"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::lambda:invoke"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"FunctionName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"${PersistOrderFnArn}"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"Payload.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultSelector"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"order.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.Payload"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultPath"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.orderResult"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PublishOrderCreated"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"PublishOrderCreated"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::events:putEvents"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"Entries"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="nl">"EventBusName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"${DomainEventBusName}"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Source"</span><span class="p">:</span><span class="w"> </span><span class="s2">"com.example.orders.workflow"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"DetailType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"OrderCreated"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Detail.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"States.JsonToString($.orderResult.order.eventEnvelope)"</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"End"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"PublishPaymentFailed"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::events:putEvents"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"Entries"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="nl">"EventBusName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"${DomainEventBusName}"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Source"</span><span class="p">:</span><span class="w"> </span><span class="s2">"com.example.orders.workflow"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"DetailType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PaymentFailed"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Detail.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"States.JsonToString($.paymentResult.paymentFailureEvent)"</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"End"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"PublishOrderRejected"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::events:putEvents"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"Entries"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="nl">"EventBusName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"${DomainEventBusName}"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Source"</span><span class="p">:</span><span class="w"> </span><span class="s2">"com.example.orders.workflow"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"DetailType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"OrderRejected"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Detail.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"States.JsonToString($.fraudResult.fraud.rejectionEvent)"</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"End"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Why I like this approach
</h3>

<p>The orchestration is visible and reviewable. You can reason about:</p>

<ul>
<li>what happens on success</li>
<li>what retries are safe</li>
<li>how failures are surfaced</li>
<li>which events are emitted</li>
<li>which steps need compensation</li>
</ul>

<p>That is much harder when orchestration is hidden across multiple services.</p>




<h2>
  
  
  Domain task implementation example (Lambda)
</h2>

<p>Here is a simplified payment authorization step.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/handlers/authorize-payment.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Context</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-lambda</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">WorkflowInput</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">meta</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">correlationId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">commandId</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="p">};</span>
  <span class="nl">data</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">orderId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">customerId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">currency</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">items</span><span class="p">:</span> <span class="nb">Array</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">sku</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">qty</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">unitPrice</span><span class="p">:</span> <span class="kr">number</span> <span class="p">}</span><span class="o">&gt;</span><span class="p">;</span>
    <span class="nl">paymentMethodToken</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="p">};</span>
<span class="p">};</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">input</span><span class="p">:</span> <span class="nx">WorkflowInput</span><span class="p">,</span> <span class="nx">context</span><span class="p">:</span> <span class="nx">Context</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">correlationId</span> <span class="o">=</span> <span class="nx">input</span><span class="p">.</span><span class="nx">meta</span><span class="p">?.</span><span class="nx">correlationId</span> <span class="o">??</span> <span class="dl">"</span><span class="s2">unknown</span><span class="dl">"</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">totalAmount</span> <span class="o">=</span> <span class="nx">input</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">sum</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">sum</span> <span class="o">+</span> <span class="nx">i</span><span class="p">.</span><span class="nx">qty</span> <span class="o">*</span> <span class="nx">i</span><span class="p">.</span><span class="nx">unitPrice</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>

  <span class="c1">// Simulated payment auth</span>
  <span class="kd">const</span> <span class="nx">paymentAuthorized</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>

  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
    <span class="na">level</span><span class="p">:</span> <span class="dl">"</span><span class="s2">INFO</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Authorizing payment</span><span class="dl">"</span><span class="p">,</span>
    <span class="nx">correlationId</span><span class="p">,</span>
    <span class="na">awsRequestId</span><span class="p">:</span> <span class="nx">context</span><span class="p">.</span><span class="nx">awsRequestId</span><span class="p">,</span>
    <span class="na">orderId</span><span class="p">:</span> <span class="nx">input</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">orderId</span><span class="p">,</span>
    <span class="nx">totalAmount</span>
  <span class="p">}));</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">paymentAuthorized</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Payment authorization failed</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">authorized</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">authorizationId</span><span class="p">:</span> <span class="s2">`auth_</span><span class="p">${</span><span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()}</span><span class="s2">`</span><span class="p">,</span>
    <span class="na">amount</span><span class="p">:</span> <span class="nc">Number</span><span class="p">(</span><span class="nx">totalAmount</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)),</span>
    <span class="na">currency</span><span class="p">:</span> <span class="nx">input</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">currency</span>
  <span class="p">};</span>
<span class="p">};</span>
</code></pre>

</div>






<h2>
  
  
  CDK example (wiring the core pieces)
</h2>

<p>Below is a simplified AWS CDK (TypeScript) example to show how the pieces connect.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// lib/command-bus-stack.ts</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">cdk</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Construct</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">constructs</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">events</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-events</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">targets</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-events-targets</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">lambda</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-lambda</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">dynamodb</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-dynamodb</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">sfn</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-stepfunctions</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">CommandBusStack</span> <span class="kd">extends</span> <span class="nc">cdk</span><span class="p">.</span><span class="nx">Stack</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="nx">scope</span><span class="p">:</span> <span class="nx">Construct</span><span class="p">,</span> <span class="nx">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">props</span><span class="p">?:</span> <span class="nx">cdk</span><span class="p">.</span><span class="nx">StackProps</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">super</span><span class="p">(</span><span class="nx">scope</span><span class="p">,</span> <span class="nx">id</span><span class="p">,</span> <span class="nx">props</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">commandBus</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">events</span><span class="p">.</span><span class="nc">EventBus</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">CommandBus</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">eventBusName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">command-bus</span><span class="dl">"</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">domainEventBus</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">events</span><span class="p">.</span><span class="nc">EventBus</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">DomainEventBus</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">eventBusName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">domain-event-bus</span><span class="dl">"</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">idempotencyTable</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nc">Table</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">IdempotencyTable</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">partitionKey</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">pk</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">AttributeType</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="na">billingMode</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">BillingMode</span><span class="p">.</span><span class="nx">PAY_PER_REQUEST</span><span class="p">,</span>
      <span class="na">timeToLiveAttribute</span><span class="p">:</span> <span class="dl">"</span><span class="s2">ttl</span><span class="dl">"</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">createOrderCommandFn</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">lambda</span><span class="p">.</span><span class="nc">Function</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">CreateOrderCommandFn</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">runtime</span><span class="p">:</span> <span class="nx">lambda</span><span class="p">.</span><span class="nx">Runtime</span><span class="p">.</span><span class="nx">NODEJS_20_X</span><span class="p">,</span>
      <span class="na">handler</span><span class="p">:</span> <span class="dl">"</span><span class="s2">create-order-command.handler</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">code</span><span class="p">:</span> <span class="nx">lambda</span><span class="p">.</span><span class="nx">Code</span><span class="p">.</span><span class="nf">fromAsset</span><span class="p">(</span><span class="dl">"</span><span class="s2">dist</span><span class="dl">"</span><span class="p">),</span>
      <span class="na">environment</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">COMMAND_BUS_NAME</span><span class="p">:</span> <span class="nx">commandBus</span><span class="p">.</span><span class="nx">eventBusName</span><span class="p">,</span>
        <span class="na">IDEMPOTENCY_TABLE</span><span class="p">:</span> <span class="nx">idempotencyTable</span><span class="p">.</span><span class="nx">tableName</span>
      <span class="p">}</span>
    <span class="p">});</span>

    <span class="nx">commandBus</span><span class="p">.</span><span class="nf">grantPutEventsTo</span><span class="p">(</span><span class="nx">createOrderCommandFn</span><span class="p">);</span>
    <span class="nx">idempotencyTable</span><span class="p">.</span><span class="nf">grantReadWriteData</span><span class="p">(</span><span class="nx">createOrderCommandFn</span><span class="p">);</span>

    <span class="c1">// Placeholder state machine - replace with full definition</span>
    <span class="kd">const</span> <span class="nx">createOrderStateMachine</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">sfn</span><span class="p">.</span><span class="nc">StateMachine</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">CreateOrderStateMachine</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">definitionBody</span><span class="p">:</span> <span class="nx">sfn</span><span class="p">.</span><span class="nx">DefinitionBody</span><span class="p">.</span><span class="nf">fromString</span><span class="p">(</span><span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
        <span class="na">StartAt</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Done</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">States</span><span class="p">:</span> <span class="p">{</span> <span class="na">Done</span><span class="p">:</span> <span class="p">{</span> <span class="na">Type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Succeed</span><span class="dl">"</span> <span class="p">}</span> <span class="p">}</span>
      <span class="p">}))</span>
    <span class="p">});</span>

    <span class="k">new</span> <span class="nx">events</span><span class="p">.</span><span class="nc">Rule</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">RouteCreateOrderCommand</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">eventBus</span><span class="p">:</span> <span class="nx">commandBus</span><span class="p">,</span>
      <span class="na">eventPattern</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">source</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">com.example.orders.api</span><span class="dl">"</span><span class="p">],</span>
        <span class="na">detailType</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">CreateOrderCommand</span><span class="dl">"</span><span class="p">]</span>
      <span class="p">},</span>
      <span class="na">targets</span><span class="p">:</span> <span class="p">[</span><span class="k">new</span> <span class="nx">targets</span><span class="p">.</span><span class="nc">SfnStateMachine</span><span class="p">(</span><span class="nx">createOrderStateMachine</span><span class="p">)]</span>
    <span class="p">});</span>

    <span class="k">new</span> <span class="nx">cdk</span><span class="p">.</span><span class="nc">CfnOutput</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">CommandBusName</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">value</span><span class="p">:</span> <span class="nx">commandBus</span><span class="p">.</span><span class="nx">eventBusName</span> <span class="p">});</span>
    <span class="k">new</span> <span class="nx">cdk</span><span class="p">.</span><span class="nc">CfnOutput</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">DomainEventBusName</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">value</span><span class="p">:</span> <span class="nx">domainEventBus</span><span class="p">.</span><span class="nx">eventBusName</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  CDK notes from real projects
</h3>

<p>In production, I also add:</p>

<ul>
<li>DLQs where supported</li>
<li>alarms on failed executions and throttles</li>
<li>log retention policies</li>
<li>encryption settings</li>
<li>resource policies for cross-account publishing</li>
<li>least-privilege IAM per state/task</li>
<li>tagging and cost allocation labels</li>
</ul>




<h2>
  
  
  Auditability and tracing (this is where the pattern really pays off)
</h2>

<p>A lot of teams build async systems and then struggle to answer a simple question:</p>

<blockquote>
<p>“What happened to this request?”</p>
</blockquote>

<p>I design for that from day one.</p>

<h3>
  
  
  What I want to trace across the platform
</h3>

<p>At minimum:</p>

<ul>
<li>
<code>correlationId</code> (request-level trace)</li>
<li>
<code>commandId</code> / <code>eventId</code>
</li>
<li>
<code>causationId</code> (what produced this)</li>
<li>workflow execution ID / ARN</li>
<li>domain entity ID (e.g., <code>orderId</code>)</li>
<li>tenant ID (if multi-tenant)</li>
<li>timestamps and status transitions</li>
</ul>

<h3>
  
  
  Where I record this
</h3>

<p>I typically combine:</p>

<ul>
<li>
<strong>structured logs</strong> (CloudWatch Logs)</li>
<li>
<strong>execution history</strong> (Step Functions)</li>
<li>
<strong>traces</strong> (X-Ray / OpenTelemetry where applicable)</li>
<li>
<strong>audit store</strong> (DynamoDB or S3, depending on retention/query needs)</li>
</ul>

<h3>
  
  
  Audit log strategy (practical approach)
</h3>

<p>I usually write a normalized audit record for each important transition:</p>

<ul>
<li>command accepted</li>
<li>workflow started</li>
<li>step completed/failed</li>
<li>event published</li>
<li>consumer processed/failed</li>
</ul>

<p>This can be as simple as a DynamoDB table keyed by <code>correlationId</code> and <code>timestamp</code>, or an append-only S3 log for long-term retention.</p>

<h3>
  
  
  Important distinction: audit vs debug logs
</h3>

<ul>
<li>
<strong>Debug logs</strong> help developers troubleshoot</li>
<li>
<strong>Audit records</strong> help operators, compliance, and support teams reconstruct what happened</li>
</ul>

<p>Do not assume one replaces the other.</p>




<h2>
  
  
  Contract evolution (how to avoid breaking consumers)
</h2>

<p>This is one of the biggest long-term risks in event-driven systems.</p>

<h3>
  
  
  My default rule: evolve, do not break
</h3>

<p>For events, I prefer:</p>

<ul>
<li>additive changes</li>
<li>optional fields</li>
<li>versioned schemas</li>
<li>compatibility checks in CI</li>
<li>deprecation windows</li>
</ul>

<p>For commands, I am stricter because commands are business API boundaries.</p>

<h3>
  
  
  Practical contract evolution patterns
</h3>

<h4>
  
  
  1) Envelope version + payload schema version
</h4>

<p>I keep envelope metadata stable and version the business schema independently.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"meta"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"schemaVersion"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.1"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h4>
  
  
  2) Add fields, do not rename/remove casually
</h4>

<p>Safer changes:</p>

<ul>
<li>add optional field</li>
<li>add new event type (if semantics change)</li>
<li>add new metadata</li>
</ul>

<p>Riskier changes:</p>

<ul>
<li>renaming fields</li>
<li>changing field meaning</li>
<li>changing enum semantics</li>
<li>changing required/optional behavior</li>
</ul>

<h4>
  
  
  3) Prefer new event types when semantics change materially
</h4>

<p>If <code>OrderCreated</code> starts meaning something different, I usually create a new event type rather than mutating semantics.</p>

<h4>
  
  
  4) Validate contracts in CI
</h4>

<p>I recommend automated checks for:</p>

<ul>
<li>JSON schema validation</li>
<li>compatibility rules (backward/forward depending on consumer strategy)</li>
<li>sample payload fixtures</li>
<li>consumer contract tests</li>
</ul>

<h3>
  
  
  Command vs event evolution nuance
</h3>

<ul>
<li>
<strong>Commands</strong> are usually controlled at ingress and can be validated more strictly</li>
<li>
<strong>Events</strong> may have many consumers and require more conservative change management</li>
</ul>

<p>That is another reason to keep commands and events conceptually separate.</p>




<h2>
  
  
  Governance and domain boundaries (the part teams often postpone)
</h2>

<p>A command bus is not just a technical component. It is a governance surface.</p>

<p>If you skip governance, you will eventually get:</p>

<ul>
<li>inconsistent naming</li>
<li>ambiguous ownership</li>
<li>wildcard subscriptions everywhere</li>
<li>unbounded fan-out</li>
<li>accidental data exposure</li>
<li>“shared bus chaos”</li>
</ul>

<h3>
  
  
  Governance principles I use
</h3>

<h4>
  
  
  1) Explicit ownership by domain
</h4>

<p>Each command/event type should have a clear owner.</p>

<p>Examples:</p>

<ul>
<li>Orders owns <code>CreateOrder</code>, <code>OrderCreated</code>, <code>OrderRejected</code>
</li>
<li>Inventory owns <code>ReserveInventory</code> (if exposed), <code>InventoryReserved</code>
</li>
<li>Payments owns <code>PaymentAuthorized</code>, <code>PaymentFailed</code>
</li>
</ul>

<h4>
  
  
  2) Naming conventions
</h4>

<p>I standardize at least:</p>

<ul>
<li>
<code>source</code> values (e.g., <code>com.example.orders.workflow</code>)</li>
<li>
<code>detail-type</code> values (e.g., <code>OrderCreated</code>)</li>
<li>envelope metadata keys</li>
<li>versioning conventions</li>
</ul>

<p>This prevents drift and simplifies routing/analytics.</p>

<h4>
  
  
  3) Separate buses by concern (when warranted)
</h4>

<p>I often separate:</p>

<ul>
<li>
<strong>command bus</strong> (internal command routing)</li>
<li>
<strong>domain event bus</strong> (business events)</li>
<li>optional <strong>integration bus</strong> (external partner/integration events)</li>
</ul>

<p>In larger organizations, I may also separate by:</p>

<ul>
<li>environment (<code>dev</code>, <code>test</code>, <code>prod</code>)</li>
<li>business domain</li>
<li>account boundaries</li>
<li>compliance/data residency needs</li>
</ul>

<h4>
  
  
  4) Least privilege on publishers and subscribers
</h4>

<p>Producers should not be able to publish arbitrary event types to every bus. IAM and EventBridge resource policies should reflect domain boundaries.</p>

<h4>
  
  
  5) Data classification rules
</h4>

<p>Do not let sensitive payloads flow “because it is convenient.” Define:</p>

<ul>
<li>PII handling rules</li>
<li>masking/redaction requirements</li>
<li>retention policies</li>
<li>replay restrictions</li>
<li>external egress controls</li>
</ul>




<h2>
  
  
  Failure handling and resilience (what makes this production-grade)
</h2>

<p>Async architecture is easy to demo and hard to run well unless failure paths are designed intentionally.</p>

<h3>
  
  
  Areas I design explicitly
</h3>

<h4>
  
  
  Idempotency
</h4>

<p>Commands can be retried by clients, gateways, or operators. Idempotency is non-negotiable for business writes.</p>

<p>Use:</p>

<ul>
<li>client-provided idempotency key (preferred)</li>
<li>server-generated fallback only as backup</li>
<li>TTL on idempotency records to manage table growth</li>
</ul>

<h4>
  
  
  Retries and backoff
</h4>

<p>Retries should be defined per step, not copy-pasted blindly.</p>

<p>Questions I ask:</p>

<ul>
<li>Is the step retry-safe?</li>
<li>Is the downstream side effect idempotent?</li>
<li>What is the blast radius if retried?</li>
<li>Do I need jitter/backoff?</li>
</ul>

<h4>
  
  
  Dead-letter and replay
</h4>

<p>Some failures need operator intervention, not endless retries.</p>

<p>I usually implement:</p>

<ul>
<li>DLQ for failed async processing where supported</li>
<li>replay tooling with filters (by date, event type, correlation ID)</li>
<li>safe replay mode (e.g., dry-run validation before reprocessing)</li>
</ul>

<h4>
  
  
  Compensation
</h4>

<p>Not every workflow can be truly atomic. For distributed business flows, compensation matters.</p>

<p>Examples:</p>

<ul>
<li>if inventory reservation succeeds but payment later fails, release reservation</li>
<li>if payment succeeds but order persistence fails, void/mark payment for manual review</li>
</ul>

<p>Step Functions makes these paths explicit and testable.</p>




<h2>
  
  
  Testing strategy (what I actually test)
</h2>

<h3>
  
  
  1) Contract tests
</h3>

<ul>
<li>validate command/event envelopes against schemas</li>
<li>enforce required metadata fields</li>
<li>check backward compatibility rules</li>
</ul>

<h3>
  
  
  2) Workflow tests
</h3>

<ul>
<li>success path</li>
<li>step-specific failure paths</li>
<li>retries/timeouts</li>
<li>compensation branches</li>
</ul>

<h3>
  
  
  3) Consumer isolation tests
</h3>

<ul>
<li>consumers should handle unknown fields</li>
<li>consumers should tolerate duplicate events</li>
<li>consumers should reject malformed payloads safely</li>
</ul>

<h3>
  
  
  4) Observability assertions
</h3>

<ul>
<li>logs include correlation IDs</li>
<li>emitted events include causation/correlation metadata</li>
<li>alarms trigger on failed workflows</li>
</ul>

<p>I treat observability metadata as part of the contract, not a nice-to-have.</p>




<h2>
  
  
  When not to use this pattern
</h2>

<p>I like this pattern a lot, but I would not use it everywhere.</p>

<p>I avoid it when:</p>

<ul>
<li>the flow is very simple and synchronous</li>
<li>latency is critical and async orchestration adds unnecessary overhead</li>
<li>the team is not ready to operate event-driven systems yet</li>
<li>there is no real need for decoupling/auditability</li>
<li>a direct service call is clearer and sufficient</li>
</ul>

<p>Architecture should reduce complexity, not relocate it.</p>




<h2>
  
  
  A practical rollout plan (if you are introducing this incrementally)
</h2>

<p>If I were introducing this into an existing microservice platform, I would phase it in:</p>

<h3>
  
  
  Phase 1: Standardize envelopes and correlation IDs
</h3>

<p>Keep current calls, but enforce consistent metadata.</p>

<h3>
  
  
  Phase 2: Introduce EventBridge for non-critical domain events
</h3>

<p>Start with notifications/analytics fan-out.</p>

<h3>
  
  
  Phase 3: Move one business workflow to Step Functions
</h3>

<p>Choose a flow with clear business value and observable pain today.</p>

<h3>
  
  
  Phase 4: Add command routing and idempotent ingress
</h3>

<p>Introduce a command bus for selected workflows.</p>

<h3>
  
  
  Phase 5: Formalize governance and contract checks
</h3>

<p>Naming, ownership, CI validation, replay procedures, IAM boundaries.</p>

<p>This sequence reduces organizational friction while building platform confidence.</p>




<h2>
  
  
  Final thoughts
</h2>

<p>The biggest advantage of a serverless command bus/workflow backbone is not just that it is “event-driven” or “serverless.” It is that it gives your microservices ecosystem a <strong>clear execution model</strong>:</p>

<ul>
<li>commands express intent</li>
<li>workflows coordinate business actions</li>
<li>events publish facts</li>
<li>consumers react independently</li>
<li>audit and tracing are built in</li>
<li>governance is enforceable at boundaries</li>
</ul>

<p>That combination is what helps a platform scale across both <strong>services</strong> and <strong>teams</strong>.</p>

<p>If I am building for long-term maintainability, this is one of the patterns I reach for early, especially when I know business workflows will grow in complexity and scrutiny.</p>




<h2>
  
  
  References
</h2>

<ul>
<li>AWS EventBridge documentation</li>
<li>AWS Step Functions documentation</li>
<li>AWS Lambda developer guide</li>
<li>AWS API Gateway developer guide</li>
<li>AWS CDK documentation</li>
<li>AWS X-Ray and observability documentation</li>
<li>AWS Well-Architected Framework (especially reliability and operational excellence guidance)</li>
<li>Event-driven architecture patterns and domain-driven design references (commands/events, bounded contexts, integration events)</li>
</ul>

