---
Title: What If Your Database Goes Down? REST vs Kafka Under Fire
Description: 
Author: Bhupesh Chikara
Date: 2026-02-26T22:01:13.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  What If Your Database Goes Down? REST vs Kafka Under Fire
</h1>

<p>Companies like Uber, Netflix, and Airbnb have migrated from traditional REST APIs to event-driven architectures. This shift isn't driven by trends, but by fundamental architectural resilience requirements. To quantify these differences, I built a production-grade chaos engineering proof-of-concept, simulating real-world database failures under sustained load.</p>

<h2>
  
  
  The Hypothesis
</h2>

<p>Traditional synchronous REST APIs create tight coupling between API servers and databases. When the database fails, the API fails. Event-driven architectures using Kafka decouple producers from consumers through message buffering, theoretically providing resilience during infrastructure failures.</p>

<p>We designed an experiment to measure this resilience difference empirically.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5s7g07yrdedkdf2l9yud.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5s7g07yrdedkdf2l9yud.png" alt="REST vs Kafka Architecture Comparison" width="800" height="436"></a></p>

<h2>
  
  
  Experiment Design
</h2>

<h3>
  
  
  Scenario
</h3>

<p>We simulated Uber's real-time driver location tracking system, where high-frequency updates must be reliably persisted to PostgreSQL.</p>

<h3>
  
  
  Infrastructure
</h3>

<ul>
<li>
<strong>Load Pattern</strong>: Constant 50 requests/second</li>
<li>
<strong>Failure Injection</strong>: PostgreSQL crash for 120 seconds (t=90s to t=210s)</li>
<li>
<strong>Duration</strong>: 5 minutes total (300 seconds)</li>
<li>
<strong>Load Testing</strong>: k6 with automated orchestration</li>
<li>
<strong>Monitoring</strong>: Prometheus + Grafana for real-time metrics collection</li>
</ul>

<h3>
  
  
  Architectures Tested
</h3>

<p><strong>Architecture A: Synchronous REST</strong></p>

<ul>
<li>Direct HTTP POST to REST API</li>
<li>Immediate database INSERT</li>
<li>Response after database confirmation</li>
</ul>

<p><strong>Architecture B: Asynchronous Kafka</strong></p>

<ul>
<li>HTTP POST to Producer API</li>
<li>Event published to Kafka topic</li>
<li>Consumer processes events with circuit breaker pattern</li>
<li>Asynchronous database INSERT</li>
</ul>

<p>Both architectures handled identical traffic patterns and experienced the same database failure window.</p>

<h2>
  
  
  Results
</h2>

<h3>
  
  
  REST API Performance
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Metric                  Value
Total Requests         30,002
Successful             15,001
Failed                 15,001
Error Rate             50.00%
P50 Latency            3.60ms
P95 Latency            15.55ms
Average Latency        11.24ms
</code></pre>

</div>



<p><strong>Failure Characteristics:</strong></p>

<ul>
<li>Immediate error propagation to clients</li>
<li>No request buffering capability</li>
<li>Manual intervention required for recovery</li>
<li>50% of requests returned HTTP 500 errors</li>
<li>Complete service degradation during database outage</li>
</ul>

<h3>
  
  
  Kafka Architecture Performance
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Metric                  Value
Total Requests         15,000
Successful             15,000
Failed                 0
Error Rate             0.00%
P50 Latency            3.47ms
P95 Latency            12.10ms
Average Latency        6.77ms
Latency Improvement    39.77%
</code></pre>

</div>



<p><strong>Failure Characteristics:</strong></p>

<ul>
<li>Zero client-facing errors</li>
<li>Automatic request buffering in Kafka topics</li>
<li>Circuit breaker pattern enabled graceful degradation</li>
<li>Automatic recovery when database restored</li>
<li>Transparent failure handling from client perspective</li>
</ul>

<h2>
  
  
  Architecture Analysis
</h2>

<h3>
  
  
  REST: Synchronous Coupling
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Client → REST API → PostgreSQL → Response
           ↓ (if DB fails)
       HTTP 500 Error
</code></pre>

</div>



<p><strong>Failure Mode:</strong><br>
When PostgreSQL becomes unavailable, the REST API has no buffering mechanism. Each incoming request attempts a database connection, fails after timeout (2000ms), and returns an error to the client. This creates a cascading failure pattern where API health is directly tied to database availability.</p>

<p><strong>Characteristics:</strong></p>

