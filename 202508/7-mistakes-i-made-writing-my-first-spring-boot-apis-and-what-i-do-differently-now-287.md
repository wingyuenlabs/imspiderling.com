---
Title: 🛠️ 7 Mistakes I Made Writing My First Spring Boot APIs (And What I Do Differently Now)
Description: 
Author: Ashish Saran Shakya
Date: 2025-08-30T21:35:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>Spring Boot makes building APIs <em>feel</em> deceptively easy.</p>

<p>When I started out, I focused on just “getting things working.” But when things break—or just scale badly—you realize that working code isn’t always <strong>good code</strong>.</p>

<p>Here are <strong>7 mistakes I made writing my first Spring Boot APIs</strong>, and what I now do to write cleaner, more maintainable backends.</p>




<h3>
  
  
  1. <strong>Not Using Validation Annotations (<code>@Valid</code>, <code>@NotNull</code>, etc.)</strong>
</h3>

<p><strong>What I did:</strong><br><br>
Manually validated request payloads deep inside controllers or services. Missed fields caused confusing errors or null pointer exceptions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="k">if</span> <span class="o">(</span><span class="n">userRequest</span><span class="o">.</span><span class="na">getEmail</span><span class="o">()</span> <span class="o">==</span> <span class="kc">null</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nf">IllegalArgumentException</span><span class="o">(</span><span class="s">"Email is required"</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p><strong>Why it's a problem:</strong></p>

<ul>
<li>Makes code noisy and repetitive</li>
<li>Delays validation too deep into the flow</li>
<li>Prone to human error</li>
</ul>

<p><strong>What I do now:</strong><br><br>
Use Spring's built-in validation support with annotations and <code>@Valid</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">UserRequest</span> <span class="o">{</span>
    <span class="nd">@NotNull</span>
    <span class="nd">@Email</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">email</span><span class="o">;</span>
<span class="o">}</span>
</code></pre>

</div>



