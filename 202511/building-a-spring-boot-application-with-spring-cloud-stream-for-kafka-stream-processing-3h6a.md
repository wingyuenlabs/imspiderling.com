---
Title: Building a Spring Boot Application with Spring Cloud Stream for Kafka Stream Processing
Description: 
Author: Khalid Edaoudi
Date: 2025-11-23T22:01:54.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  1. Download Kafka
</h2>

<p>Download the version <strong>2.13-2.7.0</strong> of Kafka (<a href="https://kafka.apache.org/downloads" rel="noopener noreferrer">https://kafka.apache.org/downloads</a>)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvqqed8ml6x5na9y82kzv.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvqqed8ml6x5na9y82kzv.jpg" alt=" " width="800" height="407"></a></p>

<h2>
  
  
  2. Start Kafka
</h2>

<p>To start Kafka we should first execute <strong>Zookeeper</strong> (It handles kafka instances in our application).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm8rdigdhw88iqr5ckajr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm8rdigdhw88iqr5ckajr.png" alt=" " width="800" height="441"></a></p>

<p>we add start command to open the output terminal in an new prompt, <code>bin\windows\zookeeper-server.start.bat</code> is the script file to run zookeeper  based on configuration file <code>config/zookeeper.properties</code>, if you have Linux operating system, the zookeeper script is in <code>bin\zookeeper-server.start.sh</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5t2mm4pqbhgl39put7kz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5t2mm4pqbhgl39put7kz.png" alt=" " width="800" height="462"></a></p>

<p>So we have now zookeeper running on port <code>2181</code>.</p>

<p>By the same way we start <strong>Kafka</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgeqrql5al3sl9v6tjfmm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgeqrql5al3sl9v6tjfmm.png" alt=" " width="800" height="440"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6va97cou44fgl3y97lv0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6va97cou44fgl3y97lv0.png" alt=" " width="800" height="462"></a></p>

<p>Sometimes Kafka won't be running because <em>wmic</em> (<strong>Windows Management Instrumentation Command-line</strong>) is missing, it used to check JAVA_HOME and java version, to fix that open <code>kafka-start-server.bat</code> file in your editor and replace the whole content by the following code :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>@echo off
rem Licensed to the Apache Software Foundation <span class="o">(</span>ASF<span class="o">)</span> under one or more
rem contributor license agreements.  See the NOTICE file distributed with
rem this work <span class="k">for </span>additional information regarding copyright ownership.
rem The ASF licenses this file to You under the Apache License, Version 2.0
rem <span class="o">(</span>the <span class="s2">"License"</span><span class="o">)</span><span class="p">;</span> you may not use this file except <span class="k">in </span>compliance with
rem the License.  You may obtain a copy of the License at
rem
rem     http://www.apache.org/licenses/LICENSE-2.0
rem
rem Unless required by applicable law or agreed to <span class="k">in </span>writing, software
rem distributed under the License is distributed on an <span class="s2">"AS IS"</span> BASIS,
rem WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
rem See the License <span class="k">for </span>the specific language governing permissions and
rem limitations under the License.


IF <span class="o">[</span>%1] EQU <span class="o">[]</span> <span class="o">(</span>
    <span class="nb">echo </span>USAGE: %0 server.properties
    EXIT /B 1
<span class="o">)</span>

SetLocal
IF <span class="o">[</span><span class="s2">"%KAFKA_LOG4J_OPTS%"</span><span class="o">]</span> EQU <span class="o">[</span><span class="s2">""</span><span class="o">]</span> <span class="o">(</span>
    <span class="nb">set </span><span class="nv">KAFKA_LOG4J_OPTS</span><span class="o">=</span><span class="nt">-Dlog4j</span>.configuration<span class="o">=</span>file:%~dp0../../config/log4j.properties
<span class="o">)</span>

REM <span class="nt">---</span> Force 64-bit heap, supprime la détection avec wmic <span class="nt">---</span>
IF <span class="o">[</span><span class="s2">"%KAFKA_HEAP_OPTS%"</span><span class="o">]</span> EQU <span class="o">[</span><span class="s2">""</span><span class="o">]</span> <span class="o">(</span>
    <span class="nb">set </span><span class="nv">KAFKA_HEAP_OPTS</span><span class="o">=</span><span class="nt">-Xmx1G</span> <span class="nt">-Xms1G</span>
<span class="o">)</span>

<span class="s2">"%~dp0kafka-run-class.bat"</span> kafka.Kafka %<span class="k">*</span>
EndLocal
</code></pre>

</div>



<p>Then you would have Kafka running on port <code>9092</code>.</p>




<h2>
  
  
  3. kafla-console-producer &amp; kafka-console-consumer
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm4jvq3fx7oqwmvbb7ufg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm4jvq3fx7oqwmvbb7ufg.png" alt=" " width="800" height="276"></a></p>

<p><code>kafla-console-producer</code> and <code>kafka-console-consumer</code> are kafka console clients to test kafka on the console, let's try that.</p>

<p>first, we start the consumer to listen to a topic of name <strong>R1</strong> using <code>bin\windows\kafka-console-consumer.bat</code> script.</p>

<p>We mention the address of a running kafka instance using <code>--bootstrap-server</code> then the name of the topic to listen to.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F60hqwxuwpz965jfqqla1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F60hqwxuwpz965jfqqla1.png" alt=" " width="800" height="367"></a></p>

<p>Now we have a consumer which is subscribed to the topic <strong>R1</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6so5o6lm77vp9t6a735x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6so5o6lm77vp9t6a735x.png" alt=" " width="800" height="466"></a></p>

<p>Let's create a producer using <code>bin\windows\kafka-console-producer.bat</code> script:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvmrqh32qhjhx6t81qbwc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvmrqh32qhjhx6t81qbwc.png" alt=" " width="800" height="354"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa7khc3id9nq6nzeceos7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa7khc3id9nq6nzeceos7.png" alt=" " width="800" height="462"></a></p>

<p>On the left, the producer is sending the message "<em>kafka is great</em>" to a Kafka <strong>R1</strong> topic, while on the right, the consumer receives the same message in real-time. This illustrates Kafka’s core messaging capability: reliable, real-time communication between distributed systems.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnnigykopen6r1wfhy42h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnnigykopen6r1wfhy42h.png" alt=" " width="800" height="213"></a><br>
Documentation : <a href="https://kafka.apache.org/documentation/#quickstart" rel="noopener noreferrer">https://kafka.apache.org/documentation/#quickstart</a></p>
<h2>
  
  
  3. Create Java Application
</h2>

<p>We're going to create a <strong>Spring Boot</strong> project using <em>spring initialzr</em><br>
We will use Spring boot version <strong>3.5.8</strong> with jar Packaging and <strong>Java 17</strong> and choosing <strong>Maven</strong> as our dependency management system.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0on6iuqgq2rty1pqq8ae.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0on6iuqgq2rty1pqq8ae.png" alt=" " width="800" height="788"></a></p>

<p>As a dependencies add the followings :</p>

<ul>
<li><strong>Lombok</strong></li>
<li><strong>Spring Web</strong></li>
<li><strong>Spring for Apache Kafka</strong></li>
<li><strong>Spring for Apache Kafka Streams</strong></li>
<li><strong>Cloud Stream</strong></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9jzo7g5ff27o2natxvpa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9jzo7g5ff27o2natxvpa.png" alt=" " width="800" height="673"></a></p>

<p>Then click on <strong>Generate</strong>, unzip the file and open the project on your favorite Java Editor (I recommend <strong>Intellij IDEA</strong> which is the editor I used in this demo).</p>
<h2>
  
  
  4. Create The PageEvent Entity
</h2>

<p>inside the main package create another package <code>entities</code> then create class named <code>PageEvent.java</code></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz4lztaf0iy1rss6ol91y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz4lztaf0iy1rss6ol91y.png" alt=" " width="418" height="546"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.example.kafka.entities</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">lombok.*</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.Date</span><span class="o">;</span>

<span class="nd">@Getter</span> <span class="nd">@Setter</span> <span class="nd">@AllArgsConstructor</span> <span class="nd">@NoArgsConstructor</span> <span class="nd">@ToString</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">PageEvent</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">name</span><span class="o">;</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">user</span><span class="o">;</span>
    <span class="kd">private</span> <span class="nc">Date</span> <span class="n">date</span><span class="o">;</span>
    <span class="kd">private</span> <span class="kt">long</span> <span class="n">duration</span><span class="o">;</span>
<span class="o">}</span>
</code></pre>

