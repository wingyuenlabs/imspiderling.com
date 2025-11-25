---
Title: How to Align Spring Boot Validation Errors with Your JSON Property Naming Strategy
Description: 
Author: İbrahim Gündüz
Date: 2025-11-25T21:00:18.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Problem
</h1>

<p>Our API uses a snake_case naming strategy for DTO deserialization and serialization. However, when validation fails, the error messages still use the original object property names, ignoring the configured naming strategy, which leads to inconsistent API responses.</p>

<h1>
  
  
  Solution
</h1>

<p>This issue occurs because bean validation happens after the incoming request has been deserialized into the DTO, but before the response is serialized. As a result, validation errors report the Java field names rather than the JSON property names defined by the configured naming strategy.</p>

<p>First, make sure that the property naming strategy is configured as shown below:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># application.properties</span>

<span class="py">spring.jackson.property-naming-strategy</span><span class="p">=</span><span class="err">SNAKE_CASE</span>
</code></pre>

</div>



<p>Override the default error generation with creating a custom error handler.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@RestControllerAdvice</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">ValidationExceptionHandler</span> <span class="o">{</span>

    <span class="nd">@ExceptionHandler</span><span class="o">(</span><span class="nc">MethodArgumentNotValidException</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">ResponseEntity</span><span class="o">&lt;?&gt;</span> <span class="n">handleMethodArgumentNotValidException</span><span class="o">(</span><span class="nc">MethodArgumentNotValidException</span> <span class="n">exception</span><span class="o">)</span> <span class="o">{</span>
       <span class="c1">// TODO: To be implemented</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>To access the mapped field names, inject an <code>ObjectMapper</code> and resolve the <strong>JSON</strong> property names by iterating over the list of field errors.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">private</span> <span class="kd">final</span> <span class="nc">ObjectMapper</span> <span class="n">objectMapper</span><span class="o">;</span>

<span class="c1">//...</span>

<span class="nd">@ExceptionHandler</span><span class="o">(</span><span class="nc">MethodArgumentNotValidException</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
<span class="kd">public</span> <span class="nc">ResponseEntity</span><span class="o">&lt;?&gt;</span> <span class="n">handleMethodArgumentNotValidException</span><span class="o">(</span><span class="nc">MethodArgumentNotValidException</span> <span class="n">exception</span><span class="o">)</span> <span class="o">{</span>
  <span class="nc">List</span><span class="o">&lt;</span><span class="nc">String</span><span class="o">&gt;</span> <span class="n">errors</span> <span class="o">=</span> <span class="n">exception</span><span class="o">.</span><span class="na">getBindingResult</span><span class="o">()</span>
          <span class="o">.</span><span class="na">getFieldErrors</span><span class="o">()</span>
          <span class="o">.</span><span class="na">stream</span><span class="o">()</span>
          <span class="o">.</span><span class="na">map</span><span class="o">(</span><span class="k">this</span><span class="o">::</span><span class="n">formatErrorMessage</span><span class="o">)</span>
          <span class="o">.</span><span class="na">toList</span><span class="o">();</span>

  <span class="k">return</span> <span class="nc">ResponseEntity</span><span class="o">.</span><span class="na">badRequest</span><span class="o">().</span><span class="na">body</span><span class="o">(</span><span class="nc">Map</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="s">"errors"</span><span class="o">,</span> <span class="n">errors</span><span class="o">));</span>
<span class="o">}</span>

<span class="kd">private</span> <span class="nc">String</span> <span class="nf">formatErrorMessage</span><span class="o">(</span><span class="nc">FieldError</span> <span class="n">fieldError</span><span class="o">)</span> <span class="o">{</span>
  <span class="nc">PropertyNamingStrategy</span> <span class="n">mapping</span> <span class="o">=</span> <span class="n">objectMapper</span><span class="o">.</span><span class="na">getPropertyNamingStrategy</span><span class="o">();</span>
  <span class="nc">String</span> <span class="n">mappedFieldName</span> <span class="o">=</span> <span class="n">mapping</span><span class="o">.</span><span class="na">nameForField</span><span class="o">(</span><span class="kc">null</span><span class="o">,</span> <span class="kc">null</span><span class="o">,</span> <span class="n">fieldError</span><span class="o">.</span><span class="na">getField</span><span class="o">());</span>

  <span class="k">return</span> <span class="nc">String</span><span class="o">.</span><span class="na">format</span><span class="o">(</span><span class="s">"%s.%s: %s"</span><span class="o">,</span> <span class="n">fieldError</span><span class="o">.</span><span class="na">getObjectName</span><span class="o">(),</span> <span class="n">mappedFieldName</span><span class="o">,</span> <span class="n">fieldError</span><span class="o">.</span><span class="na">getDefaultMessage</span><span class="o">());</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Let’s test it!<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>curl <span class="nt">-XPOST</span> <span class="s2">"http://api.yourdomain.com/v1/merchants"</span> <span class="se">\</span>
<span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
<span class="nt">-d</span> <span class="s2">"{}"</span>

<span class="o">{</span>
  <span class="s2">"errors"</span> : <span class="o">[</span> 
    <span class="s2">"merchantCreateRequest.partner_id: must not be null"</span>, 
    <span class="s2">"merchantCreateRequest.merchant_name: must not be null"</span> 
  <span class="o">]</span>
<span class="o">}</span>%                                                    
</code></pre>

</div>



<p>You can find all code examples in the following repository:<br>
<a href="https://github.com/ibrahimgunduz34/blog-code-examples/tree/master/spring-boot-validation-error" rel="noopener noreferrer">https://github.com/ibrahimgunduz34/blog-code-examples/tree/master/spring-boot-validation-error</a></p>

<p>Thanks for reading!</p>

