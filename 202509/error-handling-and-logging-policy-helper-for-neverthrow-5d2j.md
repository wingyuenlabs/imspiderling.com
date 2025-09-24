---
Title: Error Handling and logging policy helper for neverthrow
Description: 
Author: Camilo Andres Vera Ruiz
Date: 2025-09-24T22:10:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ever found yourself in a TypeScript project using the Result pattern, only to discover that while it promises type-safe error handling, your code is now cluttered with boilerplate <code>if (result.isErr())</code> checks and inconsistent logging? If so, you're not alone. The power of this pattern can get lost as your codebase grows, especially if you're not fully embracing a functional style. This article explores a solution to this common problem, aiming to streamline error handling and restore clarity to your code.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzqm8530ub5wpuodwjx0m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzqm8530ub5wpuodwjx0m.png" alt="Code comparison between Code without error handler and with error handler" width="800" height="730"></a></p>

<p>About a year and a half ago, I discovered the Result pattern from functional programming, allowing me to get a type safe error handling, in typescript. There are multiple libraries for that, such as <a href="https://www.npmjs.com/package/ts-results" rel="noopener noreferrer"><code>ts-results</code></a>, <a href="https://www.npmjs.com/package/neverthrow" rel="noopener noreferrer"><code>neverthrow</code></a> and the more ambitious <a href="https://effect.website/" rel="noopener noreferrer"><code>Effect</code></a> but all implement the same idea.</p>

<p>I successfully used this pattern in my production apps using the <code>ts-results</code> package at first, and then changing to <code>neverthrow</code> (due to be more complete and better maintained), and it proved to be a path toward writing more robust applications, with better observability, getting a consistent logging policy and robust error handling.</p>

<h2>
  
  
  Initial Aproach
</h2>

<p>At first I used it directly in all my functions and methods like this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// genericFunction.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">err</span><span class="p">,</span> <span class="nx">ok</span><span class="p">,</span> <span class="nx">Result</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">neverthrow</span><span class="dl">'</span>

<span class="kd">type</span> <span class="nx">FunctionError</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UNEXPECTED_ERROR</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">GENERIC_ERROR_1</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">GENERIC_ERROR_2</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">NOT_FOUND</span><span class="dl">'</span>
  <span class="nx">data</span><span class="p">?:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">unknown</span><span class="o">&gt;</span>