</div>



<p><strong>ATTENTION</strong> : Lombok may not be working in recent versions of spring boot, to fix that we specify an older lombok version explicitly in <code>pom.xml</code> file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;dependency&gt;</span>
    <span class="nt">&lt;groupId&gt;</span>org.projectlombok<span class="nt">&lt;/groupId&gt;</span>
    <span class="nt">&lt;artifactId&gt;</span>lombok<span class="nt">&lt;/artifactId&gt;</span>
    <span class="nt">&lt;version&gt;</span>1.18.32<span class="nt">&lt;/version&gt;</span>
    <span class="nt">&lt;optional&gt;</span>true<span class="nt">&lt;/optional&gt;</span>
<span class="nt">&lt;/dependency&gt;</span>
</code></pre>

</div>



<p>And also in the plugins section.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;configuration&gt;</span>
    <span class="nt">&lt;annotationProcessorPaths&gt;</span>
        <span class="nt">&lt;path&gt;</span>
        <span class="nt">&lt;groupId&gt;</span>org.projectlombok<span class="nt">&lt;/groupId&gt;</span>
        <span class="nt">&lt;artifactId&gt;</span>lombok<span class="nt">&lt;/artifactId&gt;</span>
        <span class="nt">&lt;version&gt;</span>1.18.32<span class="nt">&lt;/version&gt;</span>
    <span class="nt">&lt;/path&gt;</span>
    <span class="nt">&lt;/annotationProcessorPaths&gt;</span>
