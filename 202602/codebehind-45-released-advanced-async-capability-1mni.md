---
Title: CodeBehind 4.5 Released; Advanced Async Capability
Description: 
Author: Elanat Framework
Date: 2026-02-06T22:03:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>As we mentioned in previous articles, version 4.5 is the latest version of the <a href="https://github.com/elanatframework/Code_behind" rel="noopener noreferrer">CodeBehind framework</a> based on .NET 7, and the next version, 4.6, is developed directly on .NET 10 by the <a href="https://elanat.net" rel="noopener noreferrer">Elanat</a> team.</p>

<p>In these versions, the framework structure has been completely rewritten based on async to be more responsive and perform faster than before.</p>

<p>Elanat's CodeBehind Framework is a full-stack, open-source (MIT licensed) framework for ASP.NET Core, introduced in 2023, with the main goal of combining the simplicity and familiarity of Web Forms (using .aspx files and the Code-Behind pattern) with the power and modern architecture of ASP.NET Core. By introducing WebForms Core as a modern, ViewStateless technology, this framework allows for direct server-side UI management. It also offers the ability to add features dynamically without recompiling the application with its modular architecture. This framework is a competitor to the default ASP.NET frameworks, filling the gap left by Microsoft's former WebForms in .NET Core by solving its problems and presenting itself as a flexible and high-performance alternative for developers looking for greater simplicity and reduced routing complexity.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7ch026bl4egfb2eoh90v.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7ch026bl4egfb2eoh90v.jpg" alt="CodeBehind framework 4.5" width="800" height="800"></a></p>

<p>In addition to adding the async structure, in this version we have done significant work to increase response performance to make the framework faster than before.</p>

<h2>
  
  
  Changes in the structure of controllers
</h2>

<p>Previously, controllers were defined as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">partial</span> <span class="k">class</span> <span class="nc">home</span> <span class="p">:</span> <span class="n">CodeBehindController</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">void</span> <span class="nf">PageLoad</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>
<span class="p">...</span>
</code></pre>

</div>



<p>But depending on the type of framework configuration, the new structure of controllers is different:</p>

<p><strong>1- Controller configuration in Route</strong><br>
In this case, you can use <code>async Task</code> instead of <code>void</code> automatically:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">PageLoad</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
<span class="p">{</span>
<span class="p">...</span>
</code></pre>

</div>



<p>Here you need to configure "Program.cs" using the <code>app.UseCodeBehindRouteAsync();</code> middleware.</p>

<p><strong>2- Default configuration (first-view MVC based on aspx pages)</strong></p>

<p>In this case, to perform <strong>async</strong> processing in the controller, you must use the <code>await</code> keyword and specify it in the view controller metadata section.</p>

<p><strong>Razor syntax example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>@page
@controller await MyController
<span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
...
</code></pre>

</div>



<p><strong>Standard syntax example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;</span><span class="err">%@</span> <span class="na">Page</span> <span class="na">Controller=</span><span class="s">"await MyController"</span> <span class="err">%</span><span class="nt">&gt;</span>
<span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
...
</code></pre>

</div>



<p>Here you need to configure "Program.cs" using the <code>app.UseCodeBehindAsync();</code> middleware.</p>

<h3>
  
  
  Practical example
</h3>

<p><strong>View</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>@page
@controller await MyController
<span class="nt">&lt;h1&gt;</span>Test Async<span class="nt">&lt;/h1&gt;</span>
</code></pre>

</div>



<p><strong>Explanation:</strong></p>

<ul>
<li>This view is defined using <strong>Razor syntax</strong>.</li>
<li>
<code>@controller await MyController</code> specifies that the <strong>MyController</strong> should run <strong>asynchronously (async)</strong>.</li>
</ul>

<p><strong>Controller</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">CodeBehind</span><span class="p">;</span>

<span class="k">public</span> <span class="k">partial</span> <span class="k">class</span> <span class="nc">MyController</span> <span class="p">:</span> <span class="n">CodeBehindController</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">PageLoad</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">string</span> <span class="n">result</span> <span class="p">=</span> <span class="k">await</span> <span class="nf">DoSomethingAsync</span><span class="p">();</span>
        <span class="nf">Write</span><span class="p">(</span><span class="n">result</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="kt">string</span><span class="p">&gt;</span> <span class="nf">DoSomethingAsync</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">await</span> <span class="n">Task</span><span class="p">.</span><span class="nf">Delay</span><span class="p">(</span><span class="m">2000</span><span class="p">);</span>

        <span class="k">return</span> <span class="s">"Hello CodeBehind! (after async work)"</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Explanation:</strong></p>

<ul>
<li>
<code>MyController</code> inherits from <code>CodeBehindController</code>.</li>
<li>The <code>PageLoad</code> method is now <strong>async</strong> and <strong>returns a Task</strong>, not void.</li>
<li>Inside <code>PageLoad</code>, the <code>DoSomethingAsync</code> method is called, and we <strong>await</strong> its result.</li>
<li>
<code>DoSomethingAsync</code> simulates an asynchronous operation (<code>Task.Delay(2000)</code> waits for 2 seconds) and then returns a string.</li>
<li>
<code>Write(result)</code> prints the result to the view.</li>
</ul>

<p>In other words: <strong>This controller processes data asynchronously and then displays it on the page, without blocking the entire application.</strong></p>

<p><strong>Program.cs</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">var</span> <span class="n">builder</span> <span class="p">=</span> <span class="n">WebApplication</span><span class="p">.</span><span class="nf">CreateBuilder</span><span class="p">(</span><span class="n">args</span><span class="p">);</span>

<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="nf">AddCodeBehind</span><span class="p">();</span>

<span class="kt">var</span> <span class="n">app</span> <span class="p">=</span> <span class="n">builder</span><span class="p">.</span><span class="nf">Build</span><span class="p">();</span>

<span class="n">app</span><span class="p">.</span><span class="nf">UseCodeBehindAsync</span><span class="p">();</span>

<span class="n">app</span><span class="p">.</span><span class="nf">Run</span><span class="p">();</span>
</code></pre>

</div>



<p><strong>Explanation:</strong></p>

<ul>
<li>In <code>Program.cs</code>, the CodeBehind service is added to the application.</li>
<li>
<code>app.UseCodeBehindAsync();</code> tells the app to handle <strong>controllers asynchronously</strong>.</li>
<li>The application is then started with <code>app.Run()</code>.</li>
</ul>

<p>Together, these three components demonstrate <strong>how CodeBehind 4.5 processes a web page asynchronously</strong>, improving <strong>performance and responsiveness</strong> without blocking the app.</p>

<h2>
  
  
  Summary
</h2>

<p>CodeBehind 4.5 brings <strong>full async support</strong> to both route-based and ASPX-based MVC controllers, improving <strong>responsiveness</strong> and <strong>performance</strong>. Developers can now implement async operations easily using <code>await</code> without adding complexity to their code.</p>

<h2>
  
  
  Related links
</h2>

<p>CodeBehind on GitHub:<br>
<a href="https://github.com/elanatframework/Code_behind" rel="noopener noreferrer">https://github.com/elanatframework/Code_behind</a></p>

<p>Get CodeBehind from NuGet:<br>
<a href="https://www.nuget.org/packages/CodeBehind/" rel="noopener noreferrer">https://www.nuget.org/packages/CodeBehind/</a></p>

<p>CodeBehind page:<br>
<a href="https://elanat.net/page_content/code_behind" rel="noopener noreferrer">https://elanat.net/page_content/code_behind</a></p>