<span class="p">}</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">genericFunction</span><span class="p">(</span><span class="nx">input</span><span class="p">:</span> <span class="p">{</span> <span class="nl">param</span><span class="p">:</span> <span class="kr">string</span> <span class="p">}):</span> 
<span class="nx">Result</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">param</span><span class="p">:</span> <span class="kr">string</span> <span class="p">},</span> <span class="nx">FunctionError</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="c1">// ...</span>
  <span class="c1">// Something went wrong</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">input</span><span class="p">.</span><span class="nx">param</span> <span class="o">!==</span> <span class="dl">'</span><span class="s1">ok</span><span class="dl">'</span><span class="p">)</span> <span class="k">return</span> <span class="nf">err</span><span class="p">({</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">GENERIC_ERROR_1</span><span class="dl">'</span> <span class="p">})</span>

  <span class="c1">// All ok</span>
  <span class="k">return</span> <span class="nf">ok</span><span class="p">({</span> <span class="na">param</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ok</span><span class="dl">'</span> <span class="p">})</span>
<span class="p">}</span>


</code></pre>

</div>


<p>Then when I needed to call a method or a function inside another, I did something like this:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// upperFunction.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">err</span><span class="p">,</span> <span class="nx">ok</span><span class="p">,</span> <span class="nx">Result</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">neverthrow</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">genericFunction</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./genericFunction</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Logger</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../utils/logger</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">logger</span> <span class="o">=</span> <span class="nx">Logger</span>
<span class="kd">type</span> <span class="nx">UpperFunctionError</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UNEXPECTED_ERROR</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">UPPER_ERROR</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">NOT_FOUND</span><span class="dl">'</span>
  <span class="nx">data</span><span class="p">?:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">unknown</span><span class="o">&gt;</span>
<span class="p">}</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">upperFunction</span><span class="p">(</span><span class="nx">input</span><span class="p">:</span> <span class="p">{</span> <span class="nl">param</span><span class="p">:</span> <span class="kr">string</span> <span class="p">}):</span> 
<span class="nx">Result</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">param</span><span class="p">:</span> <span class="kr">string</span> <span class="p">},</span> <span class="nx">UpperFunctionError</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="nf">genericFunction</span><span class="p">(</span><span class="nx">input</span><span class="p">)</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nf">isErr</span><span class="p">())</span> <span class="p">{</span>
    <span class="k">switch </span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">error</span><span class="p">.</span><span class="nx">code</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">case</span> <span class="dl">'</span><span class="s1">UNEXPECTED_ERROR</span><span class="dl">'</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">err</span><span class="p">({</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UNEXPECTED_ERROR</span><span class="dl">'</span> <span class="p">})</span>
      <span class="k">case</span> <span class="dl">'</span><span class="s1">GENERIC_ERROR_1</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">customError</span> <span class="o">=</span> <span class="p">{</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UNEXPECTED_ERROR</span><span class="dl">'</span> <span class="k">as</span> <span class="kd">const</span> <span class="p">}</span>
        <span class="nx">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">This should not happen</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
          <span class="p">...</span><span class="nx">customError</span><span class="p">,</span>
          <span class="na">params</span><span class="p">:</span> <span class="nx">input</span>
        <span class="p">})</span>
        <span class="k">return</span> <span class="nf">err</span><span class="p">(</span><span class="nx">customError</span><span class="p">)</span>
      <span class="p">}</span>
      <span class="k">case</span> <span class="dl">'</span><span class="s1">GENERIC_ERROR_2</span><span class="dl">'</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">err</span><span class="p">({</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UPPER_ERROR</span><span class="dl">'</span> <span class="p">})</span>
      <span class="k">case</span> <span class="dl">'</span><span class="s1">NOT_FOUND</span><span class="dl">'</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">err</span><span class="p">({</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">NOT_FOUND</span><span class="dl">'</span> <span class="p">})</span>
      <span class="na">default</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">err</span><span class="p">({</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UNEXPECTED_ERROR</span><span class="dl">'</span> <span class="p">})</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="c1">// All ok, executing the next step</span>
  <span class="c1">// ...</span>
  <span class="k">return</span> <span class="nf">ok</span><span class="p">({</span> <span class="na">param</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ok</span><span class="dl">'</span> <span class="p">})</span>
<span class="p">}</span>
</code></pre>

</div>


<p>This approach works well as a first step and of course can be improved in many ways e.g. using the functional programming tools that the library gave us, and sure there will be cases when some errors are expected and even desired and the function logic will translate them into behaviors instead of another errors, but this should wrap the main idea.</p>

<p>While one might think it's simpler to combine the error types and just propagate the error, this can lead to confusing and meaningless errors in the upper layers of the application. For instance, returning a <code>GROUP_NOT_FOUND</code> error from a function that's supposed to get purchasable items for an e-commerce platform, it really doesn't provide useful context to the calling function</p>
<h2>
  
  
  The Problem
</h2>

<p>Looking at the code we can imagine how complex error handling can get, especially in complex services that implement business logic, a method can turn into 80% repetitive error handling where we have to take into account: when to log, what to log and try to be consistent over all the services, something that even with a LLM code assistant helping us in the code editor, we can get it wrong.</p>
<h2>
  
  
  The Solution
</h2>

<p>Wouldn't it be nice if we had a helper that implements a consistent set of rules, and it required us to handle all the posible error cases, with type safety and autocompletion. Well I've developed a helper for this exact purpose, which you can find in this GitHub gist:</p>


<div class="ltag_gist-liquid-tag">
  
</div>



<h2>
  
  
  How to use it
</h2>

<p>The idea with this helper is managing a consistent logging policy and an error handling methodology similar in what is found in the <strong>Rust programming language</strong>, where we split the error in expected <strong>workable errors</strong>, and <strong>critical errors</strong> that should halt program execution, and cause a system redeployment.</p>

<p>To achieve this, the helper performs three main functions:</p>

<ul>
<li><p>It enforces a consistent error type across all methods. We should use the <code>errorBuilder</code> helper to construct the specific error type that must be returned by all functions that could fail. In essence, this helper standardizes the error type for all our "Result" functions.</p></li>
<li><p>It provides a <code>Panic</code> function to force the process to stop when necessary. The primary use case for this is identifying situations that make your application unable to function and getting this information as soon as possible in a production environment. For example, if a fundamental configuration variable or a database connection string is missing during the application's startup validation, we can immediately call <code>Panic</code>. This stops the process, allowing developers to be notified through observability tools or cloud alarms that something has gone critically wrong, triggering an automatic redeployment of the last working version of the service.</p></li>
</ul>

<blockquote>
<p>Remember to use this function with caution. It's probably not a good idea to trigger a panic for a temporary failure like a third-party API or database connection issue. Depending on the context, these might be normal or even scheduled events, such as a cloud provider performing database maintenance and updates at 3:00 a.m.</p>
</blockquote>

<ul>
<li>Using some <strong>TypeScript magic</strong>, it includes the <code>errorHandler</code> helper. This allows us to directly map all possible error codes, eliminating repetitive boilerplate code. It also automatically handles logging and constructs an error object that conforms to the expected return type.</li>
</ul>

<p>The handler implements the next set of rules</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj5omq0dpf9awwofjayii.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj5omq0dpf9awwofjayii.png" alt="Diagram that resumes the implemented logging policy" width="643" height="381"></a></p>

<ul>
<li>All the <code>reservedErrorCodes</code> get automatically translated to <code>UNEXPECTED_ERROR</code>, they are treated as non critical unexpected errors, that means that when this error is returned, let say in the catch in a query function, we must log all the failure details, and the error will propagate as <code>UNEXPECTED_ERROR</code> over all the function calls, that way the error is logged only once.</li>
<li>When the error is translated to the same error or another expected error, it just log with log level, that way it help us to follow the execution trace.</li>
<li>When the expected error is translated to <code>UNEXPECTED_ERROR</code> meaning that when an error that should not occur is detected, (e.g. <code>USER_ALREADY_EXIST</code> when we just deleted the user so probably a bug), the error is logged with error level, including all the data given to the handler.</li>
<li>When the error is translated to <code>PANIC</code> the handler calls the <code>Panic</code> function to log the cause of the critical error and finalize the process, which should trigger a server redeployment. Depending on the project and framework, we may need to modify this function.</li>
</ul>

<p>The next example include all the mentioned cases<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">ok</span><span class="p">,</span> <span class="nx">Result</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">neverthrow</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">genericFunction</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./genericFunction</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Logger</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../logger</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">ErrorBuilder</span><span class="p">,</span> <span class="nx">errorHandlerResult</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">src/utils/error</span><span class="dl">'</span>

<span class="c1">// Build the function error using the ErrorBuilder type  </span>
<span class="kd">type</span> <span class="nx">UpperFunctionError</span> <span class="o">=</span> <span class="nx">ErrorBuilder</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UPPER_ERROR</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">NOT_FOUND</span><span class="dl">'</span> <span class="p">}</span><span class="o">&gt;</span>
<span class="k">export</span> <span class="kd">function</span> <span class="nf">upperFunction</span><span class="p">(</span><span class="nx">input</span><span class="p">:</span> <span class="p">{</span> <span class="nl">param</span><span class="p">:</span> <span class="kr">string</span> <span class="p">}):</span> 
<span class="nx">Result</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">param</span><span class="p">:</span> <span class="kr">string</span> <span class="p">},</span> <span class="nx">UpperFunctionError</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="nf">genericFunction</span><span class="p">(</span><span class="nx">input</span><span class="p">)</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nf">isErr</span><span class="p">())</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nf">errorHandlerResult</span><span class="p">(</span><span class="nx">res</span><span class="p">.</span><span class="nx">error</span><span class="p">,</span> <span class="nx">Logger</span><span class="p">,</span> <span class="p">{</span>
      <span class="c1">// The compiler force us to managing all the posible error </span>
      <span class="c1">// codes, getting autocompletion for all the input and </span>
      <span class="c1">// output codes</span>

      <span class="c1">// This detects a unmanageable error and crash the app, so </span>
      <span class="c1">// we add all the posible info to log, PANIC doesn't need to </span>
      <span class="c1">// be defined in the returnable errors, the handler knows </span>
      <span class="c1">// that this will throw</span>
      <span class="na">GENERIC_ERROR_1</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">PANIC</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">This should not happen</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">extraParams</span><span class="p">:</span> <span class="nx">input</span>
      <span class="p">},</span>
      <span class="c1">// Translation to UNEXPECTED_ERROR so include all the posible</span>
      <span class="c1">// data and log as error, when the error is non critical. </span>
      <span class="c1">// Note that UNEXPECTED_ERROR is always defined for all the </span>
      <span class="c1">// error types generated by the ErrorBuilder helper</span>
      <span class="na">GENERIC_ERROR_2</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">UNEXPECTED_ERROR</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">This should not happen nut is not critical</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">extraParams</span><span class="p">:</span> <span class="nx">input</span>
      <span class="p">},</span>
      <span class="c1">// Either the same error or another expected error is </span>
      <span class="c1">// mapped so just log as log level without message</span>
      <span class="na">NOT_FOUND</span><span class="p">:</span> <span class="p">{</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">NOT_FOUND</span><span class="dl">'</span> <span class="p">}</span>
    <span class="p">})</span>
  <span class="p">}</span>

  <span class="c1">// Everything ok and typescript knows</span>
  <span class="c1">// ...</span>
  <span class="k">return</span> <span class="nf">ok</span><span class="p">({</span> <span class="na">param</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ok</span><span class="dl">'</span> <span class="p">})</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Applying this, we can reduce the error handling code in our service, and avoid the manual logging, not only that but we can changue the logging rules in any moment (modifying the provided handler) without having to changue that in every service.</p>

<p>This helper includes both a raw <code>errorHandler</code> and a <code>errorHandlerResult</code> so we can directly construct the result error object or not, depending on the case, e.g. we can use the standalone <code>errorHandler</code> inside a map like method from neverthrow Result, that automatically transforms the returned value into a result value.</p>

<h2>
  
  
  Extra Recommendations
</h2>

<ul>
<li>The handler always receives the logger instance that should be used to log the error, This way it's easy to integrate with custom logging solutions such as pino logger. Is recommended to implement a correlationId to follow the execution trace for each service call.</li>
<li>About another logs cases, that of course depend on the project, but a reasonable rule of thumb is to log with log level, all the mutations in a consistent format like
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">logger</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">''</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">param1</span><span class="p">:</span> <span class="dl">'</span><span class="s1">123</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">param2</span><span class="p">:</span> <span class="mi">123</span>
<span class="p">})</span>
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>I think that using this helper customized for you project needs, and following the extra recommendations, is a good start point that  will help you to make your system more robust and observable.</p>

<p>If you like the idea and have another log policies in mind, let me known in the comments, maybe I can make this into a npm package if another rules and cases come to the discussion, otherwise just take the helper and use it directly in your project.</p>

<p>Hope it helps. </p>