<span class="nt">&lt;/configuration&gt;</span>
</code></pre>

</div>



<h2>
  
  
  5. Create PageEventRestController
</h2>

<p>Create <code>PageEventRestController.java</code> Class in a new package <code>web</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpkp9r9pzcfsa9k14wb81.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpkp9r9pzcfsa9k14wb81.png" alt=" " width="412" height="608"></a></p>

<p>This REST controller exposes an endpoint that publishes PageEvent messages to any Kafka topic. The key part is <code>StreamBridge</code>, a Spring Cloud Stream utility that lets you send messages programmatically without binding the controller to a specific producer function. When the <code>/publish/{topic}/{name}</code> endpoint is called, the controller creates a new PageEvent object, then uses <code>streamBridge.send(topic, pageEvent)</code> to dynamically route the event to the given Kafka topic. This makes it easy to produce messages on demand, especially for testing or building flexible producer APIs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.example.kafka.web</span><span class="o">;</span>


<span class="kn">import</span> <span class="nn">com.example.kafka.entities.PageEvent</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.beans.factory.annotation.Autowired</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.cloud.stream.function.StreamBridge</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.web.bind.annotation.GetMapping</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.web.bind.annotation.PathVariable</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.web.bind.annotation.RestController</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.Date</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">java.util.Random</span><span class="o">;</span>

<span class="nd">@RestController</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">PageEventRestController</span> <span class="o">{</span>

    <span class="nd">@Autowired</span>
    <span class="kd">private</span> <span class="nc">StreamBridge</span> <span class="n">streamBridge</span><span class="o">;</span>

    <span class="nd">@GetMapping</span><span class="o">(</span><span class="s">"/publish/{topic}/{name}"</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">PageEvent</span> <span class="nf">publish</span><span class="o">(</span><span class="nd">@PathVariable</span> <span class="nc">String</span> <span class="n">topic</span><span class="o">,</span> <span class="nd">@PathVariable</span> <span class="nc">String</span> <span class="n">name</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">PageEvent</span> <span class="n">pageEvent</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">PageEvent</span><span class="o">(</span><span class="n">name</span><span class="o">,</span><span class="nc">Math</span><span class="o">.</span><span class="na">random</span><span class="o">()</span> <span class="o">&gt;</span> <span class="mf">0.5</span><span class="o">?</span><span class="s">"U1"</span><span class="o">:</span><span class="s">"U2"</span><span class="o">,</span><span class="k">new</span> <span class="nc">Date</span><span class="o">(),</span><span class="k">new</span> <span class="nc">Random</span><span class="o">().</span><span class="na">nextInt</span><span class="o">(</span><span class="mi">9000</span><span class="o">));</span>
        <span class="n">streamBridge</span><span class="o">.</span><span class="na">send</span><span class="o">(</span><span class="n">topic</span><span class="o">,</span>
                <span class="n">pageEvent</span><span class="o">);</span>
        <span class="k">return</span> <span class="n">pageEvent</span><span class="o">;</span>
    <span class="o">}</span>

<span class="o">}</span>
</code></pre>

</div>



<p>Then You run the <code>KafkaApplication</code>class.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F290d372ao82lpbzl8vre.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F290d372ao82lpbzl8vre.png" alt=" " width="800" height="264"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fixbqpqkw9yfbu0a6k12i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fixbqpqkw9yfbu0a6k12i.png" alt=" " width="800" height="409"></a></p>

