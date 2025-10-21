---
Title: Simply Order (Part 5) — Hands-On: Building the Outbox Pattern for Reliable Event
Description: 
Author: Mohamed Hassan
Date: 2025-10-21T22:05:30.000Z
Robots: noindex,nofollow
Template: index
---
<p>This is the fifth article in our series, where we design a simple order solution for a hypothetical company called <strong>Simply Order</strong>. The company expects high traffic and needs a resilient, scalable, and distributed order system.</p>

<p>In the previous articles:</p>

<ul>
<li><a href="https://dev.to/hassan314159/distributed-transactions-in-microservices-why-2pc-doesnt-fit-and-how-sagas-help-1lb">Simply Order (Part 1) Distributed Transactions in Microservices: Why 2PC Doesn’t Fit and How Sagas Help</a></li>
<li><a href="https://dev.to/hassan314159/simply-order-part-2-designing-and-implementing-the-saga-workflow-with-temporal-3o23">Simply Order (Part 2) — Designing and Implementing the Saga Workflow with Temporal</a></li>
<li><a href="https://dev.to/hassan314159/simply-order-part-3-linking-it-all-together-connecting-services-and-watching-temporal-in-action-19oe">Simply Order (Part 3) — Linking It All Together: Connecting Services and Watching Temporal in Action</a></li>
<li><a href="https://dev.to/hassan314159/simply-order-part-4-reliable-events-with-the-outbox-pattern-concepts-55ko">Simply Order (Part 4) — Reliable Events with the Outbox Pattern (Concepts)</a></li>
</ul>

<p>We built the core services — Order, Payment, and Inventory — and discussed different approaches for handling distributed transactions across multiple services. Then, we designed and implemented the Saga workflow. Finally, we introduced the problem of dual-write consistency and how the Outbox Pattern can solve it.</p>

<p>In this article, we’ll show how to implement the Outbox Pattern with a polling relay using PostgreSQL. Then, we’ll integrate this logic into our Order Service and see it in action.</p>

<blockquote>
<p>The code for this project can be found in this repository:<br>
<a href="https://github.com/hassan314159/simply-order" rel="noopener noreferrer">https://github.com/hassan314159/simply-order</a></p>

<p>Since this repository is continuously updated, the code specific to this lesson can be found in the <em>add_persistence_to_order_service</em>. branch. Start with:<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git checkout add_persistence_to_order_service
</code></pre>

</div>



<h2>
  
  
  Outbox Entity Definition
</h2>