<ul>
<li>Strong consistency guarantees</li>
<li>Immediate failure feedback</li>
<li>No request buffering</li>
<li>Tight coupling between components</li>
<li>Simple operational model</li>
</ul>
<h3>
  
  
  Kafka: Asynchronous Decoupling
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Client → Producer API → Kafka Topic → Consumer → PostgreSQL
         ↓ (immediate)           ↓ (buffered)
     HTTP 202 Accepted     Circuit Breaker
</code></pre>

</div>


<p><strong>Failure Mode:</strong><br>
When PostgreSQL fails, the Producer API continues accepting requests and publishing events to Kafka. The Consumer detects database failures through the circuit breaker pattern, transitions to OPEN state, and stops attempting writes. Events accumulate in Kafka's persistent log. When PostgreSQL recovers, the circuit breaker transitions to HALF_OPEN, tests connectivity, and resumes processing the buffered events.</p>

<p><strong>Characteristics:</strong></p>

<ul>
<li>Eventual consistency model</li>
<li>Request buffering via Kafka topics</li>
<li>Automatic failure detection and recovery</li>
<li>Component independence</li>
<li>Complex operational requirements</li>
</ul>
<h2>
  
  
  Circuit Breaker Implementation
</h2>

<p>The circuit breaker pattern is critical for preventing cascading failures in distributed systems. Our implementation uses a state machine with three states:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">class</span> <span class="nc">CircuitBreaker</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="nx">threshold</span> <span class="o">=</span> <span class="mi">5</span><span class="p">,</span> <span class="nx">timeout</span> <span class="o">=</span> <span class="mi">30000</span><span class="p">,</span> <span class="nx">resetTimeout</span> <span class="o">=</span> <span class="mi">60000</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">state</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">CLOSED</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">failureThreshold</span> <span class="o">=</span> <span class="nx">threshold</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">resetTimeout</span> <span class="o">=</span> <span class="nx">resetTimeout</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">failureCount</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">nextAttempt</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">execute</span><span class="p">(</span><span class="nx">fn</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">state</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">OPEN</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">&lt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">nextAttempt</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Circuit breaker OPEN - database unavailable</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">}</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">state</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">HALF_OPEN</span><span class="dl">'</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fn</span><span class="p">();</span>
      <span class="k">this</span><span class="p">.</span><span class="nf">onSuccess</span><span class="p">();</span>
      <span class="k">return</span> <span class="nx">result</span><span class="p">;</span>
    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nf">onFailure</span><span class="p">();</span>
      <span class="k">throw</span> <span class="nx">err</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="nf">onSuccess</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">failureCount</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">state</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">HALF_OPEN</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">successCount</span><span class="o">++</span><span class="p">;</span>
      <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">successCount</span> <span class="o">&gt;=</span> <span class="mi">3</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">state</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">CLOSED</span><span class="dl">'</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">successCount</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="nf">onFailure</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">failureCount</span><span class="o">++</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">failureCount</span> <span class="o">&gt;=</span> <span class="k">this</span><span class="p">.</span><span class="nx">failureThreshold</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">state</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">OPEN</span><span class="dl">'</span><span class="p">;</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">nextAttempt</span> <span class="o">=</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()</span> <span class="o">+</span> <span class="k">this</span><span class="p">.</span><span class="nx">resetTimeout</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>State Transitions:</strong></p>

<ul>
<li>
<strong>CLOSED</strong>: Normal operation, all requests processed</li>
<li>
<strong>OPEN</strong>: Failure threshold exceeded, stop attempting database writes</li>
<li>
<strong>HALF_OPEN</strong>: Testing recovery, require 3 consecutive successes before full recovery</li>
</ul>

<h2>
  
  
  Business Impact Analysis
</h2>

<h3>
  
  
  Assumptions
</h3>

<ul>
<li>Request rate: 1,000 requests/second</li>
<li>Revenue per request: $0.01</li>
<li>Database outage duration: 5 minutes</li>
<li>Error rate impact: 50% (REST) vs 0% (Kafka)</li>
</ul>

<h3>
  
  
  REST Architecture Impact
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Failed Requests:    150,000 (50% × 1,000 req/s × 300s)
Revenue Loss:       $1,500
Recovery Time:      Manual intervention required
Customer Impact:    Severe degradation, visible errors
</code></pre>

</div>



<h3>
  
  
  Kafka Architecture Impact
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Failed Requests:    0 (buffered in Kafka)
Revenue Loss:       $0
Recovery Time:      Automatic
Customer Impact:    None (transparent to clients)
</code></pre>

</div>



<p><strong>ROI Calculation:</strong><br>
At scale, assuming one 5-minute database outage per month, Kafka prevents $18,000 in annual revenue loss. This excludes:</p>

<ul>
<li>Customer support costs from incident tickets</li>
<li>Engineering time for manual recovery</li>
<li>Reputational damage from service degradation</li>
<li>Compliance implications for SLA violations</li>
</ul>
<h2>
  
  
  Technical Stack
</h2>

<ul>
<li>
<strong>Runtime</strong>: Node.js 18+ with Express</li>
<li>
<strong>Message Broker</strong>: Apache Kafka 3.x with KafkaJS client</li>
<li>
<strong>Database</strong>: PostgreSQL 15 with connection pooling</li>
<li>
<strong>Load Testing</strong>: k6 with custom JavaScript scenarios</li>
<li>
<strong>Monitoring</strong>: Prometheus 2.x + Grafana 9.x</li>
<li>
<strong>Orchestration</strong>: Docker Compose for reproducible environments</li>
<li>
<strong>Chaos Engineering</strong>: Automated failure injection scripts</li>
</ul>
<h2>
  
  
  Observability Metrics
</h2>
<h3>
  
  
  Prometheus Instrumentation
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// REST API Metrics</span>
<span class="nx">rest_http_requests_total</span><span class="p">{</span><span class="nx">status</span><span class="o">=</span><span class="dl">"</span><span class="s2">200|500</span><span class="dl">"</span><span class="p">}</span>
<span class="nx">rest_request_duration_seconds</span><span class="p">{</span><span class="nx">quantile</span><span class="o">=</span><span class="dl">"</span><span class="s2">0.5|0.95|0.99</span><span class="dl">"</span><span class="p">}</span>

<span class="c1">// Kafka Producer Metrics</span>
<span class="nx">kafka_produce_total</span>
<span class="nx">kafka_produce_latency_milliseconds</span>
<span class="nx">kafka_produce_errors_total</span>

<span class="c1">// Consumer Metrics with Circuit Breaker</span>
<span class="nx">circuit_breaker_state</span><span class="p">{</span><span class="nx">state</span><span class="o">=</span><span class="dl">"</span><span class="s2">CLOSED|HALF_OPEN|OPEN</span><span class="dl">"</span><span class="p">}</span>
<span class="nx">kafka_consumer_lag_seconds</span>
<span class="nx">db_write_failures_total</span>
<span class="nx">db_connection_pool_size</span>
</code></pre>

</div>


<p>These metrics enabled real-time failure detection and post-mortem analysis of system behavior during the outage window.</p>
<h2>
  
  
  Reproduction Guide
</h2>
<h3>
  
  
  Prerequisites
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install k6 load testing tool</span>
brew <span class="nb">install </span>k6  <span class="c"># macOS</span>
<span class="nb">sudo </span>apt-get <span class="nb">install </span>k6  <span class="c"># Linux</span>

<span class="c"># Clone repository</span>
git clone https://github.com/builtbychikara/WhatIfSeries
<span class="nb">cd </span>WhatIfSeries/what-if/kafka-vs-rest-polling
</code></pre>

</div>

<h3>
  
  
  Infrastructure Startup
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start Kafka, PostgreSQL, Prometheus, and Grafana</span>
docker-compose up <span class="nt">-d</span>

<span class="c"># Verify all services are healthy</span>
docker-compose ps
</code></pre>

</div>

<h3>
  
  
  Service Deployment
</h3>

<p>Open three terminal windows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Terminal 1: REST API (Port 3001)</span>
npm run start:rest

<span class="c"># Terminal 2: Kafka Producer API (Port 3002)</span>
npm run start:kafka

<span class="c"># Terminal 3: Kafka Consumer with Circuit Breaker (Port 3003)</span>
npm run start:consumer
</code></pre>

</div>



<h3>
  
  
  Execute Experiment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run automated chaos engineering experiment</span>
npm run experiment
</code></pre>

</div>



<h3>
  
  
  Experiment Timeline
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Phase         Time        Load        Database State
Warmup        0-30s       25 req/s    Healthy
Normal        30-90s      50 req/s    Healthy
Crash         90-210s     50 req/s    OFFLINE
Recovery      210-300s    50 req/s    Healthy
</code></pre>

</div>



<p>The experiment script automatically injects the database failure at t=90s and restores it at t=210s, while maintaining constant load throughout.</p>

<h2>
  
  
  Key Findings
</h2>

<h3>
  
  
  1. Resilience Through Decoupling
</h3>

<p>Event-driven architectures achieve resilience not through redundancy, but through temporal decoupling. Kafka's persistent log provides a buffer that absorbs transient failures, converting availability problems into latency problems.</p>

<h3>
  
  
  2. Circuit Breaker Necessity
</h3>

<p>Without the circuit breaker pattern, the Kafka consumer would continuously retry failed database writes, wasting resources and potentially overwhelming the database during recovery. The circuit breaker provides:</p>

<ul>
<li>Automatic failure detection</li>
<li>Resource conservation during outages</li>
<li>Controlled recovery testing</li>
<li>Metrics for failure state visibility</li>
</ul>

<h3>
  
  
  3. Consistency Trade-offs
</h3>

<p>REST provides strong consistency - clients know immediately whether their write succeeded. Kafka provides eventual consistency - clients receive acknowledgment that the write is buffered, not that it's persisted. This trade-off must align with business requirements.</p>

<h3>
  
  
  4. Operational Complexity
</h3>

<p>Kafka introduces operational overhead:</p>

<ul>
<li>Additional infrastructure (brokers, ZooKeeper/KRaft)</li>
<li>Message ordering guarantees to maintain</li>
<li>Consumer lag monitoring requirements</li>
<li>Topic partition management</li>
<li>Schema evolution considerations</li>
</ul>

<p>This complexity must be justified by resilience requirements.</p>

<h2>
  
  
  Decision Framework
</h2>

<h3>
  
  
  Choose REST When:
</h3>

<ol>
<li>
<strong>Traffic Patterns</strong>: Request rate under 100 req/s</li>
<li>
<strong>Consistency Requirements</strong>: Strong consistency required</li>
<li>
<strong>Team Capabilities</strong>: Limited operational expertise</li>
<li>
<strong>System Simplicity</strong>: Monolithic architecture preferred</li>
<li>
<strong>Latency Requirements</strong>: Sub-10ms response times required</li>
</ol>

<h3>
  
  
  Choose Kafka When:
</h3>

<ol>
<li>
<strong>Traffic Patterns</strong>: Request rate exceeds 1,000 req/s</li>
<li>
<strong>Resilience Requirements</strong>: Infrastructure failures must be transparent</li>
<li>
<strong>Event Consumers</strong>: Multiple downstream systems need the same events</li>
<li>
<strong>Consistency Trade-offs</strong>: Eventual consistency acceptable</li>
<li>
<strong>Operational Maturity</strong>: Team can manage distributed systems</li>
</ol>

<h2>
  
  
  Future Work
</h2>

<p>This experiment is part of the What If Series, exploring system design decisions through empirical measurement rather than theoretical analysis.</p>

<p><strong>Upcoming Experiments:</strong></p>

<ul>
<li>High-throughput scaling: Measuring Kafka performance at 1M+ req/s</li>
<li>Cache failure analysis: Redis outage impact on application tier</li>
<li>Real-time processing: Stream processing 1TB of logs with sub-second latency</li>
<li>Network partition testing: Consistency guarantees under split-brain scenarios</li>
</ul>

<h2>
  
  
  Code Repository
</h2>

<p>Full implementation including:</p>

<ul>
<li>Complete source code for both architectures</li>
<li>Docker Compose infrastructure definitions</li>
<li>k6 load testing scenarios</li>
<li>Prometheus metric exporters</li>
<li>Automated chaos injection scripts</li>
<li>Grafana dashboard configurations</li>
</ul>

<p><strong>GitHub</strong>: <a href="https://github.com/builtbychikara/WhatIfSeries/tree/main/what-if/kafka-vs-rest-polling" rel="noopener noreferrer">https://github.com/builtbychikara/WhatIfSeries/tree/main/what-if/kafka-vs-rest-polling</a></p>

<p><strong>Key Files</strong>:</p>

<ul>
<li>
<code>kafka-pubsub-service/consumer.js</code> - Circuit breaker implementation</li>
<li>
<code>k6/runner.js</code> - Chaos engineering experiment orchestration</li>
<li>
<code>docker-compose.yml</code> - Complete infrastructure stack</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>Event-driven architectures using Kafka demonstrate measurably superior resilience during database failures, with zero client-facing errors compared to 50% error rates in synchronous REST implementations. However, this resilience comes at the cost of operational complexity and consistency guarantees.</p>

<p>The choice between REST and Kafka should be driven by quantified requirements for resilience, throughput, and acceptable consistency models, not by architectural preferences or industry trends.</p>

<p><strong>Acknowledgments</strong>: Built in collaboration with Aakanksh Singh to demonstrate production-grade system design patterns through empirical measurement.</p>

<p><strong>Tech Stack</strong>: Node.js, Apache Kafka, PostgreSQL, k6, Prometheus, Grafana, Docker</p>

