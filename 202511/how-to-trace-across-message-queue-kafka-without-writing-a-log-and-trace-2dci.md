---
Title: How to trace across message queue - kafka, without writing a log and trace
Description: 
Author: FrankNPC
Date: 2025-11-20T21:48:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>Once the system sent messages through the queue, or other asynchronized service, as part of business logic, it's hard to trace who, when and where do they go to and, who, when and where did they come from. </p>

<p>Now we have bitryon logger to connect puzzles together into workflow and stack-trace. Following <a href="https://dev.to/franknpc/how-to-automate-log-and-trace-without-writing-a-log-and-trace-2a0n">how to automate log and trace without writing a log and trace</a>, we can configure and initiate bitryon logger with spring to cover more traces with logs.</p>

<p>Take Kafka as an example. Supposedly we had kafka server installed already. </p>

<p>Here is a taste of the trace logs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>2025-11-20 15:42:24.785|http-nio-80-exec-1#38|7G5HPZyA3QWSF2f1SMRN5qV2U95tjOLi|4|JSON|
MedicService.java#io.bitryon.example.web.service.MedicService#callSelfInvocation#68#|
[{
    "testString": "68ssTfeP43IeSVqFWx1jH1VigFuEdUbt"
}]
2025-11-20 15:42:24.787|http-nio-80-exec-1#38|7G5HPZyA3QWSF2f1SMRN5qV2U95tjOLi|13|JSON|
MedicService.java#io.bitryon.example.web.service.MedicService#callSelfInvocation#68#R|
["1763653344787"]
2025-11-20 15:42:26.073|org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1#51|7G5HPZyA3QWSF2f1SMRN5qV2U95tjOLi|14-1|JSON|
UserService.java#io.bitryon.example.web.service.rpc.UserService#getBySessionId#0#|
[{
    "sessionId": "68ssTfeP43IeSVqFWx1jH1VigFuEdUbt"
}]
2025-11-20 15:42:26.693|http-nio-80-exec-4#41|7G5HPZyA3QWSF2f1SMRN5qV2U95tjOLi|14-1-2|JSON|
UserServiceImpl.java#io.bitryon.example.web.service.rpc.UserServiceImpl#getBySessionId#45#|
[{
    "sessionId": "68ssTfeP43IeSVqFWx1jH1VigFuEdUbt"
}]
2025-11-20 15:42:26.694|http-nio-80-exec-4#41|7G5HPZyA3QWSF2f1SMRN5qV2U95tjOLi|14-1-3|JSON|
UserServiceImpl.java#io.bitryon.example.web.service.rpc.UserServiceImpl#getBySessionId#45#R|
[{
    "name": "ra***ly",
    "id": 456,
    "driverLisenceId": "****************************",
    "age": null
}]
2025-11-20 15:42:26.918|org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1#51|7G5HPZyA3QWSF2f1SMRN5qV2U95tjOLi|14-2|JSON|
UserService.java#io.bitryon.example.web.service.rpc.UserService#getBySessionId#0#R|
[{
    "name": "ra***ly",
    "id": 456,
    "driverLisenceId": "****************************",
    "age": null
}]
</code></pre>

</div>



<p><em>Both, bitryon logger is a tracer; bitryon logger is a logger;</em></p>

<h3>
  
  
  Steps to trace everywhere
</h3>

<ol>
<li>Write next step log id to metadata/header of the message</li>
<li>Read the step log id from metadata/header of the message</li>
<li>Configure beans for metadata/header Interceptors</li>
<li>Send and receive messages through the queue, in case of kafka.</li>
<li>Upload the logs to web portal and access log traces</li>
</ol>

<p><em>Now, show you my code</em></p>

<h3>
  
  
  Write next step log id to metadata/header of the message
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">ProducerLoggerInterceptor</span> <span class="kd">implements</span> <span class="nc">ProducerInterceptor</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="kd">static</span> <span class="kd">final</span> <span class="nc">Logger</span> <span class="n">logger</span> <span class="o">=</span> <span class="nc">LoggerFactory</span><span class="o">.</span><span class="na">getLogger</span><span class="o">();</span>

    <span class="nd">@Override</span>
    <span class="kd">public</span> <span class="nc">ProducerRecord</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="nf">onSend</span><span class="o">(</span><span class="nc">ProducerRecord</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="n">record</span><span class="o">)</span> <span class="o">{</span>
        <span class="c1">// CRITICAL .getNextStepLogId()</span>
        <span class="nc">String</span> <span class="n">nextSteplogId</span> <span class="o">=</span> <span class="n">logger</span><span class="o">.</span><span class="na">getNextStepLogId</span><span class="o">();</span>
        <span class="n">record</span><span class="o">.</span><span class="na">headers</span><span class="o">().</span><span class="na">add</span><span class="o">(</span><span class="nc">PreDefinition</span><span class="o">.</span><span class="na">HTTP_HEADER_STEP_LOG_ID</span><span class="o">,</span> <span class="n">nextSteplogId</span><span class="o">.</span><span class="na">getBytes</span><span class="o">(</span><span class="nc">PreDefinition</span><span class="o">.</span><span class="na">CharsetEncoding</span><span class="o">));</span>
        <span class="k">return</span> <span class="n">record</span><span class="o">;</span>
    <span class="o">}</span>