<p>Now you open a <strong>consumer console</strong> subscribed to topic <strong>R1</strong> and test the publish controller end-point which represent our producer.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs6oh636sxzctp1h2tzyv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs6oh636sxzctp1h2tzyv.png" alt=" " width="800" height="441"></a></p>

<p>the producer publishes a message and the consumer receives it instantly, confirming that everything is working correctly.</p>

<h2>
  
  
  6. Create PageEventConsumer
</h2>

<p>Now we will create a consumer inside our Spring Boot application, you name it <code>PageEventService.java</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedgonyih3worwh3xcese.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedgonyih3worwh3xcese.png" alt=" " width="412" height="752"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kn">package</span> <span class="nn">com.example.kafka.services</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">com.example.kafka.entities.PageEvent</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.context.annotation.Bean</span><span class="o">;</span>
<span class="kn">import</span> <span class="nn">org.springframework.stereotype.Service</span><span class="o">;</span>

<span class="kn">import</span> <span class="nn">java.util.function.Consumer</span><span class="o">;</span>

<span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">PageEventService</span> <span class="o">{</span>

    <span class="nd">@Bean</span>
    <span class="kd">public</span> <span class="nc">Consumer</span><span class="o">&lt;</span><span class="nc">PageEvent</span><span class="o">&gt;</span> <span class="nf">pageEventConsumer</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="o">(</span><span class="n">input</span><span class="o">)</span> <span class="o">-&gt;</span> <span class="o">{</span>
            <span class="nc">System</span><span class="o">.</span><span class="na">out</span><span class="o">.</span><span class="na">println</span><span class="o">(</span><span class="s">"**************************************"</span><span class="o">);</span>
            <span class="nc">System</span><span class="o">.</span><span class="na">out</span><span class="o">.</span><span class="na">println</span><span class="o">(</span><span class="n">input</span><span class="o">.</span><span class="na">toString</span><span class="o">());</span>
            <span class="nc">System</span><span class="o">.</span><span class="na">out</span><span class="o">.</span><span class="na">println</span><span class="o">(</span><span class="s">"**************************************"</span><span class="o">);</span>
        <span class="o">};</span>
    <span class="o">}</span>

<span class="o">}</span>
</code></pre>

</div>



<p>This class defines a Kafka consumer using <strong>Spring Cloud Stream’s functional programming model</strong>.<br>
Spring Cloud Stream allows you to create message-driven microservices by exposing functions like <code>Consumer&lt;T&gt;</code> that automatically bind to Kafka topics.</p>

<p>In this example, the <strong>PageEventService</strong> declares a <code>Consumer&lt;PageEvent&gt;</code> bean named <code>pageEventConsumer</code>.<br>
When a PageEvent message arrives from Kafka, Spring Cloud Stream passes it to this function. The consumer then prints the event details between separators, making it easy to verify that the message was received and processed.</p>

<p>This setup provides a clean, minimal way to build real-time Kafka consumers without dealing directly with low-level Kafka APIs—Spring Cloud Stream handles the boilerplate, letting you focus on processing your data.</p>

<p>Then open your <code>application.properties</code> (or <code>application.yml</code>) file and put those properties<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="py">spring.cloud.function.definition</span><span class="p">=</span><span class="s">pageEventConsumer</span>
<span class="py">spring.cloud.stream.bindings.pageEventConsumer-in-0.destination</span><span class="p">=</span><span class="s">R1</span>
<span class="py">spring.cloud.stream.bindings.pageEventConsumer-in-0.group</span><span class="p">=</span><span class="s">group1</span>
</code></pre>

</div>



<p>This configuration tells Spring Cloud Stream which function should be activated, which Kafka topic it should listen to, and which consumer group it belongs to.</p>

<ul>
<li><p><code>spring.cloud.function.definition=pageEventConsumer</code><br>
This selects the functional bean named pageEventConsumer as the active consumer function. Spring Cloud Stream will bind this function to Kafka automatically.</p></li>
<li><p><code>spring.cloud.stream.bindings.pageEventConsumer-in-0.destination=R1</code><br>
This maps the input channel of the function to the Kafka topic R1.<br>
Whenever a message is published to the R1 topic, it will be delivered to pageEventConsumer.</p></li>
<li><p><code>spring.cloud.stream.bindings.pageEventConsumer-in-0.group=group1</code><br>
This assigns the consumer to the Kafka consumer group named group1.</p></li>
</ul>