<p>Lets have a look about our <code>OutboxEntity</code> definition. You can find the code in <strong>OrderService</strong> under the package <code>dev.simplyoder.order.infra.outbox</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Entity</span>
<span class="nd">@Table</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"outbox"</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OutboxEntity</span> <span class="o">{</span>
    <span class="nd">@Id</span>
    <span class="kd">private</span> <span class="no">UUID</span> <span class="n">id</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"event_type"</span><span class="o">,</span> <span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">,</span> <span class="n">length</span> <span class="o">=</span> <span class="mi">100</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">eventType</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"aggregate_id"</span><span class="o">,</span> <span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">)</span>
    <span class="kd">private</span> <span class="no">UUID</span> <span class="n">aggregateId</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"payload"</span><span class="o">,</span> <span class="n">columnDefinition</span> <span class="o">=</span> <span class="s">"jsonb"</span><span class="o">,</span> <span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">)</span>
    <span class="nd">@JdbcTypeCode</span><span class="o">(</span><span class="nc">SqlTypes</span><span class="o">.</span><span class="na">JSON</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">payload</span><span class="o">;</span>

    <span class="nd">@Enumerated</span><span class="o">(</span><span class="nc">EnumType</span><span class="o">.</span><span class="na">STRING</span><span class="o">)</span>
    <span class="nd">@Column</span><span class="o">(</span><span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">,</span> <span class="n">length</span> <span class="o">=</span> <span class="mi">16</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">Status</span> <span class="n">status</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">)</span>
    <span class="kd">private</span> <span class="kt">int</span> <span class="n">attempts</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"available_at"</span><span class="o">,</span> <span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">Instant</span> <span class="n">availableAt</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"created_at"</span><span class="o">,</span> <span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">,</span> <span class="n">updatable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">Instant</span> <span class="n">createdAt</span><span class="o">;</span>

    <span class="nd">@Version</span>
    <span class="kd">private</span> <span class="kt">long</span> <span class="n">version</span><span class="o">;</span>

    <span class="kd">public</span> <span class="kd">enum</span> <span class="nc">Status</span> <span class="o">{</span> <span class="no">PENDING</span><span class="o">,</span> <span class="no">SENT</span><span class="o">,</span> <span class="no">FAILED</span> <span class="o">}</span>

</code></pre>

</div>



<p>Where </p>

<ul>
<li>
<code>id</code>: the ID of the outbox record.</li>
<li>
<code>eventType</code>: the type of event, e.g., <strong>OrderCreated</strong>
</li>
<li>
<code>aggregateId</code>: the ID of the domain object, e.g., <strong>OrderId</strong>
</li>
<li>
<code>payload</code>: the event payload, e.g., the created order record</li>
<li>
<code>status</code>: the status of the outbox record; one of:

<ul>
<li>
<strong>PENDING</strong>: new record to be polled by the relay</li>
<li>
<strong>SENT</strong>: polled and executed successfully</li>
<li>
<strong>FAILED</strong>: polled but execution failed (after a number of attempts)</li>
</ul>


</li>

<li>
<code>attempts</code>: number of attempts before marking the record as <strong>FAILED</strong>
</li>

<li>
<code>createdAt</code>: the creation timestamp of the outbox record</li>

<li>
<code>availableAt</code>: when the record becomes eligible for execution. It starts as <code>createdAt</code> (i.e., available immediately) and is updated on failure to support a backoff mechanism—delaying the next attempt by some time.</li>

</ul>

<h2>
  
  
  Outbox Relay Logic
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Scheduled</span><span class="o">(</span><span class="n">fixedDelay</span> <span class="o">=</span> <span class="mi">500</span><span class="o">)</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">orderWorkFlowSchedular</span><span class="o">()</span> <span class="o">{</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">OutboxEntity</span><span class="o">&gt;</span> <span class="n">items</span> <span class="o">=</span> <span class="n">outboxRepo</span><span class="o">.</span><span class="na">lockNextBatch</span><span class="o">(</span><span class="nc">Instant</span><span class="o">.</span><span class="na">now</span><span class="o">(),</span> <span class="n">batchSize</span><span class="o">);</span> <span class="c1">// 1</span>
    <span class="k">for</span> <span class="o">(</span><span class="nc">OutboxEntity</span> <span class="n">ob</span> <span class="o">:</span> <span class="n">items</span><span class="o">)</span> <span class="o">{</span> <span class="c1">// 2</span>
        <span class="k">try</span> <span class="o">{</span>
            <span class="nc">String</span> <span class="n">workflowId</span> <span class="o">=</span> <span class="s">"order-"</span> <span class="o">+</span> <span class="n">ob</span><span class="o">.</span><span class="na">getAggregateId</span><span class="o">();</span>

            <span class="nc">WorkflowOptions</span> <span class="n">options</span> <span class="o">=</span> <span class="nc">WorkflowOptions</span><span class="o">.</span><span class="na">newBuilder</span><span class="o">()</span>
                    <span class="o">.</span><span class="na">setTaskQueue</span><span class="o">(</span><span class="s">"order-task-queue"</span><span class="o">)</span>
                    <span class="o">.</span><span class="na">setWorkflowId</span><span class="o">(</span><span class="n">workflowId</span><span class="o">)</span>
                    <span class="o">.</span><span class="na">setWorkflowIdReusePolicy</span><span class="o">(</span><span class="no">WORKFLOW_ID_REUSE_POLICY_REJECT_DUPLICATE</span><span class="o">)</span>
                    <span class="o">.</span><span class="na">build</span><span class="o">();</span>

            <span class="nc">OrderWorkflow</span> <span class="n">wf</span> <span class="o">=</span> <span class="n">temporalClient</span><span class="o">.</span><span class="na">newWorkflowStub</span><span class="o">(</span><span class="nc">OrderWorkflow</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="n">options</span><span class="o">);</span>

            <span class="n">wf</span><span class="o">.</span><span class="na">placeOrder</span><span class="o">(</span><span class="n">ob</span><span class="o">.</span><span class="na">getPayload</span><span class="o">());</span>
            <span class="n">ob</span><span class="o">.</span><span class="na">markSent</span><span class="o">();</span> <span class="c1">// 3</span>

        <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">WorkflowServiceException</span> <span class="n">e</span><span class="o">)</span> <span class="o">{</span>
            <span class="c1">// Workflow already exists with the same ID? Treat as success (idempotent start).</span>
            <span class="k">if</span> <span class="o">(</span><span class="n">isAlreadyStarted</span><span class="o">(</span><span class="n">e</span><span class="o">))</span> <span class="o">{</span> <span class="c1">// 4</span>
                <span class="n">ob</span><span class="o">.</span><span class="na">markSent</span><span class="o">();</span>
            <span class="o">}</span> <span class="k">else</span> <span class="o">{</span>
                <span class="c1">// backoff</span>
                <span class="k">if</span> <span class="o">(</span><span class="n">ob</span><span class="o">.</span><span class="na">getAttempts</span><span class="o">()</span> <span class="o">+</span> <span class="mi">1</span> <span class="o">&gt;=</span> <span class="n">maxAttempts</span><span class="o">)</span> <span class="o">{</span>
                    <span class="n">ob</span><span class="o">.</span><span class="na">fail</span><span class="o">();</span>
                <span class="o">}</span> <span class="k">else</span> <span class="o">{</span>
                    <span class="n">ob</span><span class="o">.</span><span class="na">reschedule</span><span class="o">(</span><span class="n">nextBackoff</span><span class="o">(</span><span class="n">ob</span><span class="o">.</span><span class="na">getAttempts</span><span class="o">()));</span>  <span class="c1">// 5</span>
                <span class="o">}</span>
            <span class="o">}</span>
        <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">Exception</span> <span class="n">e</span><span class="o">)</span> <span class="o">{</span>
            <span class="k">if</span> <span class="o">(</span><span class="n">ob</span><span class="o">.</span><span class="na">getAttempts</span><span class="o">()</span> <span class="o">+</span> <span class="mi">1</span> <span class="o">&gt;=</span> <span class="n">maxAttempts</span><span class="o">)</span> <span class="o">{</span>
                <span class="n">ob</span><span class="o">.</span><span class="na">fail</span><span class="o">();</span> <span class="c1">// 6</span>
            <span class="o">}</span> <span class="k">else</span> <span class="o">{</span>
                <span class="n">ob</span><span class="o">.</span><span class="na">reschedule</span><span class="o">(</span><span class="n">nextBackoff</span><span class="o">(</span><span class="n">ob</span><span class="o">.</span><span class="na">getAttempts</span><span class="o">()));</span>
            <span class="o">}</span>
        <span class="o">}</span>
    <span class="o">}</span>
    <span class="n">outboxRepo</span><span class="o">.</span><span class="na">saveAll</span><span class="o">(</span><span class="n">items</span><span class="o">);</span>
<span class="o">}</span>

</code></pre>

</div>



<p>The Outbox Relay is implemented as a simple scheduled task in our Order Service, running every <em>500 ms</em>  using <code>@Scheduled(fixedDelay = 500)</code>.<br>
It performs the following steps:<br>
1- Fetches and locks the next batch of outbox records using <code>outboxRepo.lockNextBatch</code><br>
2- Loops through each record and starts a new transaction or Temporal workflow using the <em>payload</em> and <em>OrderId (aggregateId)</em>.<br>
3- Updates the outbox record <code>status</code> to SENT in case of success — <code>ob.markSent();</code>.<br>
4- If a workflow has already been started for the same OrderId, it marks the outbox record as <strong>SENT</strong> as well —  <code>if (isAlreadyStarted(e))</code><br>
5- Updates the <code>availableAt</code> field in case of failure (but only if the number of attempts has not yet been exceeded).<br>
6- If the number of attempts exceeds the limit, it updates the <code>status</code> to <em>FAILED</em></p>

<blockquote>
<p><code>lockNextBatch</code> in <em>OutboxRepository</em> — fetches and locks the selected records to ensure that, if multiple service instances or relays are running, each outbox record is processed by only one relay.</p>

<p>To support idempotency (i.e., preventing the same workflow from running twice for the same ID), we configured our workflow with: <code>.setWorkflowIdReusePolicy(WORKFLOW_ID_REUSE_POLICY_REJECT_DUPLICATE)</code> </p>

<p>In our design, the payload contains the full Order record — including both <code>Order</code> and <code>OrderItems</code>. We chose this approach to make the relay self-contained and independent of the order business logic. In fact, the workflow and outbox code could be completely moved out of the <strong>Order Service</strong> into a standalone service, but for simplicity, we kept it within the Order Service.</p>
</blockquote>
<h2>
  
  
  Updated Order Logic
</h2>
<h3>
  
  
  Order Entity
</h3>

<p>We use <strong>JPA</strong> to persist order entities in our <strong>PostgreSQL</strong> database.<br>
Our repositories are implemented using <strong>Spring Data</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Repository</span>
<span class="kd">public</span> <span class="kd">interface</span> <span class="nc">OrderRepository</span> <span class="kd">extends</span> <span class="nc">JpaRepository</span><span class="o">&lt;</span><span class="nc">OrderEntity</span><span class="o">,</span> <span class="no">UUID</span><span class="o">&gt;</span> <span class="o">{</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This allows us to handle all basic CRUD operations without writing boilerplate code, while still being able to define custom queries when needed.</p>

<p>Here is the <code>OrderEntity</code> class.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Entity</span>
<span class="nd">@Table</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"orders"</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OrderEntity</span><span class="o">{</span>

    <span class="nd">@Id</span>
    <span class="kd">private</span> <span class="no">UUID</span> <span class="n">id</span><span class="o">;</span>

    <span class="kd">private</span> <span class="no">UUID</span> <span class="n">customerId</span><span class="o">;</span>

    <span class="nd">@Enumerated</span><span class="o">(</span><span class="nc">EnumType</span><span class="o">.</span><span class="na">STRING</span><span class="o">)</span>
    <span class="nd">@Column</span><span class="o">(</span><span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">,</span> <span class="n">length</span> <span class="o">=</span> <span class="mi">40</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">OrderStatus</span> <span class="n">status</span> <span class="o">=</span> <span class="nc">OrderStatus</span><span class="o">.</span><span class="na">OPEN</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">,</span> <span class="n">precision</span> <span class="o">=</span> <span class="mi">19</span><span class="o">,</span> <span class="n">scale</span> <span class="o">=</span> <span class="mi">2</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">BigDecimal</span> <span class="n">total</span> <span class="o">=</span> <span class="nc">BigDecimal</span><span class="o">.</span><span class="na">ZERO</span><span class="o">;</span>

    <span class="nd">@OneToMany</span><span class="o">(</span><span class="n">mappedBy</span> <span class="o">=</span> <span class="s">"order"</span><span class="o">,</span> <span class="n">cascade</span> <span class="o">=</span> <span class="nc">CascadeType</span><span class="o">.</span><span class="na">ALL</span><span class="o">,</span> <span class="n">orphanRemoval</span> <span class="o">=</span> <span class="kc">true</span><span class="o">)</span>
    <span class="nd">@JsonManagedReference</span>
    <span class="kd">private</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">OrderItemEntity</span><span class="o">&gt;</span> <span class="n">items</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayList</span><span class="o">&lt;</span><span class="nc">OrderItemEntity</span><span class="o">&gt;();</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"created_at"</span><span class="o">,</span> <span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">,</span> <span class="n">updatable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">OffsetDateTime</span> <span class="n">createdAt</span><span class="o">;</span>

    <span class="nd">@Column</span><span class="o">(</span><span class="n">name</span> <span class="o">=</span> <span class="s">"updated_at"</span><span class="o">,</span> <span class="n">nullable</span> <span class="o">=</span> <span class="kc">false</span><span class="o">)</span>
    <span class="kd">private</span> <span class="nc">OffsetDateTime</span> <span class="n">updatedAt</span><span class="o">;</span>

</code></pre>

</div>



<h3>
  
  
  Order Service
</h3>

<p>Right now, the Order Service is much simpler and focused purely on the order domain.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="no">UUID</span> <span class="nf">createOrder</span><span class="o">(</span><span class="nc">CreateOrderRequest</span> <span class="n">request</span><span class="o">)</span> <span class="kd">throws</span> <span class="nc">JsonProcessingException</span> <span class="o">{</span>
    <span class="no">UUID</span> <span class="n">orderId</span> <span class="o">=</span> <span class="no">UUID</span><span class="o">.</span><span class="na">randomUUID</span><span class="o">();</span>
    <span class="nc">CreateOrderCommand</span> <span class="n">cmd</span> <span class="o">=</span> <span class="nc">CreateOrderCommand</span><span class="o">.</span><span class="na">from</span><span class="o">(</span><span class="n">orderId</span><span class="o">,</span> <span class="n">request</span><span class="o">);</span>
    <span class="nc">OrderEntity</span> <span class="n">order</span> <span class="o">=</span> <span class="nc">OrderEntity</span><span class="o">.</span><span class="na">create</span><span class="o">(</span><span class="n">cmd</span><span class="o">);</span>
    <span class="n">orderRepo</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span> <span class="c1">//1</span>

    <span class="nc">String</span> <span class="n">payload</span> <span class="o">=</span> <span class="n">objectMapper</span><span class="o">.</span><span class="na">writeValueAsString</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="n">outboxRepo</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="nc">OutboxEntity</span><span class="o">.</span><span class="na">pending</span><span class="o">(</span><span class="s">"OrderCreated"</span><span class="o">,</span> <span class="n">orderId</span><span class="o">,</span> <span class="n">payload</span><span class="o">));</span> <span class="c1">//2</span>

    <span class="k">return</span> <span class="n">orderId</span><span class="o">;</span>
<span class="o">}</span>

</code></pre>

</div>



<p>The <code>OrderService</code> logic consists of just two operations in a single transaction:<br>
1- Create the <code>Order</code> entity<br>
2- Create the <code>Outbox</code>order</p>
<h2>
  
  
  Run Application
</h2>

<p>As discussed in <a href="https://dev.to/hassan314159/simply-order-part-3-linking-it-all-together-connecting-services-and-watching-temporal-in-action-19oe">Lesson 3</a>, </p>

<p>to run the application<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd</span> ./deploy/local
docker compose up
</code></pre>

</div>



<p>please follow the same steps mentioned in Lesson 3 to create orders.</p>

<h2>
  
  
  Home Work
</h2>

<p>Try stopping the Temporal service and then create a new order.<br>
After restarting Temporal, you’ll notice that the order gets created and the distributed transaction continues automatically once Temporal becomes healthy again.</p>

<h2>
  
  
  Wrapping Up
</h2>

<p>In this lesson, we implemented the Outbox Pattern with a polling relay using PostgreSQL and integrated it into our Order Service.<br>
We saw how the relay reliably picks up pending events, triggers the corresponding Temporal workflows, and ensures eventual consistency across distributed services — even in cases of temporary failures or service downtime.</p>