<span class="c1">// other code</span>
<span class="o">}</span>
</code></pre>

</div>



<p>The difference between service and message queue to use step log id to connect the trace, is <code>getStepLogId</code> VS <code>getNextStepLogId</code>: trace inside the service invoke VS start a new step.</p>

<h3>
  
  
  Read the step log id from metadata/header of the message
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">ConsumerLoggerInterceptor</span> <span class="kd">implements</span> <span class="nc">ConsumerInterceptor</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="kd">static</span> <span class="kd">final</span> <span class="nc">Logger</span> <span class="n">logger</span> <span class="o">=</span> <span class="nc">LoggerFactory</span><span class="o">.</span><span class="na">getLogger</span><span class="o">();</span>

    <span class="nd">@Override</span>
    <span class="kd">public</span> <span class="nc">ConsumerRecords</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="nf">onConsume</span><span class="o">(</span><span class="nc">ConsumerRecords</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="n">records</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">records</span><span class="o">.</span><span class="na">forEach</span><span class="o">(</span><span class="n">record</span> <span class="o">-&gt;</span> <span class="o">{</span>
            <span class="nc">Header</span> <span class="n">header</span> <span class="o">=</span> <span class="n">record</span><span class="o">.</span><span class="na">headers</span><span class="o">().</span><span class="na">lastHeader</span><span class="o">(</span><span class="nc">PreDefinition</span><span class="o">.</span><span class="na">HTTP_HEADER_STEP_LOG_ID</span><span class="o">);</span>
            <span class="k">if</span> <span class="o">(</span><span class="n">header</span><span class="o">!=</span><span class="kc">null</span><span class="o">)</span> <span class="o">{</span>
                <span class="nc">String</span> <span class="n">stepLogId</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">String</span><span class="o">(</span><span class="n">header</span><span class="o">.</span><span class="na">value</span><span class="o">(),</span> <span class="nc">PreDefinition</span><span class="o">.</span><span class="na">CharsetEncoding</span><span class="o">);</span>
                <span class="n">logger</span><span class="o">.</span><span class="na">setStepLogId</span><span class="o">(</span><span class="n">stepLogId</span><span class="o">);</span>
            <span class="o">}</span><span class="k">else</span> <span class="o">{</span>
                <span class="n">logger</span><span class="o">.</span><span class="na">reset</span><span class="o">();</span><span class="c1">// reset to decouple each consume</span>
            <span class="o">}</span>
        <span class="o">});</span>

        <span class="k">return</span> <span class="n">records</span><span class="o">;</span>
    <span class="o">}</span>