<p>And in the controller:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@PostMapping</span><span class="o">(</span><span class="s">"/users"</span><span class="o">)</span>
<span class="kd">public</span> <span class="nc">ResponseEntity</span><span class="o">&lt;?&gt;</span> <span class="n">createUser</span><span class="o">(</span><span class="nd">@Valid</span> <span class="nd">@RequestBody</span> <span class="nc">UserRequest</span> <span class="n">request</span><span class="o">)</span> <span class="o">{</span>
    <span class="o">...</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Pair this with <code>@ControllerAdvice</code> to handle validation exceptions cleanly.</p>




<h3>
  
  
  2. <strong>Hardcoding config with <code>@Value</code> instead of using <code>@ConfigurationProperties</code></strong>
</h3>

<p><strong>What I did:</strong><br><br>
Injected config values everywhere using <code>@Value</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Value</span><span class="o">(</span><span class="s">"${aws.s3.bucket}"</span><span class="o">)</span>
<span class="kd">private</span> <span class="nc">String</span> <span class="n">bucketName</span><span class="o">;</span>
</code></pre>

</div>



<p><strong>Why it's a problem:</strong></p>

<ul>
<li>Scattered config makes things hard to manage</li>
<li>No way to validate values</li>
<li>Harder to write unit tests</li>
</ul>

<p><strong>What I do now:</strong><br><br>
Use <code>@ConfigurationProperties</code> to bind entire config blocks into structured classes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@ConfigurationProperties</span><span class="o">(</span><span class="n">prefix</span> <span class="o">=</span> <span class="s">"aws.s3"</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">S3Properties</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">bucket</span><span class="o">;</span>
    <span class="c1">// getters/setters</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This gives you validation, IDE support, and cleaner wiring.</p>




<h3>
  
  
  3. <strong>Logging the Wrong Way</strong>
</h3>

<p><strong>What I did:</strong><br><br>
Used <code>System.out.println()</code> and vague messages like “Error occurred” or “Debug here”.</p>

<p><strong>Why it's a problem:</strong></p>

<ul>
<li>Not visible in production logs</li>
<li>Lacks context (who, what, when)</li>
<li>No severity levels or structured format</li>
</ul>

<p><strong>What I do now:</strong><br><br>
Use SLF4J with clear, parameterized logging:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="n">log</span><span class="o">.</span><span class="na">info</span><span class="o">(</span><span class="s">"User {} uploaded file {}"</span><span class="o">,</span> <span class="n">userId</span><span class="o">,</span> <span class="n">fileName</span><span class="o">);</span>
</code></pre>

</div>



<p>And for errors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="n">log</span><span class="o">.</span><span class="na">error</span><span class="o">(</span><span class="s">"Failed to process request for user {}"</span><span class="o">,</span> <span class="n">userId</span><span class="o">,</span> <span class="n">e</span><span class="o">);</span>
</code></pre>

</div>



<p>Be intentional with log levels: <code>info</code>, <code>warn</code>, <code>error</code>, and avoid logging entire stack traces unless necessary.</p>




<h3>
  
  
  4. <strong>Messy Exception Handling</strong>
</h3>

<p><strong>What I did:</strong><br><br>
Wrapped everything in try-catch blocks, often rethrowing or printing stack traces manually.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="k">try</span> <span class="o">{</span>
    <span class="o">...</span>
<span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">Exception</span> <span class="n">e</span><span class="o">)</span> <span class="o">{</span>
    <span class="n">e</span><span class="o">.</span><span class="na">printStackTrace</span><span class="o">();</span>
    <span class="k">throw</span> <span class="n">e</span><span class="o">;</span>
<span class="o">}</span>
</code></pre>

</div>



<p><strong>Why it's a problem:</strong></p>

<ul>
<li>Duplicates logic</li>
<li>Makes error responses inconsistent</li>
<li>Hides real issues in production</li>
</ul>

<p><strong>What I do now:</strong><br><br>
Define custom exceptions and handle them centrally using <code>@ControllerAdvice</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@ResponseStatus</span><span class="o">(</span><span class="nc">HttpStatus</span><span class="o">.</span><span class="na">NOT_FOUND</span><span class="o">)</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">ResourceNotFoundException</span> <span class="kd">extends</span> <span class="nc">RuntimeException</span> <span class="o">{</span>
    <span class="kd">public</span> <span class="nf">ResourceNotFoundException</span><span class="o">(</span><span class="nc">String</span> <span class="n">msg</span><span class="o">)</span> <span class="o">{</span>
        <span class="kd">super</span><span class="o">(</span><span class="n">msg</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>And in a global exception handler:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@RestControllerAdvice</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">GlobalExceptionHandler</span> <span class="o">{</span>
    <span class="nd">@ExceptionHandler</span><span class="o">(</span><span class="nc">ResourceNotFoundException</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">ResponseEntity</span><span class="o">&lt;?&gt;</span> <span class="n">handleNotFound</span><span class="o">(</span><span class="nc">ResourceNotFoundException</span> <span class="n">ex</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="nc">ResponseEntity</span><span class="o">.</span><span class="na">status</span><span class="o">(</span><span class="nc">HttpStatus</span><span class="o">.</span><span class="na">NOT_FOUND</span><span class="o">)</span>
                             <span class="o">.</span><span class="na">body</span><span class="o">(</span><span class="k">new</span> <span class="nc">ErrorResponse</span><span class="o">(</span><span class="n">ex</span><span class="o">.</span><span class="na">getMessage</span><span class="o">()));</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Clean, consistent, and maintainable.</p>




<h3>
  
  
  5. <strong>Overusing <code>@Autowired</code> Field Injection</strong>
</h3>

<p><strong>What I did:</strong><br><br>
Injected everything using <code>@Autowired</code> on fields.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Autowired</span>
<span class="kd">private</span> <span class="nc">UserService</span> <span class="n">userService</span><span class="o">;</span>
</code></pre>

</div>



<p><strong>Why it's a problem:</strong></p>

<ul>
<li>Makes dependencies hard to track</li>
<li>Breaks immutability</li>
<li>Difficult to write unit tests</li>
</ul>

<p><strong>What I do now:</strong><br><br>
Use constructor injection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">UserController</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">UserService</span> <span class="n">userService</span><span class="o">;</span>

    <span class="kd">public</span> <span class="nf">UserController</span><span class="o">(</span><span class="nc">UserService</span> <span class="n">userService</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">this</span><span class="o">.</span><span class="na">userService</span> <span class="o">=</span> <span class="n">userService</span><span class="o">;</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Bonus: With one constructor, Spring injects it automatically—no <code>@Autowired</code> needed.</p>




<h3>
  
  
  6. <strong>Forgetting API Documentation (like Swagger/OpenAPI)</strong>
</h3>

<p><strong>What I did:</strong><br><br>
Left API documentation in someone's Notion page or none at all.</p>

<p><strong>Why it's a problem:</strong></p>

<ul>
<li>Frontend teams have to guess request/response formats</li>
<li>No way to test endpoints quickly</li>
<li>Becomes a bottleneck during integration</li>
</ul>

<p><strong>What I do now:</strong><br><br>
Use <a href="https://springdoc.org/" rel="noopener noreferrer">springdoc-openapi</a> to generate live docs with minimal setup:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="c">&lt;!-- build.gradle / pom.xml --&gt;</span>
<span class="nt">&lt;dependency&gt;</span>
  <span class="nt">&lt;groupId&gt;</span>org.springdoc<span class="nt">&lt;/groupId&gt;</span>
  <span class="nt">&lt;artifactId&gt;</span>springdoc-openapi-ui<span class="nt">&lt;/artifactId&gt;</span>
  <span class="nt">&lt;version&gt;</span>1.6.14<span class="nt">&lt;/version&gt;</span>
<span class="nt">&lt;/dependency&gt;</span>
</code></pre>

</div>



<p>Then access Swagger UI at:<br><br>
<code>http://localhost:8080/swagger-ui/index.html</code></p>

<p>It reflects your controller annotations and helps both devs and testers.</p>


<h3>
  
  
  7. <strong>Mixing DTOs with Entities</strong>
</h3>

<p><strong>What I did:</strong><br><br>
Used JPA entities directly in request bodies and responses.</p>

<p><strong>Why it's a problem:</strong></p>

<ul>
<li>Leaks internal persistence logic</li>
<li>Adds accidental complexity (lazy loading, cascade issues)</li>
<li>Makes refactoring risky</li>
</ul>

<p><strong>What I do now:</strong><br><br>
Create dedicated DTO classes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">UserDto</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">name</span><span class="o">;</span>
    <span class="kd">private</span> <span class="nc">String</span> <span class="n">email</span><span class="o">;</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Use mappers like MapStruct, ModelMapper, or just write a utility:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="nc">UserDto</span> <span class="nf">toDto</span><span class="o">(</span><span class="nc">User</span> <span class="n">user</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nf">UserDto</span><span class="o">(</span><span class="n">user</span><span class="o">.</span><span class="na">getName</span><span class="o">(),</span> <span class="n">user</span><span class="o">.</span><span class="na">getEmail</span><span class="o">());</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Entities stay clean. APIs stay stable.</p>




<h2>
  
  
  💬 Final Thoughts
</h2>

<p>Spring Boot abstracts a lot of complexity—but with that power comes responsibility.</p>

<p>These 7 mistakes cost me time, performance, and headaches. But fixing them made me a better engineer.</p>

<p><strong>What’s one mistake <em>you</em> made in your early Spring Boot journey?</strong><br><br>
Would love to hear your thoughts below 👇</p>