<p>Now you run the application and you test (tap <em>localhost:8080/publish/R1/blog</em>)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frevhavz2cgqm0adif0pg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frevhavz2cgqm0adif0pg.png" alt=" " width="800" height="312"></a></p>

<h2>
  
  
  7. Create Producer Poller
</h2>

<p>In Spring Cloud Stream, a <strong>producer poller</strong> is a mechanism that automatically triggers a <strong>Supplier function</strong> at a fixed interval so it can generate and publish new messages to a topic without needing an external event.<br>
Instead of manually calling the producer or waiting for an HTTP request, the poller acts like a timer: every few milliseconds or seconds, it executes the Supplier, collects the returned object, and sends it to the configured message channel<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code> <span class="nd">@Bean</span>
    <span class="kd">public</span> <span class="nc">Supplier</span><span class="o">&lt;</span><span class="nc">PageEvent</span><span class="o">&gt;</span> <span class="nf">pageEventSupplier</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">return</span> <span class="o">()</span> <span class="o">-&gt;</span> <span class="k">new</span> <span class="nc">PageEvent</span><span class="o">(</span>
                <span class="nc">Math</span><span class="o">.</span><span class="na">random</span><span class="o">()</span> <span class="o">&gt;</span> <span class="mf">0.5</span><span class="o">?</span><span class="s">"P1"</span><span class="o">:</span><span class="s">"P2"</span><span class="o">,</span>
                <span class="nc">Math</span><span class="o">.</span><span class="na">random</span><span class="o">()</span> <span class="o">&gt;</span> <span class="mf">0.5</span><span class="o">?</span><span class="s">"U1"</span><span class="o">:</span><span class="s">"U2"</span><span class="o">,</span>
                <span class="k">new</span> <span class="nf">Date</span><span class="o">(),</span><span class="k">new</span> <span class="nc">Random</span><span class="o">().</span><span class="na">nextInt</span><span class="o">(</span><span class="mi">9000</span><span class="o">));</span>
    <span class="o">}</span>
</code></pre>

</div>



<p>In appilication.properties file I set <code>spring.cloud.stream.poller.fixed-delay</code> property to <strong>100ms</strong>, that mean 100ms is the time interval between each new message.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="py">spring.cloud.function.definition</span><span class="p">=</span><span class="s">pageEventConsumer;pageEventSupplier</span>

<span class="py">spring.cloud.stream.bindings.pageEventConsumer-in-0.destination</span><span class="p">=</span><span class="s">R1</span>
<span class="py">spring.cloud.stream.bindings.pageEventConsumer-in-0.group</span><span class="p">=</span><span class="s">group1</span>

<span class="py">spring.cloud.stream.bindings.pageEventSupplier-out-0.destination</span><span class="p">=</span><span class="s">R2</span>
<span class="py">spring.cloud.stream.bindings.pageEventSupplier-out-0.group</span><span class="p">=</span><span class="s">group2</span>
<span class="py">spring.cloud.stream.poller.fixed-delay</span><span class="p">=</span><span class="s">100</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqbhhmwu45epk9a5i3qko.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqbhhmwu45epk9a5i3qko.png" alt=" " width="800" height="464"></a></p>

<h2>
  
  
  8. Summary
</h2>

<p>In this tutorial, we explored the essential building blocks of using Kafka with Spring Boot and Spring Cloud Stream, starting from installing Kafka locally, testing message flow with console producers and consumers, and finally building a real-time event streaming application in Java.</p>

<p>You learned how to:</p>

<ul>
<li><p>Install and run <strong>Zookeeper</strong> and <strong>Kafka</strong> on your machine</p></li>
<li><p>Use <strong>kafka-console-producer</strong> and <strong>kafka-console-consumer</strong> to test topics manually</p></li>
<li><p>Build a REST producer using <strong>StreamBridge</strong></p></li>
<li><p>Implement a Kafka consumer using <strong>Spring Cloud Stream's functional model</strong></p></li>
<li><p>Create a <strong>Producer Poller</strong> with a Supplier to automatically publish messages on a fixed schedule</p></li>
</ul>

<p>Together, these components demonstrate how powerful and simple it can be to create event-driven microservices using Spring Cloud Stream. With only a few lines of configuration and functional beans, your application becomes capable of consuming, producing, transforming, and streaming data in real time all without the complexity of low-level Kafka APIs.</p>

<p>From here, you can extend your project by adding Kafka Streams, function compositions, message filtering, windowing, or even advanced analytics pipelines. Event-driven architectures open the door to creating scalable, reactive systems that handle data as it flows.</p>

