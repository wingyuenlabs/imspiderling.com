---
Title: Introduction to the Confluent REST Proxy
Description: 
Author: Aisalkyn Aidarova
Date: 2025-11-20T21:39:04.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  <strong>1. Why Do We Need the REST Proxy?</strong>
</h1>

<p>Kafka works best with Java (JVM-based languages).<br>
Kafka has clients for other languages too:</p>

<ul>
<li>Node.js</li>
<li>Go</li>
<li>Python</li>
<li>C#</li>
<li>Rust</li>
<li>PHP</li>
</ul>

<p>But these clients <strong>do not always support Avro well</strong>.<br>
Some of them:</p>

<ul>
<li>cannot register schemas</li>
<li>cannot handle binary Avro easily</li>
<li>do not support schema evolution</li>
<li>require many libraries</li>
</ul>
<h3>
  
  
  The solution:
</h3>
<h2>
  
  
  ✔️ <strong>Confluent REST Proxy</strong>
</h2>

<p>It allows any application to use <strong>simple HTTP requests</strong> (POST, GET) to send or read Kafka messages <strong>without writing Kafka code</strong>.</p>

<p>Every language can make HTTP requests, so every language can talk to Kafka through the REST Proxy.</p>


<h1>
  
  
  <strong>2. Where Does the REST Proxy Fit?</strong>
</h1>

<p>Here is your normal system:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[ Java Producer ] → Kafka Cluster + Schema Registry → [ Java Consumer ]
</code></pre>

</div>



<p>Now we introduce REST Proxy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[ Any Language Producer ] → HTTP → REST Proxy → Kafka → Schema Registry
</code></pre>

</div>



<p>And for consumers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[ Any Language Consumer ] ← HTTP ← REST Proxy ← Kafka
</code></pre>

</div>



<p>The REST Proxy communicates with:</p>

<ul>
<li>Kafka brokers</li>
<li>Schema Registry</li>
</ul>

<p>It does the “Kafka work” for you.</p>




<h1>
  
  
  <strong>3. What Does REST Proxy Do?</strong>
</h1>

<p>The REST Proxy acts as a <strong>translator</strong>.</p>

<p>It allows you to:</p>

<h3>
  
  
  ✔ Produce messages to Kafka via HTTP POST
</h3>

<h3>
  
  
  ✔ Consume messages from Kafka via HTTP GET
</h3>

<h3>
  
  
  ✔ Use Avro, JSON, or raw bytes
</h3>

<h3>
  
  
  ✔ Automatically register schemas
</h3>

<h3>
  
  
  ✔ Automatically validate schemas
</h3>

<h3>
  
  
  ✔ Easily integrate with languages that have poor Kafka libraries
</h3>

<p>It simplifies everything.</p>




<h1>
  
  
  <strong>4. Why REST Proxy Is Useful</strong>
</h1>

<h3>
  
  
  ✔ Works with any programming language
</h3>

<p>All languages support HTTP calls.</p>

<h3>
  
  
  ✔ Easy to produce Avro
</h3>

<p>Even if the language has no Avro serializer.</p>

<h3>
  
  
  ✔ Automatically works with Schema Registry
</h3>

<p>Schema Registry is fully integrated.</p>

<h3>
  
  
  ✔ Easy debugging
</h3>

<p>HTTP requests are easier to test than Kafka clients.</p>




<h1>
  
  
  <strong>5. What Are the Downsides?</strong>
</h1>

<h3>
  
  
  ❌ It is slower than direct Kafka clients
</h3>

<p>Why?</p>

<p>Because:</p>

<ul>
<li>HTTP is slower than the Kafka wire protocol</li>
<li>REST Proxy does extra work</li>
<li>Messages are serialized and validated through REST Proxy first</li>
</ul>

<p><strong>Performance cost:</strong><br>
<strong>3x–4x slower than normal Kafka clients</strong></p>

<p>For most use cases this is okay, but high-performance systems should use Kafka clients directly.</p>
<h3>
  
  
  ❌ Requires batching on the application side