<span class="c1">// other code</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  Configure beans for metadata/header Interceptors
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Configuration</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">KafkaBeansConfiguration</span> <span class="o">{</span>
    <span class="nd">@Bean</span>
    <span class="kd">public</span> <span class="nc">KafkaTemplate</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="nf">kafkaTemplate</span><span class="o">(</span><span class="nc">ProducerFactory</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="n">producerFactory</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nc">KafkaTemplate</span><span class="o">&lt;&gt;(</span><span class="n">producerFactory</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="nd">@Bean</span>
    <span class="kd">public</span> <span class="nc">ProducerFactory</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="nf">producerFactory</span><span class="o">(</span><span class="nc">KafkaProperties</span> <span class="n">properties</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">Map</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">Object</span><span class="o">&gt;</span> <span class="n">config</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">HashMap</span><span class="o">&lt;&gt;(</span><span class="n">properties</span><span class="o">.</span><span class="na">buildProducerProperties</span><span class="o">());</span>
        <span class="n">config</span><span class="o">.</span><span class="na">put</span><span class="o">(</span><span class="nc">ProducerConfig</span><span class="o">.</span><span class="na">INTERCEPTOR_CLASSES_CONFIG</span><span class="o">,</span> 
                <span class="nc">ProducerLoggerInterceptor</span><span class="o">.</span><span class="na">class</span><span class="o">.</span><span class="na">getName</span><span class="o">());</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nc">DefaultKafkaProducerFactory</span><span class="o">&lt;&gt;(</span><span class="n">config</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="nd">@Bean</span>
    <span class="kd">public</span> <span class="nc">ConsumerFactory</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="nf">consumerFactory</span><span class="o">(</span><span class="nc">KafkaProperties</span> <span class="n">properties</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">Map</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">Object</span><span class="o">&gt;</span> <span class="n">config</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">HashMap</span><span class="o">&lt;&gt;(</span><span class="n">properties</span><span class="o">.</span><span class="na">buildConsumerProperties</span><span class="o">());</span>
        <span class="n">config</span><span class="o">.</span><span class="na">put</span><span class="o">(</span><span class="nc">ConsumerConfig</span><span class="o">.</span><span class="na">INTERCEPTOR_CLASSES_CONFIG</span><span class="o">,</span> 
                <span class="nc">ConsumerLoggerInterceptor</span><span class="o">.</span><span class="na">class</span><span class="o">.</span><span class="na">getName</span><span class="o">());</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nc">DefaultKafkaConsumerFactory</span><span class="o">&lt;&gt;(</span><span class="n">config</span><span class="o">);</span>
    <span class="o">}</span>

    <span class="nd">@Bean</span>
    <span class="kd">public</span> <span class="nc">ConcurrentKafkaListenerContainerFactory</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="nf">kafkaListenerContainerFactory</span><span class="o">(</span>
            <span class="nc">ConsumerFactory</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="n">consumerFactory</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">ConcurrentKafkaListenerContainerFactory</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="n">factory</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ConcurrentKafkaListenerContainerFactory</span><span class="o">&lt;&gt;();</span>
        <span class="n">factory</span><span class="o">.</span><span class="na">setConsumerFactory</span><span class="o">(</span><span class="n">consumerFactory</span><span class="o">);</span>
        <span class="k">return</span> <span class="n">factory</span><span class="o">;</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  Send and receive messages through the queue, in case of kafka.
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">KafkaProducer</span> <span class="o">{</span>

    <span class="nd">@Resource</span>
    <span class="nc">MedicService</span> <span class="n">medicService</span><span class="o">;</span>

    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">KafkaTemplate</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="n">kafkaTemplate</span><span class="o">;</span>

    <span class="nd">@Autowired</span>
    <span class="kd">public</span> <span class="nf">KafkaProducer</span><span class="o">(</span><span class="nc">KafkaTemplate</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">,</span> <span class="nc">String</span><span class="o">&gt;</span> <span class="n">kafkaTemplate</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">this</span><span class="o">.</span><span class="na">kafkaTemplate</span> <span class="o">=</span> <span class="n">kafkaTemplate</span><span class="o">;</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">sendMessage</span><span class="o">(</span><span class="nc">String</span> <span class="n">message</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">medicService</span><span class="o">.</span><span class="na">callSelfInvocation</span><span class="o">(</span><span class="n">message</span><span class="o">);</span>
        <span class="n">kafkaTemplate</span><span class="o">.</span><span class="na">send</span><span class="o">(</span><span class="s">"demo-topic"</span><span class="o">,</span> <span class="n">message</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">KafkaConsumer</span> <span class="o">{</span>

    <span class="nd">@Resource</span>
    <span class="nc">UserService</span> <span class="n">userService</span><span class="o">;</span>

    <span class="nd">@KafkaListener</span><span class="o">(</span><span class="n">topics</span> <span class="o">=</span> <span class="s">"demo-topic"</span><span class="o">,</span> <span class="n">groupId</span> <span class="o">=</span> <span class="s">"demo-group"</span><span class="o">)</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">listen</span><span class="o">(</span><span class="nc">String</span> <span class="n">message</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">userService</span><span class="o">.</span><span class="na">getBySessionId</span><span class="o">(</span><span class="n">message</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  Upload the logs to web portal and access log traces
</h3>

<p>Please check out the section <code>Upload logs through the agent or OpenTelemetry</code> -&gt; <a href="https://dev.to/franknpc/how-to-automate-log-and-trace-without-writing-a-log-and-trace-2a0n">how to automate log and trace without writing a log and trace</a></p>

<h3>
  
  
  More over
</h3>

<ul>
<li>It's not only for Kafka, it works for all message queue systems.</li>
<li>The trace runs into messaging, will be part of the full trace.</li>
<li>Keep updated <a href="https://github.com/FrankNPC/bitryon-logging-examples" rel="noopener noreferrer">bitryon-logging-examples</a> for more use cases.</li>
</ul>

<h3>
  
  
  Conclusion
</h3>

<p>Now developers can see the <code>TRUE</code> trace at everywhere, microservice and message queue, kafka etc..</p>

<p>#java<br>
#message-queue<br>
#microservice<br>
#startup<br>
#observation<br>
#logging<br>
#log<br>
#logger<br>
#trace<br>
#tracer<br>
#tracing<br>
#workflow</p>