</h3>

<p>The REST Proxy will not batch automatically.<br>
If you want high throughput, your application must batch multiple messages together before sending.</p>


<h1>
  
  
  <strong>6. REST Proxy Is Already in Your Docker Setup</strong>
</h1>

<p>Good news:</p>

<p>If you're using Confluent’s Docker Compose, the REST Proxy is already available at a port like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>http://localhost:8082
</code></pre>

</div>



<p>So you can start using it immediately.</p>




<h1>
  
  
  <strong>7. What We Will Learn in This Section</strong>
</h1>

<p>In this REST Proxy module you will learn:</p>

<h3>
  
  
  🔹 <strong>How REST Proxy works internally</strong>
</h3>

<ul>
<li>Supported endpoints</li>
<li>Supported API versions</li>
<li>How messages are formatted</li>
</ul>

<h3>
  
  
  🔹 <strong>How to create topics via REST Proxy</strong>
</h3>

<h3>
  
  
  🔹 <strong>How to produce messages</strong>
</h3>

<ul>
<li>In binary</li>
<li>In JSON</li>
<li>In Avro</li>
</ul>

<h3>
  
  
  🔹 <strong>How to consume messages</strong>
</h3>

<ul>
<li>With different formats</li>
<li>With schemas</li>
<li>With offsets</li>
</ul>

<h3>
  
  
  🔹 <strong>How to deploy and scale the REST Proxy</strong>
</h3>

<ul>
<li>Production tips</li>
<li>High availability</li>
<li>Load balancing</li>
</ul>

<h2>
  
  
  1. V1 vs V2 – Which REST Proxy API Should We Use?
</h2>

<h3>
  
  
  1.1 Background: Old Kafka APIs vs New Kafka APIs
</h3>

<p>A long time ago, Kafka had:</p>

<ul>
<li>an <strong>old producer API</strong>
</li>
<li>an <strong>old consumer API</strong>
</li>
</ul>

<p>These were used in Kafka <strong>before version 0.8</strong>.</p>

<ul>
<li>The <strong>old consumer</strong> stored offsets in <strong>Zookeeper</strong>.</li>
<li>Then Kafka introduced a <strong>new consumer</strong> and <strong>new producer</strong> that store offsets <strong>in Kafka itself</strong>, not Zookeeper.</li>
</ul>

<p>All modern code and all modern courses (including yours) use the <strong>new producer/consumer APIs</strong>.</p>




<h3>
  
  
  1.2 How REST Proxy fits into this history
</h3>

<p>The <strong>REST Proxy</strong> has existed for a long time too.</p>

<ul>
<li>Originally, it was built on top of the <strong>old</strong> Kafka consumer/producer APIs.
That version was called <strong>V1</strong> in the REST Proxy.</li>
<li>Later, the REST Proxy was updated to use the <strong>new</strong> Kafka consumer/producer APIs.
That new version is called <strong>V2</strong> in the REST Proxy.</li>
</ul>

<p>So:</p>

<ul>
<li>
<strong>V1 REST Proxy API</strong> → old Kafka APIs</li>
<li>
<strong>V2 REST Proxy API</strong> → new Kafka APIs (what we use today)</li>
</ul>




<h3>
  
  
  1.3 Which one should you use?
</h3>

<p>This part is very simple:</p>

<blockquote>
<p><strong>Always use V2.</strong><br>
Ignore V1.<br>
If you see <code>.../v1/...</code> in examples – that is legacy.</p>
</blockquote>

<ul>
<li>V1 is old and will eventually disappear.</li>
<li>V2 has better performance.</li>
<li>V2 matches everything you already know about Kafka.</li>
</ul>

<p>So the rule for your students:</p>

<blockquote>
<p>“If the URL says <code>/v1/</code> – don’t use it.<br>
In this course and in real projects, we always use <code>/v2/</code> REST Proxy APIs.”</p>
</blockquote>




<h2>
  
  
  2. REST Proxy Content-Type Header – How to Build It
</h2>

<p>When you send an HTTP request to the REST Proxy, you must set a <strong>Content-Type</strong> header that tells the proxy:</p>

<ul>
<li>what kind of Kafka data you’re sending (Avro, JSON, binary)</li>
<li>which API version you’re using (V2)</li>
<li>how the HTTP body itself is encoded (JSON)</li>
</ul>

<p>There is a <strong>pattern</strong> for the Content-Type.</p>

<p>Let’s break it into 4 parts.</p>




<h3>
  
  
  2.1 The 4 Parts of the Content-Type
</h3>

<p>General pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>application/vnd.kafka.&lt;embedded-format&gt;.v2+json
</code></pre>

</div>



<p>Let’s understand each piece:</p>

<ol>
<li><p><code>application</code><br>
– normal MIME type prefix.</p></li>
<li><p><code>vnd.kafka</code><br>
– means “Kafka-specific vendor type”.</p></li>
<li><p><code>&lt;embedded-format&gt;</code><br>
– this tells REST Proxy what <strong>Kafka message format</strong> you are using:</p></li>
</ol>

<ul>
<li><code>json</code></li>
<li><code>binary</code></li>
<li><code>avro</code></li>
</ul>

<ol>
<li><p><code>.v2</code><br>
– REST Proxy API version. We use <strong>v2</strong> only.</p></li>
<li><p><code>+json</code><br>
– this tells REST Proxy that the <strong>HTTP body</strong> is encoded as JSON text.</p></li>
</ol>




<h3>
  
  
  2.2 Some concrete examples
</h3>

<h4>
  
  
  Example 1: JSON messages
</h4>

<p>You want to send <strong>JSON messages</strong> to Kafka via V2:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight http"><code><span class="err">Content-Type: application/vnd.kafka.json.v2+json
</span></code></pre>

</div>



<ul>
<li>Kafka message format: <code>json</code>
</li>
<li>REST Proxy API version: <code>v2</code>
</li>
<li>HTTP body format: JSON (<code>+json</code>)</li>
</ul>




<h4>
  
  
  Example 2: Avro messages
</h4>

<p>You want to send <strong>Avro messages</strong> to Kafka via V2:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight http"><code><span class="err">Content-Type: application/vnd.kafka.avro.v2+json
</span></code></pre>

</div>



<ul>
<li>Kafka message format: <code>avro</code>
</li>
<li>REST Proxy version: <code>v2</code>
</li>
<li>HTTP body format: JSON (<code>+json</code>)</li>
</ul>

<p>The <em>payload</em> is described in JSON, but REST Proxy converts it to <strong>Avro</strong> using Schema Registry.</p>




<h4>
  
  
  Example 3: Binary messages
</h4>

<p>You want to send <strong>raw binary</strong> data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight http"><code><span class="err">Content-Type: application/vnd.kafka.binary.v2+json
</span></code></pre>

</div>






<h3>
  
  
  2.3 Easy way to remember
</h3>

<p>Just memorize this template:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>application/vnd.kafka.&lt;json|avro|binary&gt;.v2+json
</code></pre>

</div>



<p>And remember:</p>

<ul>
<li>Always use <code>.v2</code> → because we use V2 API.</li>
<li>Always end with <code>+json</code> → because the HTTP request body is JSON.</li>
<li>Change only the <code>&lt;embedded-format&gt;</code> to: <code>json</code>, <code>avro</code>, or <code>binary</code>.</li>
</ul>




<h2>
  
  
  3. Quick Summary
</h2>

<ul>
<li>Kafka used to have old APIs; REST Proxy V1 was built on top of those.</li>
<li>New Kafka APIs → REST Proxy V2 → this is what we use today.</li>
<li><strong>Always use V2 REST Proxy endpoints.</strong></li>
<li>REST Proxy Content-Type follows a pattern:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  application/vnd.kafka.&lt;embedded-format&gt;.v2+json
</code></pre>

</div>



<p>where <code>&lt;embedded-format&gt;</code> = <code>json</code>, <code>avro</code>, or <code>binary</code>.</p>

